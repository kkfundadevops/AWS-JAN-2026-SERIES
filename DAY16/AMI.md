# Golden AMI Demo Guide (Amazon Linux)

## Build Base Instance → Create AMI → Launch New Instance → Verify

---

## 1. What is a Golden AMI?

A **Golden AMI** is a **pre-configured Amazon Machine Image** that contains:

* OS + security baseline
* Required packages and tools
* Application/runtime setup (example: Apache)
* Common user accounts / configs (if needed)
* Hardening and standard configuration

### Why companies use Golden AMI

* Faster server provisioning (ready-to-use instances)
* Consistent configuration across environments
* Less manual work for every new instance
* Better compliance and standardization

### What a Golden AMI actually captures

When you create an AMI from an EC2 instance (EBS-backed):

* AWS takes **snapshots of the attached EBS volumes**
* Creates an AMI that references those snapshots
* New instances launched from that AMI start with **the same disk contents**

---

## 2. Build the Base Instance (Amazon Linux)

### 2.1 Launch EC2 (Base)

* AMI: **Amazon Linux 2** or **Amazon Linux 2023** (either is okay)
* Instance type: `t2.micro` / `t3.micro` for demo
* Key pair: create/select
* Security Group:

  * SSH: **22** (My IP recommended)
  * HTTP: **80** (Anywhere for web demo)
* Tags (recommended):

  * Name: `golden-ami-base`
  * Project: `GoldenAMI`
  * Platform: `Linux`

---

## 3. Install Required Packages + Apache + Sample HTML

SSH into instance:

```bash
ssh -i <your-key>.pem ec2-user@<PUBLIC_IP>
```

Switch to root (optional but common for setup):

```bash
sudo su -
```

### 3.1 Update OS

Amazon Linux 2:

```bash
yum update -y
```

Amazon Linux 2023:

```bash
dnf update -y
```

### 3.2 Install tools: tree, zip, unzip

Amazon Linux 2:

```bash
yum install -y tree zip unzip
```

Amazon Linux 2023:

```bash
dnf install -y tree zip unzip
```

Verify:

```bash
tree --version || tree
zip -v | head
unzip -v | head
```

### 3.3 Install Apache (httpd)

Amazon Linux 2:

```bash
yum install -y httpd
```

Amazon Linux 2023:

```bash
dnf install -y httpd
```

Enable and start:

```bash
systemctl enable httpd
systemctl start httpd
systemctl status httpd --no-pager
```

### 3.4 Add Sample HTML Page

Create/replace default page:

```bash
cat > /var/www/html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
  <title>KK FUNDA - Golden AMI Demo</title>
</head>
<body>
  <h1>Golden AMI Demo - Apache is Working</h1>
  <p>Server built from a Golden AMI.</p>
  <p><b>Host:</b> <span id="host"></span></p>
  <p><b>Date:</b> <span id="date"></span></p>

  <script>
    document.getElementById("host").innerText = window.location.hostname;
    document.getElementById("date").innerText = new Date().toString();
  </script>
</body>
</html>
EOF
```

Set permissions (safe defaults):

```bash
chmod 644 /var/www/html/index.html
```

Test locally on server:

```bash
curl -I http://localhost
curl http://localhost | head
```

Now test from browser:

* Open: `http://<PUBLIC_IP>`

---

## 4. Create Two Users + Enable Password Login

### 4.1 Create Users

Example users:

* `kkfunda1`
* `kkfunda2`

Create:

```bash
useradd kkfunda1
useradd kkfunda2
```

Set passwords:

```bash
passwd kkfunda1
passwd kkfunda2
```

(Optional) Give sudo access (Amazon Linux typically uses `wheel` group):

```bash
usermod -aG wheel kkfunda1
```

Verify:

```bash
id kkfunda1
id kkfunda2
getent group wheel | grep kkfunda1 || true
```

---

### 4.2 Enable SSH Password Authentication

Edit SSH config:

```bash
sudo vi /etc/ssh/sshd_config
```

Set / ensure these values (add them if missing):

```
PasswordAuthentication yes
PermitRootLogin no
```

If you see `PasswordAuthentication no`, change to `yes`.

Restart SSH service:

```bash
sudo systemctl restart sshd
sudo systemctl status sshd --no-pager
```

---

### 4.3 Test Password Login

From your laptop:

```bash
ssh kkfunda1@<PUBLIC_IP>
```

It should ask for password.

**Important security note (tell in class):**

* Password login is okay for demo/training,
* In production, prefer **key-based auth + SSM** and restrict SSH by IP.

---

## 5. Create AMI from the Instance

### 5.1 What happens when you create AMI

* AWS creates EBS snapshot(s)
* Builds an AMI from those snapshots
* You can launch identical servers later

### 5.2 Create AMI from Console

EC2 Console → Instances → select base instance →
Actions → Image and templates → **Create image**

Provide:

* Image name: `kkfunda-golden-ami-v1`
* Image description: `Apache + tools + users + password login enabled`

### 5.3 Should the instance be Stopped or Running?

You have two options:

#### Option A: Create AMI while instance is Running (No downtime)

* Faster for demo
* AWS may briefly **freeze filesystem** to keep snapshot consistent
* For simple Apache demo, this is fine

#### Option B: Stop the instance first (Best consistency)

* More reliable snapshot consistency
* Recommended for production golden images

**Teaching line:**

* “Running AMI creation is okay for normal workloads, but for strict consistency we stop the instance.”

### 5.4 “No reboot” option

During AMI creation, there is usually an option like **No reboot**:

* If enabled, AWS won’t reboot the instance
* But snapshot consistency is your responsibility

**Recommendation:**

* For training: you can use No reboot (to save time)
* For production: allow reboot or stop instance for best image consistency

---

## 6. Launch New Instance From AMI and Prove Everything is Copied

### 6.1 Launch Instance from AMI

EC2 Console → AMIs → select `kkfunda-golden-ami-v1` → **Launch instance from AMI**

While launching:

* Same SG ports:

  * 22 (My IP)
  * 80 (Anywhere)
* Choose key pair (still needed for emergency access)
* Tags:

  * Name: `golden-ami-server-01`

---

## 7. Verification Checklist (Show in Demo)

### 7.1 Verify Apache is already installed and running

SSH to the new instance:

```bash
ssh -i <your-key>.pem ec2-user@<NEW_PUBLIC_IP>
```

Check Apache:

```bash
sudo systemctl status httpd --no-pager
curl -I http://localhost
```

Browser check:

* `http://<NEW_PUBLIC_IP>`
  You should see your **same HTML page**.

---

### 7.2 Verify tools are installed

```bash
tree
zip -v | head
unzip -v | head
```

---

### 7.3 Verify users are copied

```bash
getent passwd kkfunda1
getent passwd kkfunda2
```

Test password login to the new instance:

```bash
ssh kkfunda1@<NEW_PUBLIC_IP>
```

---

## 8. What is NOT copied (Important to explain)

Even with a Golden AMI, some things may change per instance:

* **Public IP** changes (unless Elastic IP)
* **Hostname / instance-id** changes
* **SSH host keys** can be regenerated depending on OS/image settings
* **Instance metadata** always differs
* Anything configured via **User Data** is not “inside AMI” unless you baked it in

---

## 9. Best Practices for a Real Golden AMI (Optional but valuable)

Before creating final AMI in production:

* Remove secrets (API keys, tokens, passwords)
* Clean temporary files and logs
* Patch OS packages
* Use SSM Agent (for secure management)
* Use least privilege IAM role
* Standard tagging and monitoring agents



If you want, I can also give you a **single automation shell script** that does the full base setup (install tools + Apache + HTML + users + ssh config) so you can run it once and then create the AMI.
