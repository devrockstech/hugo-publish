---
title: Installing Jenkins
weight: 1
---
# System Setup for Jenkins Installation

## Update System Packages (Optional)
It's a good practice to update your system's package repository and installed packages:
```bash
sudo apt update
sudo apt upgrade
```
## Install Java
Jenkins is a Java-based application, so you need to have Java installed. You can install OpenJDK with the following command:

sudo apt install openjdk-11-jdk

## Add Jenkins Repository and Install Jenkins
Add the Jenkins repository key to your system:

```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```

Append the Jenkins Debian package repository address to the server's sources.list:

```bash
echo "deb https://pkg.jenkins.io/debian-stable binary/" | sudo tee -a /etc/apt/sources.list
```

Update your package index:

```bash
sudo apt update
```

Install Jenkins:

```bash
sudo apt install jenkins
```

## Start and Enable Jenkins Service

Start the Jenkins service:

```bash
sudo systemctl start jenkins
```

Enable the Jenkins service to start on boot:

```bash
sudo systemctl enable jenkins
```