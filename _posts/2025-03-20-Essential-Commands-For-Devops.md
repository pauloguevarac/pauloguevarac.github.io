---
title: "Essential Commands For DevOps: Build Your Core Toolbox"
date: 2025-03-20 10:00:00 -0600
categories: [Devops, Cheatsheet]
tags: [devops, linux, bash, commands]
toc: true
comments: true
---

# Let’s Dive In and Build Your DevOps Toolbox Together

By the time you finish reading, you’ll approach that 3 AM alert with confidence rather than dread.

## Table of Contents
1. [System Reconnaissance: Know Your Battlefield](#system-reconnaissance-know-your-battlefield)
2. [Docker Domination: Container Commands that Save Lives](#docker-domination-container-commands-that-save-lives)
3. [Kubernetes Kung Fu: Orchestrating with Confidence](#kubernetes-kung-fu-orchestrating-with-confidence)
4. [Networking Ninjas: Diagnose and Fix Connectivity Issues](#networking-ninjas-diagnose-and-fix-connectivity-issues)
5. [Monitoring Magic: Catching Problems Before Users Do](#monitoring-magic-catching-problems-before-users-do)
6. [Shell Scripting Sorcery: Automation for the Win](#shell-scripting-sorcery-automation-for-the-win)
7. [Cloud Companion: AWS, Azure, and GCP Essentials](#cloud-companion-aws-azure-and-gcp-essentials)
8. [Git Guru: Version Control Mastery](#git-guru-version-control-mastery)
9. [Security Sentinel: Protecting Your Kingdom](#security-sentinel-protecting-your-kingdom)
10. [Performance Prowess: Making Things Blazing Fast](#performance-prowess-making-things-blazing-fast)

## System Reconnaissance: Know Your Battlefield

### 1. The Full System Checkup
```
top
```
Real-life scenario: When Jake joined PixelPerfect Apps as their first DevOps hire, he inherited a mysterious server that nobody understood. The application would randomly slow to a crawl. His first step? Running top to see what processes were consuming resources. He discovered a forgotten data backup script that ran at random intervals, hogging all available CPU. A simple scheduling fix later, and the team thought he was a wizard.

top provides a dynamic real-time view of system processes. Look for high CPU or memory usage to quickly identify resource hogs.

### 2. Disk Space Detective
```
df -h
```
Real-life scenario: Maria was enjoying her coffee when alerts started flooding in. The payment system was down! Running df -h revealed the culprit – the disk was 100% full. The application logs had grown out of control. She freed up space and then implemented log rotation, saving the day and earning the trust of her team.

The -h flag makes the output human-readable with gigabytes instead of byte counts. When a server becomes unresponsive, checking disk space should be one of your first moves.

### 3. Finding Disk Space Hogs
```
du -h --max-depth=1 /path | sort -hr
```
Real-life scenario: After discovering a disk space issue, Carlos needed to find which directory was the culprit. This command helped him identify that the Docker images folder had grown to 50GB. He cleared unused images and implemented a scheduled cleanup policy.

This command shows the size of directories at the specified path, sorted from largest to smallest. The --max-depth=1 flag means it won't recursively list the size of every subdirectory.

### 4. Check Running Services
```
systemctl list-units --type=service --state=running
```
Real-life scenario: Taking over a new environment, Priya needed to understand what services were actually running. This command gave her a complete inventory, allowing her to document the system and discover several unnecessary services that could be disabled to improve security.

This works on systems using systemd (most modern Linux distributions). For older systems, try service --status-all instead.

### 5. Process Investigation
```
ps aux | grep [service_name]
```
Real-life scenario: The customer database was running slowly, and Tom needed to confirm if the database service was actually running with the correct configuration. Using this command, he could see the exact command-line parameters and confirm that the service was running with insufficient memory allocation.

The aux options provide detailed information about all processes, and piping to grep filters for just what you're looking for.

### 6. Check Open Ports
```
ss -tulpn
```
Real-life scenario: After implementing a new firewall rule, Sophia needed to verify which ports were actually listening on the server. This command showed her that the application was listening on an unexpected port, explaining why it wasn’t accessible after the firewall change.

This command shows all TCP and UDP ports that are listening, along with the process using them. It’s the modern replacement for the older netstat command.

### 7. Check System Load History
```
sar -q
```
Real-life scenario: Users complained about intermittent slowness, but whenever Diego checked, the system seemed fine. Using sar, he could see historical load data, revealing periodic spikes at exactly 15 minutes past each hour – precisely when a scheduled job ran.

The System Activity Reporter provides historical performance data, which is invaluable for diagnosing intermittent issues. You might need to install the sysstat package first.

8. Memory Usage Details
free -h
Real-life scenario: After deploying a new version of their Java application, Raj noticed the server becoming unresponsive after a few hours. Running free -h showed that the physical memory was nearly exhausted, indicating a memory leak in the application. He implemented better JVM heap settings and monitoring.

This command provides a clear summary of memory usage, including physical memory and swap space.

9. Real-time Resource Monitoring
htop
Real-life scenario: During a critical product launch, Emma kept htop running on a second monitor. When traffic surged and the system started to struggle, she could immediately see which services were affected and take action before users noticed problems.

htop is an improved version of top with a more user-friendly interface and additional features. You may need to install it first with your package manager.

10. Quick System Summary
neofetch
Real-life scenario: When joining as a consultant, Wei needed to quickly understand the specs of multiple servers in a client’s environment. Instead of digging through various commands, neofetch gave him a quick visual summary of each system.

While often considered a customization tool, neofetch provides a surprisingly useful system overview, including OS, kernel version, uptime, and hardware specifications.

Docker Domination: Container Commands that Save Lives
11. List All Containers
docker ps -a
Real-life scenario: After taking over a project with mysterious resource usage, Ava ran this command and discovered dozens of zombie containers in various states. These abandoned containers were consuming resources without providing any value. Cleaning them up immediately improved performance.

The -a flag shows all containers, including stopped ones, which is crucial for understanding the complete state of your Docker environment.

12. Container Resource Usage
docker stats
Real-life scenario: When the monitoring system alerted about high server load, Miguel used docker stats to see which containers were consuming the most resources. He discovered a data processing container using 90% of the CPU due to a configuration error, allowing him to fix it before it affected other services.

This command provides a live stream of resource usage (CPU, memory, network, disk) for all running containers.

13. Clean Up Docker System
docker system prune -af
Real-life scenario: The CI/CD pipeline was failing with “no space left on device” errors. Lina ran this command to clean up unused Docker resources, freeing over 30GB of disk space and getting the pipeline back on track.

This removes all stopped containers, dangling images, and unused networks and volumes. The -a flag removes all unused images, not just dangling ones, and -f skips the confirmation prompt.

14. Inspect Container Details
docker inspect [container_id]
Real-life scenario: A microservice couldn’t connect to the database despite having the correct configuration. Using docker inspect, Omar discovered the container was on a different network than the database, explaining the connection issue.

This command provides detailed information about a container, including network settings, volumes, environment variables, and more.

15. Follow Container Logs
docker logs -f [container_id]
Real-life scenario: During a deployment, Julia noticed a service was starting but then immediately exiting. By following the logs in real-time, she caught an authentication error that wasn’t appearing in the application’s own logs.

The -f flag follows the log output, similar to tail -f, showing new log entries as they occur.

16. Execute Command Inside Container
docker exec -it [container_id] bash
Real-life scenario: The API container was responding with unexpected errors. By executing a shell inside the container, Tyrone was able to check the actual configuration files and discovered that environment variables weren’t being passed correctly.

This opens an interactive bash shell inside a running container, allowing you to explore its internal state. Replace bash with sh for containers that don't have bash installed.

17. Check Docker Networks
docker network ls
Real-life scenario: After migrating to a new server, microservices couldn’t communicate. Chen listed all Docker networks and discovered that the migration script had created duplicate networks with similar names, causing containers to connect to the wrong networks.

This command lists all Docker networks, helping you understand the networking topology of your Docker environment.

18. Copy Files Between Host and Container
docker cp [container_id]:/path/to/file ./local/path
Real-life scenario: When a critical database backup job failed, Nadia couldn’t access the logs through the standard interface. She used docker cp to extract the actual backup files and logs from inside the container, allowing her to diagnose and fix the issue.

This command copies files or directories between the host and containers without needing to mount volumes.

19. Quick Temporary Container
docker run --rm -it [image] [command]
Real-life scenario: To test a complex DNS issue, Harper needed to run diagnostic tools that weren’t available on the host system. She created a quick temporary container with networking tools, diagnosed the issue, and the container cleaned itself up afterward.

The --rm flag automatically removes the container when it exits, and -it makes it interactive.

20. Build with No Cache
docker build --no-cache -t [image_name] .
Real-life scenario: A mysterious bug appeared in production that couldn’t be reproduced locally. Javier suspected cached layers in the Docker build process and used this command to create a completely fresh build, which indeed behaved differently and revealed the issue.

This forces Docker to rebuild every layer from scratch, ignoring any cached layers, which is useful for troubleshooting build issues.

Kubernetes Kung Fu: Orchestrating with Confidence
21. Check All Resources
kubectl get all --all-namespaces
Real-life scenario: After joining DigitalNomad Inc. as their first dedicated DevOps engineer, Zara needed to quickly understand their Kubernetes environment. This command gave her a complete overview of all resources across all namespaces, revealing several abandoned deployments and services.

This provides a broad overview of pods, services, deployments, and more across your entire cluster.

22. Pod Details
kubectl describe pod [pod_name]
Real-life scenario: The authentication service kept crashing and restarting. By describing the pod, Alex could see it was failing health checks because the database connection was timing out — the database was overloaded, not the auth service itself.

This command shows detailed information about a specific pod, including events, conditions, and status — critical for debugging misbehaving pods.

23. Follow Pod Logs
kubectl logs -f [pod_name] -c [container_name]
Real-life scenario: During a critical financial processing job, Leila kept watch on the logs. When she noticed unusual patterns, she was able to pause the job before it could create duplicate transactions.

The -f flag follows log output in real-time, and the -c flag specifies which container to watch if the pod has multiple containers.

24. Execute Command in Pod
kubectl exec -it [pod_name] -- [command]
Real-life scenario: Users reported incorrect data in the product catalog. By executing database queries directly inside the database pod, Richard discovered that a recent migration had failed silently, leaving some tables in an inconsistent state.

This lets you run commands directly inside a pod, similar to Docker’s exec command.

25. Port Forwarding
kubectl port-forward [pod_name] 8080:80
Real-life scenario: During an urgent debugging session, Nina needed direct access to a service running inside the cluster. Port forwarding allowed her to connect directly to the specific pod from her laptop, bypassing load balancers and ingress controllers.

This forwards a local port to a port on the pod, allowing direct access for debugging without exposing the service publicly.

26. Watch Resource Changes
kubectl get pods -w
Real-life scenario: During a rolling update, Marco wanted to ensure pods were replacing smoothly. Using the watch flag, he could see in real-time as old pods terminated and new ones came online, catching a scheduling issue before it affected all replicas.

The -w flag watches for changes to the resources, updating the output when changes occur.

27. Check Pod Resource Usage
kubectl top pods
Real-life scenario: After implementing a new caching layer, Prisha wanted to confirm that it was actually reducing database load. This command showed her that the database pods were indeed using significantly less CPU than before.

This shows CPU and memory usage for pods, similar to the top command for traditional processes.

28. Edit Resources on the Fly
kubectl edit deployment [deployment_name]
Real-life scenario: During an unexpected traffic spike, Matteo needed to quickly scale up the API servers. Rather than applying a new YAML file, he directly edited the deployment to increase the replica count, responding to the situation in seconds.

This opens the resource definition in your default text editor, allowing you to make changes directly.

29. Apply Configuration Changes
kubectl apply -f [file_or_directory]
Real-life scenario: After resolving a production incident, Aisha documented the changes needed to prevent recurrence. She created a YAML file with the new configuration and applied it to the cluster, ensuring the fix was properly documented and could be applied to other environments.

This applies configuration changes defined in YAML files, creating resources if they don’t exist or updating them if they do.

30. Drain Node for Maintenance
kubectl drain [node_name] --ignore-daemonsets
Real-life scenario: Before performing server maintenance, Raj needed to safely move all workloads off the node. Draining the node ensured all pods were rescheduled to other nodes before he took the server offline.

This cordons the node (marks it as unschedulable) and evicts all pods, allowing for safe maintenance.

Networking Ninjas: Diagnose and Fix Connectivity Issues
31. Test Connectivity with Ping
ping -c 4 [hostname]
Real-life scenario: After a network reconfiguration, Lucia needed to quickly verify basic connectivity to various services. The simple ping command helped her confirm that the network routes were working as expected.

The -c 4 flag limits the ping to 4 packets, giving you quick feedback without having to manually terminate the command.

32. Trace Network Path
traceroute [hostname]
Real-life scenario: Users in one office location were experiencing slow access to the company’s web application. Using traceroute, Dmitri discovered that traffic from that office was being routed through a congested path, allowing him to work with the network team on a better route.

This shows the path packets take to reach the destination, including each hop along the way and the time taken.

33. DNS Lookup
dig [domain] +short
Real-life scenario: After migrating to a new DNS provider, Fatima needed to verify that DNS records were propagating correctly. The dig command allowed her to check specific records and confirm the change was working as expected.

The +short flag gives just the answer section of the DNS response, making it easier to read.

34. Port Scanning
nmap -p 1-1000 [hostname]
Real-life scenario: Before deploying a new application to production, Gabriel wanted to verify the firewall settings. Using nmap, he could confirm exactly which ports were accessible, catching an unexpectedly open port that could have been a security risk.

This scans ports 1–1000 on the specified host, showing which ones are open. Be cautious about scanning systems you don’t own!

35. Check Listening Ports
lsof -i -P -n | grep LISTEN
Real-life scenario: After a configuration change, the application server wasn’t accepting connections. Using this command, Zoe discovered that the service was actually listening on a different port than expected.

This shows all processes that are listening on network ports, including the port number and process ID.

36. HTTP Request Inspection
curl -v [url]
Real-life scenario: When integrating with a third-party API, Ben was getting unexpected responses. Using curl’s verbose mode, he could see the exact headers being sent and received, revealing that the API required a specific content-type header.

The -v flag shows the full HTTP request and response, including headers, which is invaluable for debugging API issues.

37. Continuous Network Monitoring
mtr [hostname]
Real-life scenario: Users reported intermittent connection issues to the database server. By running mtr during the problem times, Hannah was able to identify packet loss at a specific router, providing evidence to the network team.

mtr combines the features of ping and traceroute with continuous monitoring, showing packet loss and latency for each hop.

38. TCP Connection Analysis
tcpdump -i eth0 host [hostname] and port [port]
Real-life scenario: A microservice communication issue was proving difficult to diagnose. By capturing and analyzing the actual TCP packets, Jamal discovered that the client was closing connections prematurely due to a timeout setting that was too aggressive.

This captures and displays TCP packets matching the specified criteria, allowing deep inspection of network traffic.

39. SSL Certificate Verification
openssl s_client -connect [hostname]:443 -servername [hostname]
Real-life scenario: After receiving alerts about an expiring SSL certificate, Elena needed to verify that the renewed certificate was correctly installed. This command showed her the full certificate chain, confirming the update was successful.

This connects to an SSL/TLS server and displays the certificate information, helping diagnose SSL-related issues.

40. Bandwidth Testing
iperf3 -c [server] -p [port]
Real-life scenario: After moving to a new data center, Kwame needed to verify the network performance met the promised specifications. Using iperf3, he could measure actual throughput between servers and confirm they were getting the expected bandwidth.

This measures network throughput between hosts. You’ll need to run iperf3 in server mode on the destination with iperf3 -s.

Monitoring Magic: Catching Problems Before Users Do
41. Simple HTTP Service Check
watch -n 5 "curl -s -o /dev/null -w '%{http_code}' [url]"
Real-life scenario: During a critical deployment, Isabella wanted to continuously monitor the API’s health. This command showed her the HTTP status code every 5 seconds, letting her immediately spot when the service returned anything other than 200.

This checks the HTTP status code of a URL every 5 seconds, making it easy to spot changes.

42. Continuous Resource Monitoring
dstat -cdnmgy --top-cpu --top-mem
Real-life scenario: While trying to diagnose an intermittent performance issue, Leo ran dstat during a load test. The comprehensive output helped him identify that the problem occurred when disk I/O spiked above a certain threshold.

This all-in-one monitoring tool displays CPU, disk, network, memory, and system statistics in a continuously updating display.

43. Log File Monitoring
tail -f /var/log/syslog | grep ERROR
Real-life scenario: After deploying a new feature, Mei kept an eye on the logs to catch any issues. This command filtered for error messages in real-time, helping her spot and fix a problem before most users encountered it.

This follows the log file in real-time and filters for lines containing “ERROR”, helping you focus on important messages.

44. Disk I/O Monitoring
iostat -xz 5
Real-life scenario: An application was performing poorly despite low CPU and memory usage. Using iostat, Victor discovered that the disk was experiencing high wait times, indicating a potential I/O bottleneck.

This displays disk I/O statistics every 5 seconds, including utilization, queue length, and wait time.

45. Check Running Processes
ps aux --sort=-%cpu | head -10
Real-life scenario: When a server became unresponsive, Yara quickly needed to identify resource hogs. This command showed her the top 10 CPU-consuming processes, revealing a runaway analytics job that was overwhelming the system.

This lists the top 10 processes sorted by CPU usage, helping you quickly identify resource-intensive processes.

46. Network Connection States
netstat -tunapl
Real-life scenario: Users reported slow connection times to the application. Using netstat, Omar discovered hundreds of connections in the CLOSE_WAIT state, indicating that the application wasn't properly closing connections.

This shows detailed information about network connections, including their state, which is crucial for diagnosing network-related issues.

47. Check Failed Systemd Services
systemctl --failed
Real-life scenario: After a power outage, Chiara needed to quickly check which services failed to restart. This command gave her a concise list, allowing her to prioritize the most critical services for recovery.

This lists systemd services that have failed, helping you quickly identify issues after system changes or failures.

48. Historical Performance Data
journalctl -u [service] --since "1 hour ago"
Real-life scenario: A service suddenly slowed down, but everything looked normal in the current metrics. By checking the logs from the past hour, Noah discovered that a database migration had run automatically, explaining the temporary slowdown.

This shows logs for a specific systemd service within a time range, helping you correlate events with performance issues.

49. Check System Load Average
uptime
Real-life scenario: During a post-mortem analysis, Priya needed a quick way to check if the system had been overloaded. The load averages from uptime confirmed that the load had been steadily increasing for hours before the crash.

This shows how long the system has been running and the average load over the last 1, 5, and 15 minutes.

50. Monitor File System Events
inotifywait -m -r /path/to/directory
Real-life scenario: A configuration file kept changing unexpectedly, causing service restarts. By monitoring the directory, Alejandro caught an automated script that was overwriting the file every hour, solving a mystery that had plagued the team for weeks.

This monitors a directory for file system events (create, modify, delete, etc.), which is useful for debugging issues related to file changes.

Shell Scripting Sorcery: Automation for the Win
51. Find and Replace in Multiple Files
find . -type f -name "*.conf" -exec sed -i 's/old_value/new_value/g' {} \;
Real-life scenario: After changing a database hostname, Nikita needed to update dozens of configuration files across multiple directories. This command allowed her to make the change consistently across all files without manual editing.

This finds all .conf files and replaces “old_value” with “new_value” in each file.

52. Batch File Renaming
for f in *.txt; do mv "$f" "${f%.txt}.md"; done
Real-life scenario: When the documentation team decided to switch from plain text to Markdown, Raj needed to convert hundreds of files. This simple loop renamed all .txt files to .md files in seconds.

This renames all .txt files in the current directory to have the .md extension instead.

53. Run Command on Multiple Servers
for server in server1 server2 server3; do ssh $server "uptime"; done
Real-life scenario: Before a major deployment, Sophia needed to check the load on all web servers. Instead of logging into each one individually, this command gave her a quick overview of all servers in seconds.

This executes the “uptime” command on multiple servers sequentially, displaying the results locally.

54. Monitor Until Success
until curl -s http://myapp/health | grep -q 'ok'; do sleep 5; done; echo "Service is up!"
Real-life scenario: After initiating a large database restore, Miguel needed to know when the application was ready to receive traffic again. This command monitored the health endpoint until it returned success, allowing him to go work on other tasks.

This periodically checks a condition and exits once it’s met, useful for waiting for services to become available.

55. Parallel Command Execution
ls *.log | parallel gzip {}
Real-life scenario: When preparing log files for archival, Hannah needed to compress hundreds of files. Using parallel, she was able to utilize all CPU cores and complete the task in a fraction of the time.

This executes the specified command on each input item in parallel, taking advantage of multiple CPU cores.

56. Create Self-Destructing Messages
echo "This will self-destruct in 60 seconds" | at now + 1 minute
Real-life scenario: While performing emergency maintenance, Chen needed a reminder to check on a critical service after making changes. This command scheduled a notification for 60 seconds later, keeping him on track during a stressful situation.

This schedules a command to run at a specified time in the future.

57. Automatic Error Handling
#!/bin/bash
set -e
set -o pipefail
Real-life scenario: After a deployment script silently failed but reported success, Aiden improved all scripts with these options. Later, when a subtle error occurred in a database migration, the script immediately halted instead of continuing with potentially damaging operations.

These options make scripts fail fast: set -e exits on any error, and set -o pipefail ensures pipelines fail if any command fails.

58. Loop with Timeout
timeout 300s bash -c 'while ! mysqladmin ping -h db.example.com --silent; do sleep 1; done'
Real-life scenario: During startup, Zara needed to wait for the database to become available, but not indefinitely. This command tried to connect to the database for five minutes before timing out, preventing the script from hanging forever.

This attempts to execute a command repeatedly until it succeeds, but gives up after a specified timeout.

59. Capture Both Output and Errors
result=$(command 2>&1)
Real-life scenario: When debugging an automated backup script, Diego needed to see both the normal output and any error messages. This command captured everything into a single variable, making it easier to understand what was happening.

This captures both stdout and stderr from a command, useful for logging or error handling.

60. Conditional Command Execution
grep -q "ERROR" /var/log/app.log && slack-alert "Errors found in application log"
Real-life scenario: As part of a nightly check, Leila set up a script that scanned logs for errors and only sent a notification if problems were found, preventing alert fatigue while ensuring critical issues were reported.

The && operator runs the second command only if the first one succeeds (returns a zero exit code).

Cloud Companion: AWS, Azure, and GCP Essentials
61. AWS: List All EC2 Instances
aws ec2 describe-instances --query "Reservations[*].Instances[*].[InstanceId, State.Name, InstanceType, PrivateIpAddress, PublicIpAddress, Tags[?Key=='Name'].Value|[0]]" --output table
Real-life scenario: During a cost optimization exercise, Rohan needed to inventory all running EC2 instances across regions. This command provided a concise table of instance details, helping him identify forgotten instances that could be terminated.

This provides a tabular summary of EC2 instances with key information, perfect for getting a quick overview of your infrastructure.

62. AWS: Find Expensive Resources
aws ce get-cost-and-usage --time-period Start=2024-01-01,End=2024-02-01 --granularity MONTHLY --metrics BlendedCost --group-by Type=DIMENSION,Key=SERVICE
Real-life scenario: After receiving an unexpectedly high AWS bill, Tanya used this command to break down costs by service. She discovered that an unused Redshift cluster was responsible for most of the increase.

This provides a cost breakdown by AWS service for the specified time period, helping identify where your cloud spend is going.

63. Azure: List Resource Groups
az group list --output table
Real-life scenario: As a consultant starting with a new client, Marcus needed to quickly understand their Azure organization. This command gave him a clear view of all resource groups, helping him map the environment.

This lists all Azure resource groups in a table format, providing a high-level view of your Azure resources.

64. Azure: VM Status Check
az vm list --show-details --output table
Real-life scenario: During a planned maintenance window, Priya needed to confirm that all VMs were successfully restarted. This command provided a quick status check of all VMs in the subscription.

This shows detailed information about all VMs, including power state, size, and location.

65. GCP: List Compute Instances
gcloud compute instances list
Real-life scenario: After a company acquisition, James needed to audit all GCP resources. This command helped him identify running instances across all projects, including several development environments that could be shut down.

This lists all Compute Engine instances in the current project, including zone, machine type, and status.

66. GCP: Check IAM Permissions
gcloud projects get-iam-policy [project-id]
Real-life scenario: During a security audit, Lina discovered unexpected API calls in the logs. This command helped her review all IAM permissions, revealing an overly permissive service account that was being misused.

This shows the IAM policy for a GCP project, helping you audit permissions and identify security risks.

67. AWS: S3 Bucket Size
aws s3 ls s3://bucket-name --recursive --human-readable --summarize
Real-life scenario: Storage costs were increasing rapidly, and Ahmed needed to identify which S3 buckets were growing the fastest. This command provided a summary of each bucket’s size, helping prioritize optimization efforts.

This shows all objects in an S3 bucket with human-readable sizes and a total summary at the end.

68. AWS: Check Security Groups
aws ec2 describe-security-groups --query "SecurityGroups[?IpPermissions[?ToPort==22]].{Name: GroupName, ID: GroupId}"
Real-life scenario: During a security review, Olivia needed to identify all security groups that allowed SSH access. This command helped her find several groups with overly permissive SSH rules that needed to be restricted.

This finds security groups that allow access to port 22 (SSH), which is useful for security audits.

69. Azure: List Recent Deployments
az deployment group list --resource-group [resource-group] --query "reverse(sort_by([].{Name:name, Time:properties.timestamp, Status:properties.provisioningState}, &Time))" --output table
Real-life scenario: After a failed deployment, Ravi needed to understand what changes had been made recently. This command showed him a timeline of deployments, helping identify when the problem was introduced.

This lists recent deployments to a resource group in reverse chronological order.

70. GCP: Check Billing Export
gcloud billing export bq describe
Real-life scenario: To prepare for a FinOps initiative, Maya needed to confirm that billing data was being exported to BigQuery. This command helped her verify the export configuration and identify missing datasets.

This confirms the billing export configuration to BigQuery, which is essential for cost analysis and optimization.

Git Guru: Version Control Mastery
71. Find Recent Branches
git for-each-ref --sort=-committerdate refs/heads/ --format='%(committerdate:short) %(refname:short)'
Real-life scenario: After returning from vacation, Yasmin needed to recall which branches she had been working on. This command showed her branches sorted by most recent activity, helping her quickly resume her work.

This lists all local branches sorted by the date of their most recent commit, helping you find branches you’ve worked on recently.

72. Search Commit History
git log --grep="fix authentication" --oneline
Real-life scenario: The authentication system was failing, and Carlos remembered fixing a similar issue months ago. This command helped him find the relevant commit, saving hours of debugging.

This searches commit messages for a specific phrase, showing only matching commits.

73. View File History
git log -p -- filename
Real-life scenario: A critical configuration file was behaving unexpectedly. Using this command, Naomi traced the evolution of the file, discovering a subtle change from weeks ago that was causing the current issue.

This shows the commit history for a specific file, including the actual changes made in each commit.

74. Find Merge Conflicts Before They Happen
git merge-tree $(git merge-base HEAD feature-branch) HEAD feature-branch
Real-life scenario: Before attempting a complex merge of a long-running feature branch, Raj wanted to identify potential conflicts. This command helped him spot several problematic areas that he could resolve proactively.

This simulates a merge without actually performing it, showing you what conflicts would occur.

75. Clean Untracked Files
git clean -fdx
Real-life scenario: After switching between multiple build configurations, Mia’s project was cluttered with build artifacts causing inconsistent behavior. This command gave her a fresh start with only tracked files.

This removes all untracked files and directories, including those ignored by .gitignore, giving you a clean working directory.

76. Track Code Ownership
git blame -L 10,20 filename
Real-life scenario: A subtle bug appeared in a critical payment processing function. Using git blame, Tomas identified that the relevant code was last modified during a performance optimization sprint, providing crucial context for understanding the issue.

This shows who last modified each line in the specified range of a file, along with the commit information.

77. Quick Commit Fixup
git commit --fixup=HEAD~3
Real-life scenario: While reviewing his code before a pull request, Daniel noticed a typo in a function he had written several commits ago. This command allowed him to create a fixup commit that would be automatically squashed during rebase.

This creates a commit that will be automatically squashed into the specified commit during a rebase with the — autosquash option.

78. Compare Branches
git log --graph --oneline --decorate branch1..branch2
Real-life scenario: Before merging a feature branch, Elena wanted to ensure she understood all the changes that would be introduced. This command showed her the commits that existed in one branch but not the other.

This shows commits that exist in branch2 but not in branch1, helping you understand what changes will be introduced by a merge.

79. Find Lost Commits
git reflog
Real-life scenario: After an accidental reset, Jamal thought he had lost hours of work. The reflog helped him find the commit hash of his lost work, allowing him to recover it with a simple cherry-pick.

This shows a log of all reference updates in the repository, helping you find commits that are no longer referenced by any branch or tag.

80. Stash Selected Changes
git stash push -p
Real-life scenario: While working on a feature, Sofia was asked to fix a critical bug. She needed to switch branches but didn’t want to commit her half-finished work. This command allowed her to selectively stash only the changes she wanted to keep.

This interactively lets you choose which changes to stash, allowing you to keep some changes in your working directory while stashing others.

Security Sentinel: Protecting Your Kingdom
81. Check for Open Ports
sudo nmap -sS -O 127.0.0.1
Real-life scenario: After configuring a new firewall, Hiroshi wanted to verify that only the intended ports were accessible. This scan revealed that an old development service was still listening on a high port, creating a potential security risk.

This performs a stealthy SYN scan on localhost and attempts to identify the operating system, showing open ports and running services.

82. Find Files with SUID Bit
find / -type f -perm -4000 2>/dev/null
Real-life scenario: During a security audit, Maya discovered an unusual SUID binary in a non-standard location. Further investigation revealed it was malware that had been installed by an attacker, allowing privilege escalation.

This finds files with the SUID bit set, which allows users to execute the file with the permissions of the file owner.

83. Check for Failed Login Attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
Real-life scenario: After noticing unusual system behavior, Leo checked for failed login attempts and discovered hundreds of failed SSH login attempts from a single IP address, indicating a brute force attack in progress.

This shows a count of failed login attempts by IP address, sorted from most to least attempts.

84. Monitor Real-time Network Connections
watch -n 1 "netstat -tuna | grep ESTABLISHED"
Real-life scenario: While investigating a potential data exfiltration incident, Anya used this command to monitor active network connections in real-time, catching an unauthorized connection to an external server.

This updates every second to show all established TCP and UDP connections, helping you monitor network activity.

85. Check File Integrity
find /etc -type f -exec md5sum {} \; > /tmp/checksum.txt
Real-life scenario: After a suspected intrusion, Gabriel created a baseline of file checksums. When comparing against this baseline a week later, he discovered that several configuration files had been subtly modified.

This creates MD5 checksums for all files in /etc, which you can compare later to detect unauthorized changes.

86. SSL Certificate Expiry Check
echo | openssl s_client -servername example.com -connect example.com:443 2>/dev/null | openssl x509 -noout -enddate
Real-life scenario: As part of a monthly security check, Zoe set up an automated script using this command to check for certificates approaching expiration, preventing unexpected outages due to expired certificates.

This shows the expiration date of an SSL certificate for a website, helping you avoid service disruptions due to expired certificates.

87. Find Writable Directories
find / -type d -writable -not -path "/proc/*" 2>/dev/null
Real-life scenario: While preparing to enable a stricter security policy, Raj needed to identify directories that were world-writable. This command helped him find several unexpected locations that needed permission adjustments.

This finds directories that are writable by the current user, which is useful for identifying potential security risks.

88. Check User Account Security
awk -F: '($2 == "") {print $1}' /etc/shadow
Real-life scenario: During an audit of a newly acquired system, Isabella checked for accounts without passwords and discovered an old service account that had been configured insecurely.

This finds user accounts without passwords, which is a serious security risk.

89. Monitor Suspicious Processes
ps aux | grep -v "^$(whoami)\|^root\|^system"
Real-life scenario: While investigating unusual system behavior, Daniel used this command to focus on processes not owned by himself, root, or system users, helping him identify a cryptominer running under an obscure user account.

This shows processes not owned by the current user, root, or system accounts, which might indicate unauthorized activity.

90. Apply Security Updates
sudo apt update && sudo apt upgrade -y
Real-life scenario: After a critical vulnerability was announced, Elena immediately ran this command on all servers to apply security updates, mitigating the risk before attackers could exploit it.

This updates the package lists and installs available upgrades, which is essential for maintaining security.

Performance Prowess: Making Things Blazing Fast
91. Find CPU-Intensive Processes
ps -eo pcpu,pid,user,args | sort -r | head -10
Real-life scenario: During a performance investigation, Noah needed to identify which processes were consuming the most CPU. This command revealed that a log parsing script was unexpectedly using 90% of CPU resources.

This lists the top 10 processes by CPU usage, showing the percentage, process ID, user, and command.

92. Find Memory Leaks
watch -n 5 "ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head"
Real-life scenario: Users reported that the application became slower the longer it ran. Using this command, Leila monitored memory usage over time and identified a component that was steadily consuming more memory, confirming a memory leak.

This updates every 5 seconds to show the top memory-consuming processes, helping you identify memory leaks.

93. Identify Disk I/O Bottlenecks
iostat -xz 1
Real-life scenario: A database server was experiencing slow query performance despite having plenty of CPU and memory. This command helped Marco identify that the disk I/O was the bottleneck, leading to a storage upgrade.

This shows detailed disk I/O statistics updated every second, helping you identify I/O bottlenecks.

94. Check Network Throughput
iftop -i eth0
Real-life scenario: After moving to a new cloud provider, Prisha needed to verify that the network performance met the expected specifications. This command helped her monitor actual network throughput during load tests.

This shows network usage by host pairs, updated in real-time, helping you identify bandwidth usage patterns.

95. Profile Application Performance
perf record -g -p [pid]
Real-life scenario: A critical service was using more CPU than expected. By profiling it with perf, Axel discovered that a specific function was being called excessively due to a configuration error.

This records performance data for a specific process, which can be analyzed to find bottlenecks.

96. Find File Access Patterns
sudo iotop -o
Real-life scenario: A system was experiencing unexplained disk activity. Using iotop, Chen discovered that a log rotation script was causing excessive I/O during peak hours, allowing him to reschedule it for off-peak times.

This shows disk I/O usage by processes, displaying only active processes, which helps identify what’s causing disk activity.

97. Analyze System Load Averages
sar -q | awk '$4 > 1.0'
Real-life scenario: To understand historical performance patterns, Wei analyzed system load data and discovered that the system consistently became overloaded every day at the same time, corresponding to an automated batch job.

This shows historical load average data where the load exceeds 1.0, helping identify periods of high system load.

98. Clear Cache to Test Performance
sudo sh -c "sync; echo 3 > /proc/sys/vm/drop_caches"
Real-life scenario: While benchmarking a new storage solution, Olivia needed to ensure that disk caching wasn’t skewing the results. This command cleared the cache between test runs, providing more accurate measurements.

This synchronizes cached writes to disk and clears the kernel cache, which is useful for performance testing.

99. Find Slow Database Queries
mysql -e "SHOW FULL PROCESSLIST" | grep -v Sleep
Real-life scenario: Users reported that the application sometimes became unresponsive. Using this command, Raoul identified several inefficient queries that were running for minutes, blocking other operations.

This shows all active MySQL queries, excluding sleeping connections, which helps identify slow or problematic queries.

100. Horizontal Load Testing
ab -n 1000 -c 50 http://localhost/api/endpoint
Real-life scenario: Before launching a major campaign, Thea needed to ensure the API could handle the expected load. Using ApacheBench, she identified a bottleneck in the database connection pool that would have caused problems during the launch.

This sends 1000 requests with 50 concurrent connections to the specified URL, helping you test how your application handles load.

Conclusion: From Survival to Mastery
As our story began with Sarah facing that dreaded 3 AM alert, it’s worth noting how her journey reflects the path many of us take in DevOps. With each alert, each incident, and each successful resolution, we add tools to our arsenal and wisdom to our approach.

The 100 commands, scripts, and hacks in this guide represent more than just technical knowledge, they represent real solutions to real problems that DevOps engineers face every day. By understanding not just the syntax but the context in which these tools shine, you transform from someone who merely survives incidents to someone who prevents them.

Remember that mastery in DevOps isn’t about memorizing commands, it’s about developing intuition for which tool fits which problem. Start with the basics in this guide, practice them in your environment, and gradually expand your expertise. Before long, you’ll find yourself confidently navigating the complex world of modern infrastructure, just like Sarah did as the sun began to rise after her night of troubleshooting.

The next time your phone buzzes at 3 AM, you’ll still feel that knot in your stomach, we all do but you’ll also feel something else: confidence. And that makes all the difference.

Creating Your Own DevOps Toolkit
As a final tip, consider creating your own personalized “toolkit” script that puts your most-used commands in one place. Here’s a simple example to get you started:

#!/bin/bash
function disk_space() {
    df -h
}
function memory_usage() {
    free -h
}
function cpu_load() {
    uptime
}
function docker_status() {
    docker ps -a
}
function kube_pods() {
    kubectl get pods --all-namespaces
}
# Display menu
PS3="Select command to run: "
select cmd in "Disk Space" "Memory Usage" "CPU Load" "Docker Status" "Kubernetes Pods" "Exit"; do
    case $cmd in
        "Disk Space") disk_space ;;
        "Memory Usage") memory_usage ;;
        "CPU Load") cpu_load ;;
        "Docker Status") docker_status ;;
        "Kubernetes Pods") kube_pods ;;
        "Exit") break ;;
        *) echo "Invalid option" ;;
    esac
done
Save this as ~/toolkit.sh, make it executable with chmod +x ~/toolkit.sh, and customize it with your most frequently used commands. This approach not only saves time but also helps you internalize the commands you're using regularly.
