# AWS EBS (Elastic Block Store) – Complete In-Depth Guide

---

## 1. What is Amazon EBS?

**Amazon Elastic Block Store (EBS)** is a **block-level storage service** designed to be attached to **EC2 instances**.

Key characteristics:

* Acts like a **virtual hard disk**
* Used for **OS disks and application data**
* Data persists **independently of EC2 lifecycle**
* Can be detached and attached to another instance (within same AZ)

EBS is tightly integrated with EC2 and is one of the **most critical services** in AWS.

---

## 2. How Do We Measure EBS Volume Performance?

EBS performance is evaluated using **four main metrics**.

---

### 2.1 IOPS (Input/Output Operations Per Second)

IOPS represents:

* Number of read/write operations a volume can handle per second

Important:

* Small I/O operations → higher IOPS demand
* Databases are **IOPS-sensitive**

Example:

* 1,000 IOPS = 1,000 read/write operations per second

---

### 2.2 Block Size

Block size is the **amount of data transferred per I/O operation**.

Common block sizes:

* 4 KB
* 8 KB
* 16 KB

Impact:

* Smaller block size → more IOPS
* Larger block size → fewer IOPS, higher throughput

---

### 2.3 Latency

Latency is:

* Time taken to complete **one I/O operation**

Measured in:

* milliseconds (ms)

Lower latency = faster response
Critical for:

* Databases
* Transaction systems
* Real-time applications

---

### 2.4 Throughput

Throughput is:

* Rate at which data is transferred

Measured in:

* MB/s
* GB/s

Important for:

* Large file processing
* Data analytics
* Streaming workloads

---

### 2.5 Relationship Between IOPS, Throughput, and Block Size

A simplified formula:

```
IOPS = (Throughput × 1024) / Block Size (KB)
```

Meaning:

* Increasing block size increases throughput but reduces IOPS
* Workload type decides which metric matters more

---

## 3. EBS Volume Types – High-Level Classification

EBS volumes are broadly classified into:

* **SSD-backed volumes**
* **HDD-backed volumes**
* **Legacy magnetic volumes**

Each type is optimized for **specific workloads**.

---

## 4. Root Volume vs Additional Volumes

### Root Volume

* Contains the **Operating System**
* Mandatory for launching an EC2 instance
* Without root volume → instance cannot boot

Supported root volume types:

* gp2
* gp3
* io1
* io2

---

### Additional (Data) Volumes

* Used for application data, logs, databases
* Can be attached/detached independently

Supported additional volume types:

* gp2, gp3
* io1, io2
* st1, sc1
* standard (magnetic)

---

## 5. SSD-Based EBS Volumes

SSD volumes are optimized for **low latency and random I/O**.

---

### 5.1 General Purpose SSD – gp2 & gp3

These are the **most commonly used EBS volumes**.

#### gp2

* Balanced performance
* Performance scales with size
* Suitable for:

  * Boot volumes
  * Web servers
  * Dev/Test workloads

Size range:

* 1 GB to 16 TB

---

#### gp3 (Recommended)

* Decouples storage size from performance
* You can independently configure:

  * IOPS
  * Throughput

Size range:

* 1 GB to 64 TiB

Why gp3 is better:

* More predictable performance
* Lower cost
* Better control

---

### 5.2 Provisioned IOPS SSD – io1 & io2

Used when **exact IOPS guarantees** are required.

Suitable for:

* Databases
* High-transaction systems
* Mission-critical workloads

---

#### io1

* Provision specific IOPS
* Good for consistent high performance
* Size range: 4 GB to 16 TB

---

#### io2

* Higher durability
* Higher IOPS capability
* Designed for critical production systems

Key characteristics:

* Minimum IOPS: 100
* Maximum IOPS: up to 100,000
* Size range: 4 GB to 64 TiB

---

## 6. HDD-Based EBS Volumes

HDD volumes are optimized for **high throughput**, not random IOPS.

---

### 6.1 Throughput Optimized HDD – st1

Best suited for:

* Big data workloads
* Log processing
* Data warehousing
* Streaming large files

Characteristics:

* High MB/s throughput
* Not suitable for boot volumes
* Lower cost than SSD

Size range:

* 125 GB to 16 TB

---

### 6.2 Cold HDD – sc1

Designed for:

* Infrequently accessed data
* Cold storage
* Cost-sensitive workloads

Characteristics:

* Lowest performance
* Lowest cost
* Sequential access only

Size range:

* 125 GB to 16 TB

---

## 7. Magnetic (Standard) Volumes

Legacy storage option.

Characteristics:

* Lowest cost
* Lowest performance
* Rarely used today

Size range:

* 1 GB to 1 TB

Used only for:

* Legacy systems
* Very low-cost requirements

---

## 8. Availability Zone Limitation (Very Important)

An EBS volume:

* Exists in **one specific Availability Zone**
* Can be attached only to EC2 instances in the **same AZ**

If AZ differs:

* Volume cannot be attached
* Snapshot must be used to move data across AZs

---

## 9. Linux Commands to Work with EBS Volumes

### 9.1 Detect Attached Volumes

```bash
lsblk
```

Shows:

* All block devices detected by OS
* Even unmounted volumes

Example volume names:

* `/dev/nvme0n1` → root volume
* `/dev/nvme1n1` → additional volume

---

### 9.2 View Mounted File Systems

```bash
df -Th
```

Shows:

* Mounted volumes only
* File system type
* Human-readable sizes

---

### 9.3 Check File System Presence

```bash
file -s /dev/nvme1n1
```

Output meaning:

* `data` → no file system
* `XFS` / `ext4` → file system exists

---

## 10. Creating File System and Mounting (Linux)

### Create XFS File System

```bash
mkfs -t xfs /dev/nvme1n1
```

---

### Mount the Volume

```bash
mount /dev/nvme1n1 /home/ec2-user/newvol
```

Temporary mount:

* Stored in `/etc/mtab`
* Lost after reboot

---

### Permanent Mount Configuration

1. Copy mount entry
2. Add it to `/etc/fstab`

Example:

```
mount /dev/nvme1n1 /home/ec2-user/newvol
tail -1 /etc/matb
sudo vi /etc/fstab
```

Activate:

```bash
mount -a
```

---

## 11. Expanding an Existing EBS Volume

### Step 1: Increase Volume Size (AWS Console)

* Modify volume
* Increase size (cannot decrease)

---

### Step 2: Expand File System (Linux – XFS)

```bash
xfs_growfs -d /home/ec2-user/newvol
```

This updates file system to use new space.

---

## 12. Expanding Root Volume (Linux)

### Step 1: Increase root volume size in AWS Console

---

### Step 2: Identify Root Partition

Usually:

* Volume: `/dev/nvme0n1`
* Partition: `1`

---

### Step 3: Grow Partition

```bash
growpart /dev/nvme0n1 1
```

---

### Step 4: Reboot Instance

```bash
reboot
```

File system expands after reboot.

---

## 13. Important Limitation (Must Tell Students)

> **AWS does NOT support reducing EBS volume size.**

If you need smaller size:

* Create new volume
* Copy data
* Replace old volume

---

## 14. Best Practices (Real-World)

* Use **gp3** by default
* Use **io2** for databases
* Never put DB on sc1 or st1
* Keep root volume separate from data
* Always tag volumes
* Take snapshots before resizing
* Monitor IOPS and throughput in CloudWatch

---

## 15. Summary

EBS is not just “disk” — it is:

* Performance-driven
* AZ-scoped
* Workload-specific
* Deeply tied to EC2 behavior

Understanding:

* Volume types
* Performance metrics
* OS-level operations
* Expansion logic

is mandatory for **real AWS engineers**.

