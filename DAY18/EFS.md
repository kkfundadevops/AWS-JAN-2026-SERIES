# Amazon EFS Setup with Two Apache Web Servers (EC2)

This guide explains how to configure Amazon Elastic File System (EFS) to be used as shared storage between two EC2 instances running Apache HTTP Server. The goal is to serve the same web content from both instances using EFS.

---

## What is Amazon EFS?

Amazon Elastic File System (EFS) is a fully managed, scalable, and elastic NFS (Network File System) that can be mounted simultaneously on multiple EC2 instances. It provides shared access to data with high availability and durability across multiple Availability Zones (AZs). EFS is ideal for scenarios that require a centralized file store accessible from multiple compute resources.

### Key Features:

* Supports NFSv4 protocol
* Accessible from multiple AZs in the same region
* Automatically scales storage as files are added or removed
* No need to provision or manage capacity
* Data is stored redundantly across multiple AZs
* Integrated with AWS Identity and Access Management (IAM)
* Can be mounted on Linux-based systems (not Windows)

---

## Use Case Overview

Two web servers in different Availability Zones serve the same web content using a shared EFS file system. This approach is commonly used in CMS platforms, multi-AZ deployments, and highly available web applications.

---

## Step 1: Launch EC2 Instances

1. Launch two EC2 instances in the same VPC, each in a different public subnet.
2. Create or use a security group that allows:

   * Port 22 (SSH) from your IP
   * Port 80 (HTTP) from anywhere (0.0.0.0/0)
   * Port 2049 (NFS) from the same security group
3. Connect to each instance using SSH.

---

## Step 2: Install Apache and EFS Utilities

Run these commands on both instances:

```bash
sudo yum update -y
sudo yum install -y httpd amazon-efs-utils
sudo systemctl enable httpd
sudo systemctl start httpd
```

---

## Step 3: Create EFS File System

1. Go to the Amazon EFS console and click "Create file system."
2. Use the same VPC as your EC2 instances.
3. Under Customize, create mount targets in the subnets where your EC2 instances are launched.
4. Attach the same security group used by EC2s.
5. Complete creation and note the File System ID (e.g., `fs-0f6ae9ddbe0eeb2b5`).

---

## Step 4: Mount EFS to Apache Web Directory

1. On both EC2 instances, mount EFS to `/var/www/html`:

```bash
sudo mount -t efs -o tls fs-0f6ae9ddbe0eeb2b5:/ /var/www/html
```

2. Add a sample index page on one instance:

```bash
echo "<h1>Shared via EFS</h1>" | sudo tee /var/www/html/index.html
```

3. On the second instance, verify the content:

```bash
cat /var/www/html/index.html
```

---

## Step 5: Access the Web Servers

1. Visit both EC2 public IP addresses in your browser:

```
http://<EC2-Instance-1-Public-IP>
http://<EC2-Instance-2-Public-IP>
```

Both should display the same web page, confirming shared EFS.

---

## Step 6: Make the Mount Persistent

1. On each EC2 instance, open `/etc/fstab` and add:

```bash
fs-0f6ae9ddbe0eeb2b5:/ /var/www/html efs defaults,_netdev 0 0
```

2. Reload fstab:

```bash
sudo mount -a
```

---

## Benefits of Using EFS for Web Servers

* Simplifies content synchronization across servers
* Centralizes storage management
* Reduces administrative overhead
* Provides fault tolerance and high availability by default
* Easily integrates with Auto Scaling and Load Balancers

---

## Summary

| Component       | Purpose                                |
| --------------- | -------------------------------------- |
| EC2 Instances   | Host the Apache web server             |
| Apache HTTPD    | Serves content from /var/www/html      |
| Amazon EFS      | Provides shared file storage           |
| NFS Port (2049) | Protocol used by EFS for communication |
| Mount Path      | `/var/www/html`                        |

This approach allows dynamic web content to be shared between multiple instances without manual sync or duplication of data.
