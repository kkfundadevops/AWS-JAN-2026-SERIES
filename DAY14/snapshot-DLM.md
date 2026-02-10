# Amazon EBS Snapshots – Deep Explanation

### What is an EBS Snapshot?

* An EBS Snapshot is a **point-in-time backup** of an Amazon EBS volume.
* Stored **internally in Amazon S3** (not visible in your buckets).
* Supports **incremental backups**: only changed blocks are saved after the first snapshot.
* Used for restoring volumes, creating new volumes, sharing, and disaster recovery.

---

## Manual Snapshot Creation (Step-by-Step)

1. Go to AWS EC2 Console → Elastic Block Store → Volumes.
2. Select the EBS volume you want to back up.
3. Click Actions → Create Snapshot.
4. Enter a name and description.
5. Click Create Snapshot.

You can monitor snapshot progress under **Snapshots** in the left-hand menu.

---

## Snapshot Use Case Scenarios (with Diagrams)

### Case 1: Restore Snapshot in Another AZ

* Snapshot is created from a volume in AZ `ap-south-1a`.
* A new volume is created in AZ `ap-south-1b` using that snapshot.
* Attach the volume to an EC2 instance in the new AZ.

Use this when migrating workloads or preparing high availability setups.

### Case 2: Copy Snapshot to Another Region

* Snapshot created in Mumbai (`ap-south-1`) is copied to Singapore (`ap-southeast-1`).
* From the copied snapshot, create a new volume in Singapore.
* Launch a new EC2 instance in Singapore and attach the volume.

Use this for disaster recovery and multi-region deployment.

### Case 3: Share Snapshot Across AWS Accounts

* Snapshot created in Account 1 (e.g., `ap-south-1a`).
* Share the snapshot using the **12-digit AWS account ID** of Account 2.
* Account 2 copies the snapshot and creates a new volume.

Use this for collaboration or migrating workloads between accounts.

### Case 4: Share Snapshot Publicly

* A snapshot is made public.
* Any AWS user worldwide can copy the snapshot and use it.

Used for sharing templates, demo volumes, public datasets. Never do this with sensitive data.

---

## Encrypted and Unencrypted Snapshot Scenarios

### Encrypted Volume → Encrypted Snapshot

* If you take a snapshot of an encrypted volume, the snapshot is also encrypted.
* When you create a volume from that snapshot, it will stay encrypted.

### Unencrypted Volume → Snapshot → Copy with Encryption

* Take snapshot of an unencrypted volume.
* Copy that snapshot and choose "Enable Encryption" during the copy.
* Resulting snapshot and volume become encrypted.

Encryption uses the default AWS-managed KMS key or a custom key if specified.

---

## Where Are Snapshots Stored?

* Snapshots are stored in Amazon S3 but not visible like S3 buckets or files.
* Region-bound by default. Must be copied explicitly to other regions.
* Highly durable — 99.999999999% (11 9s) reliability.
* You are billed per GB-month of snapshot storage.

---

## Automating Snapshots with Data Lifecycle Manager (DLM)

### Steps to Create a DLM Policy

1. Go to EC2 Console → Lifecycle Manager (under EBS).
2. Click Create Lifecycle Policy.
3. Choose EBS Snapshot Policy.
4. Target EBS volumes based on tags (example: Backup: Daily).
5. Set schedule (e.g., every 24 hours), and retention (e.g., keep last 7 snapshots).
6. Use or create an IAM role (e.g., AWSDataLifecycleManagerDefaultRole).
7. Click Create Policy.

### What Happens Next

* Snapshots are created automatically based on the schedule.
* Older snapshots are deleted based on the retention rule.

Use tags like `Environment=Prod`, `Backup=Daily` to control which volumes are included.

---

## Additional Student Notes

* Snapshots are incremental, but full copies can be restored anytime.
* Volumes can be restored to different instance types or AZs.
* You can create AMIs using snapshots, which can then launch EC2 instances.
* Always test snapshot restore and mount procedures for production systems.
* Consider using snapshot lifecycle automation to reduce manual effort and costs.

