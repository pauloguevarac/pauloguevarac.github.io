---
title: "LLMs in SRE: Building an Automated Kubernetes Incident Response Assistant with RAG"
date: 2026-07-02 10:00:00 -0600
categories: [SRE, Kubernetes]
tags: [sre, kubernetes, llm, rag, monitoring]
toc: true
comments: true
---

# Introduction

In complex Kubernetes environments, a single alert—like a CrashLoopBackOff or a failing liveness probe—can trigger a cascade of downstream notifications. When a 3 AM incident occurs, Site Reliability Engineers (SREs) must quickly ingest the alert, navigate complex cluster trees via `kubectl` to fetch logs and events, locate the company's internal wiki or markdown runbooks, correlate the symptoms, and formulate a repair strategy.

With the advances in Large Language Models (LLMs) and **Retrieval-Augmented Generation (RAG)** in 2026, we can automate the bulk of this triage process.

This article details how to build an **automated Kubernetes incident response assistant** that intercepts an alert, pulls live logs and events from your cluster, searches your company's markdown-based internal runbooks using semantic search, and generates a contextualized, step-by-step resolution plan.

---

## The SRE Assistant Architecture

Instead of feeding a raw, massive stream of logs to an expensive public API, our SRE Assistant uses RAG to surgically inject only the *exact, relevant troubleshooting steps* into the LLM's prompt alongside the active cluster errors.

```text
  [ Prometheus Alert ]
           │
           ▼
[ SRE Assistant Triggered ]
           │
           ├──> Queries k8s API (logs, events, pod spec)
           │
           ├──> Fetches matching Runbook from Vector DB (RAG)
           │
           ▼
[ Contextual LLM Analysis ]
           │
           ▼
 [ Slack / Teams Report: Root Cause + Step-by-Step Fix ]
```

---

## Step-by-Step Implementation

Let's write a Python-based SRE triage script that connects to your Kubernetes cluster and uses **ChromaDB** as a local vector database for runbooks and the **OpenAI API** to diagnose a failing deployment.

### 1. Preparing the Vector Database with Runbooks (`sre_assistant/vector_db.py`)

First, let's create a script that indexes our company's internal Markdown documentation and runbooks into a local vector database.

```python
import os
import glob
import chromadb
from chromadb.utils import embedding_functions

# Initialize ChromaDB client (local persistent database)
client = chromadb.PersistentClient(path="./runbook_db")

# Use a standard, fast local embedding function (Sentence Transformers)
embedding_fn = embedding_functions.DefaultEmbeddingFunction()
collection = client.get_or_create_collection(
    name="kubernetes_runbooks", 
    embedding_function=embedding_fn
)

def index_runbooks(runbooks_dir):
    print(f"📦 Indexing runbooks from {runbooks_dir}...")
    runbook_files = glob.glob(os.path.join(runbooks_dir, "*.md"))
    
    for file_path in runbook_files:
        with open(file_path, "r", encoding="utf-8") as f:
            content = f.read()
            
        file_name = os.path.basename(file_path)
        print(f"📝 Indexing {file_name}...")
        
        # Add the document to the vector collection
        collection.add(
            documents=[content],
            metadatas=[{"source": file_name}],
            ids=[file_name]
        )
    print("✅ Runbooks successfully indexed!")

if __name__ == "__main__":
    # Simulate an internal wiki folder of SRE runbooks
    os.makedirs("./wiki", exist_ok=True)
    
    # Write a sample runbook for demonstration
    sample_runbook = """
    # Runbook: Resolving PostgreSQL Database Connection Failures
    ## Symptoms:
    Application pods crash with 'Connection refused' or 'Dial timeout' database errors.
    
    ## Steps to Triage:
    1. Check if the database service is up: `kubectl get pods -l app=postgres`
    2. Check the service port configuration: `kubectl describe service postgres`
    3. Verify credentials and environment variables. Ensure the DB_HOST points to the service name `postgres-service` in the correct namespace.
    4. If the database pod is stuck in Pending, check the PersistentVolumeClaim: `kubectl describe pvc` and verify if there is available disk space.
    """
    
    with open("./wiki/postgres_runbook.md", "w") as f:
        f.write(sample_runbook)
        
    index_runbooks("./wiki")
```

### 2. The Core Kubernetes Triager (`sre_assistant/triager.py`)

Now we write the triager script that queries the Kubernetes cluster using the official client, performs a RAG query to fetch the correct runbook, and generates the incident report.

```python
import os
import sys
from kubernetes import client as k8s_client, config as k8s_config
import chromadb
from chromadb.utils import embedding_functions
from openai import OpenAI

# 1. Initialize API Clients and local Databases
try:
    k8s_config.load_kube_config() # Loads local ~/.kube/config
except Exception:
    k8s_config.load_incluster_config() # If running inside Kubernetes cluster

v1_core = k8s_client.CoreV1Api()
chroma_client = chromadb.PersistentClient(path="./runbook_db")
embedding_fn = embedding_functions.DefaultEmbeddingFunction()
collection = chroma_client.get_collection(name="kubernetes_runbooks", embedding_function=embedding_fn)
openai_client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

def get_failed_pod_telemetry(namespace, pod_name):
    """Gathers logs, events, and specs of a failing pod."""
    print(f"🔍 Gathering Kubernetes telemetry for {pod_name} in namespace {namespace}...")
    telemetry = {}
    
    # Get Pod specification and status
    try:
        pod = v1_core.read_namespaced_pod(name=pod_name, namespace=namespace)
        telemetry["status"] = pod.status.phase
        telemetry["conditions"] = [c.message for c in pod.status.conditions if c.message]
        telemetry["restart_count"] = pod.status.container_statuses[0].restart_count if pod.status.container_statuses else 0
    except Exception as e:
        telemetry["status_error"] = f"Failed to get pod status: {e}"
        
    # Get recent logs (last 50 lines)
    try:
        logs = v1_core.read_namespaced_pod_log(name=pod_name, namespace=namespace, tail_lines=50)
        telemetry["logs"] = logs
    except Exception as e:
        telemetry["logs"] = f"Failed to retrieve logs: {e}"
        
    # Get events associated with this pod
    try:
        events = v1_core.list_namespaced_event(namespace=namespace, field_selector=f"involvedObject.name={pod_name}")
        telemetry["events"] = [f"[{e.type}] {e.reason}: {e.message}" for e in events.items[-5:]]
    except Exception as e:
        telemetry["events"] = [f"Failed to retrieve events: {e}"]
        
    return telemetry

def query_relevant_runbooks(search_query):
    """Performs vector search over indexed runbooks."""
    print(f"📚 Searching vector database for query: '{search_query}'...")
    results = collection.query(
        query_texts=[search_query],
        n_results=1
    )
    if results and results["documents"]:
        return results["documents"][0][0], results["metadatas"][0][0]["source"]
    return "No matching runbook found.", "None"

def generate_incident_response_plan(namespace, pod_name, error_context):
    # Get telemetry
    telemetry = get_failed_pod_telemetry(namespace, pod_name)
    
    # Query runbook using logs/events summary as query
    log_summary = telemetry.get("logs", "")[:200]
    runbook_text, runbook_src = query_relevant_runbooks(log_summary)
    
    # Build prompt
    prompt = f"""
    You are an AI SRE Agent acting on a critical Kubernetes incident.
    An alert has fired on pod {pod_name} in namespace {namespace}.
    
    LIVE TELEMETRY FROM KUBERNETES:
    - Status: {telemetry.get('status')}
    - Restart Count: {telemetry.get('restart_count')}
    - Conditions: {telemetry.get('conditions')}
    - Recent Events: {telemetry.get('events')}
    - Log Tail:
    ```
    {telemetry.get('logs')}
    ```
    
    RELEVANT RUNBOOK FOUND (Source: {runbook_src}):
    ```markdown
    {runbook_text}
    ```
    
    TASK:
    Analyze the live telemetry and the company runbook. Provide a high-impact incident response report with:
    1. **🚨 Executive Summary**: What is broken and how critical is it?
    2. **🎯 Root Cause Analysis**: Why did the pod fail (correlating logs and events)?
    3. **🛠️ Actionable Resolution Steps**: Step-by-step instructions (including exact `kubectl` commands) to solve the issue.
    4. **🛡️ Prevention Suggestions**: How to avoid this in the future.
    """
    
    print("🤖 Processing incident and generating report via OpenAI...")
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.2
    )
    
    return response.choices[0].message.content

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: python3 triager.py <namespace> <pod_name>")
        sys.exit(1)
        
    namespace_arg = sys.argv[1]
    pod_arg = sys.argv[2]
    
    report = generate_incident_response_plan(namespace_arg, pod_arg, "Failing DB connection")
    print("\n" + "="*50 + "\n🚨 INCIDENT RESPONSE REPORT\n" + "="*50)
    print(report)
```

---

## How to Trigger the triager from Alertmanager

To operationalize this, you can configure your **Prometheus Alertmanager** to send webhook notifications to a receiver service (e.g. built on FastAPI), which extracts the failing namespace/pod parameters and executes the triage script, posting the output directly into your `#sre-alerts` Slack or Teams channel.

Example Alertmanager configuration snippet:

```yaml
receivers:
- name: 'sre-assistant-webhook'
  webhook_configs:
  - url: 'http://sre-assistant.monitoring.svc.cluster.local/alert-webhook'
    send_resolved: false
```

---

## Best Practices and Security Goggles

Using LLMs to automatically diagnose and potentially *remediate* production environments has several crucial SRE safety guidelines:

1. **Read-Only by Default**: The automated assistant should run with a highly restricted Kubernetes `Role` / `ClusterRole` that has only `get`, `list`, and `watch` permissions. It should **never** have write permissions (`delete`, `patch`, `create`) on your production cluster.
2. **Deterministic Sandboxing**: If you decide to experiment with *self-healing* actions (such as automated container restarts), limit the execution to a specific list of harmless commands (like `kubectl rollout restart`) and never allow raw shell-command execution.
3. **Data Anonymization**: Logs can contain sensitive Personally Identifiable Information (PII) or secrets. Implement a regex-based logging mask in your python triage pipeline to strip out passwords, emails, and tokens before sending logs to an external LLM.

---

## Conclusion

Automating Kubernetes incident triage using RAG and LLMs removes the panic from incident response. SREs no longer have to manually piece together logs, check event loops, and dig up old runbooks. Instead, they receive a beautifully structured, highly relevant triage report with exact commands to copy and paste within seconds of the alert.

The future of SRE isn't about working harder; it is about building smarter, context-aware platforms that guide us through the chaos of high-scale system operations.
