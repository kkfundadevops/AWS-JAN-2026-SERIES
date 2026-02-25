## AWS Load Balancer & EC2 Scaling – Full  Notes

This document explains the concepts of Load Balancing and EC2 Scaling, followed by a practical setup of an **Application Load Balancer (ALB)** in a VPC environment with two or more EC2 instances.

---

## What is a Load Balancer?

A **Load Balancer** in AWS is a service that automatically distributes incoming application traffic across multiple targets (like EC2 instances, containers, or IPs) in one or more Availability Zones. It helps achieve **high availability**, **fault tolerance**, and **horizontal scalability**.

---

## Types of AWS Load Balancers

1. **Application Load Balancer (ALB)**

   * Works at **Layer 7** (HTTP/HTTPS)
   * Supports **host-based and path-based routing**
   * Good for web apps, REST APIs, microservices

2. **Network Load Balancer (NLB)**

   * Works at **Layer 4** (TCP)
   * Very high performance, low latency
   * Suitable for real-time apps, gaming, VoIP

3. **Classic Load Balancer (CLB)** (Legacy)

   * Works at both Layer 4 and Layer 7
   * Not recommended for new deployments

4. **Gateway Load Balancer (GWLB)**

   * Designed for **third-party virtual appliances**
   * Handles security appliances like firewalls, intrusion detection

---

## Scaling EC2 Instances

### Vertical Scaling

* Increase the **instance size** (e.g., `t2.micro` → `t3.large`)
* Requires **stopping the instance** before changing type
* Used when single instance needs more power (RAM/CPU)
* Not fault-tolerant (still a single point of failure)

#### Steps:

1. Stop EC2 instance
2. Modify instance type under **Actions → Instance Settings → Change Instance Type**
3. Start the instance again

### Horizontal Scaling

* Add **more EC2 instances** and place them behind a Load Balancer
* Distributes traffic across instances
* Increases availability and fault tolerance
* Works well with **Auto Scaling Groups**

---

## Scenario: Load Balancer with Two Web Servers (httpd)

### Step 1: Launch Two EC2 Instances (Web Servers)

1. Use Amazon Linux 2
2. Install Apache (HTTPD)

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl enable httpd
sudo systemctl start httpd
```

3. Customize index.html:

```bash
echo "<h1>Web Server 1</h1>" | sudo tee /var/www/html/index.html
```

Repeat on second instance:

```bash
echo "<h1>Web Server 2</h1>" | sudo tee /var/www/html/index.html
```

### Step 2: Security Group for EC2

* Allow **port 80 (HTTP)** from anywhere (0.0.0.0/0)

---

## Step 3: Create an Application Load Balancer (ALB)

### 1. Go to EC2 → Load Balancers → Create Load Balancer

### 2. Choose "Application Load Balancer"

### 3. Basic Configuration:

* Name: `my-alb`
* Scheme: Choose **Internet-facing** (for public access)
* IP type: IPv4

### 4. Network Mapping:

* Select appropriate VPC
* **Choose at least 2 subnets**, each from a different Availability Zone

### 5. Security Groups:

* Create or select a Security Group with **port 80 open** (inbound rule)

### 6. Listeners and Routing:

* Listener protocol: HTTP (port 80)
* Default action: Forward to **Target Group** (create a new one below)

---

## Step 4: Create a Target Group (TG)

### 1. Choose Target Type: **Instance**

### 2. Protocol: HTTP, Port: 80

### 3. Health Checks:

* Path: `/`
* Protocol: HTTP
* Healthy threshold: 3
* Unhealthy threshold: 2

### 4. Register Targets:

* Select the two EC2 instances you launched
* Click **Include as pending below**

### 5. Create Target Group

---

## Step 5: Complete Load Balancer Creation

* Attach the newly created Target Group to the Load Balancer
* Review and click **Create Load Balancer**

### After creation:

* Copy the **DNS name** of the ALB
* Open in browser:

```
http://<ALB-DNS-name>
```

* You will see Web Server 1 or Web Server 2 randomly (based on routing)

---

## Step 6: Sticky Sessions (Session Affinity)

* Sticky Sessions ensure that the same client is routed to the same target
* You can enable it in the **Target Group settings**

### To Enable Stickiness:

1. Go to EC2 Console → Target Groups
2. Select your Target Group → Edit Attributes
3. Enable **Stickiness**
4. Set duration (e.g., 300 seconds)

Disable the same way if needed.

---

## Step 7: Add Another Target Group and Listener (Port 8080)

1. Launch a third EC2 instance → Install HTTPD
2. Update index.html:

```bash
echo "<h1>Third Server (8080)</h1>" | sudo tee /var/www/html/index.html
```

3. Create new Target Group:

* Name: `tg-8080`
* Protocol: HTTP, Port: 8080
* Register the third EC2 instance

4. Add new Listener:

* Go to Load Balancer → Listeners → Add Listener
* Port: 8080
* Forward to `tg-8080`

5. Open in browser:

```
http://<ALB-DNS-name>:8080
```

You’ll see “Third Server (8080)” page

---

## Additional  Points

* Load balancers **auto-distribute** traffic based on health
* Health checks ensure only healthy targets receive requests
* Sticky sessions are useful for session-based apps
* Port-based routing enables hosting multiple apps on the same ALB
* Use **Auto Scaling** to dynamically register/unregister instances

---

This completes a real-world ALB configuration with multiple EC2s, horizontal scaling, stickiness, and custom listeners. This approach mimics what companies use for high availability and traffic management.
