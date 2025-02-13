---
title: How to Install Jenkins on Ubuntu 22.04 A Step-by-Step Guide
date: 2023-10-04 09:54:23 +/-TTTT
categories: [Jenkins]
tags: [jenkins, ubuntu22]     # TAG names should always be lowercase
toc: true
comments: true
---
# How to Install Jenkins on Ubuntu 22.04: A Step-by-Step Guide

![jenkins-logo](/assets/img/install-jenkins-ubuntu22/Jenkins_logo.png)
![ubuntu-logo](/assets/img/install-jenkins-ubuntu22/ubuntu_logo.png)

Jenkins is a widely used open-source automation server that simplifies the process of continuous integration and continuous delivery (CI/CD). It allows developers to automate the building, testing, and deployment of their applications, making the software development process more efficient and reliable. In this tutorial, we will walk you through the steps to install Jenkins on Ubuntu 22.04.

## Prerequisites

Before we start, make sure you have the following prerequisites in place:

1. **Ubuntu 22.04 Server**: You should have a clean installation of Ubuntu 22.04 up and running.

2. **Java**: Jenkins requires Java to run. You can install it using the following command:
   
   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk
   ```

3. **A user with sudo privileges**: It's a good practice to perform administrative tasks with a non-root user that has sudo privileges. You can follow our [guide on creating a sudo user](https://www.example.com/how-to-create-a-sudo-user-on-ubuntu-22-04) if needed.

## Step 1: Update System Packages

Before installing Jenkins, it's essential to ensure that your system's package list is up to date. Open a terminal and run:

```bash
sudo apt update
```

This command will fetch the latest package information from the Ubuntu repositories.

## Step 2: Install Jenkins

To install Jenkins on Ubuntu 22.04, you need to add the Jenkins repository to your system and then install Jenkins from there. Follow these commands:

```bash
# Add the Jenkins GPG key to the system
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

# Add the Jenkins repository to the sources list
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Update the package list once more
sudo apt update

# Install Jenkins
sudo apt install jenkins
```

During the installation, you'll be prompted to confirm adding the Jenkins repository. Select 'Yes' and proceed.

## Step 3: Start and Enable Jenkins Service

After Jenkins is successfully installed, start the service and enable it to start automatically on system boot:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

You can check the status of the Jenkins service to ensure that it's running:

```bash
sudo systemctl status jenkins
```

If everything is set up correctly, you should see an output indicating that Jenkins is active and running.

## Step 4: Configuring Jenkins

Jenkins, by default, runs on port 8080. You can access the Jenkins web interface by opening a web browser and navigating to `http://your_server_ip:8080`.

To unlock Jenkins, you will need to retrieve the initial admin password. You can find it in the Jenkins server's log. Use the following command to display the password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it into the Jenkins web interface to unlock it.

Follow the on-screen instructions to complete the Jenkins setup, including installing recommended plugins and creating an admin user.

## Step 5: Accessing Jenkins

Once the setup is complete, you can log in to Jenkins using the admin credentials you just created. You will now have full access to Jenkins and can start configuring and using it for your CI/CD needs.

## Conclusion

In this tutorial, we've walked you through the process of installing Jenkins on Ubuntu 22.04. Jenkins is a powerful tool that can greatly enhance your software development process by automating repetitive tasks and streamlining your CI/CD pipeline. With Jenkins up and running, you're now ready to explore its features and begin automating your development workflow.