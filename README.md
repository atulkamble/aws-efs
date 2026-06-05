# AWS EFS (Elastic File System) Lab 

## Key Exam & Interview Points

### 1. What is EFS?

* AWS managed **shared file storage** service.
* Uses **NFS protocol**.
* Can be mounted on **multiple EC2 instances simultaneously**.
* Supports **Linux-based systems**.
* Automatically scales storage up and down.

---

### 2. EFS vs EBS vs S3

| Feature             | EFS          | EBS           | S3             |
| ------------------- | ------------ | ------------- | -------------- |
| Storage Type        | File Storage | Block Storage | Object Storage |
| Mount on EC2        | Yes          | Yes           | No             |
| Multiple EC2 Access | Yes          | No (usually)  | Via API        |
| Protocol            | NFS          | Block Device  | HTTPS          |
| Auto Scaling        | Yes          | No            | Yes            |
| Multi-AZ            | Yes          | No            | Yes            |

**Remember:**

* EBS = One server storage.
* EFS = Shared storage.
* S3 = Object storage.

---

### 3. Port Required

```text
2049
```

Protocol:

```text
NFS
```

Security Group must allow:

```text
Inbound TCP 2049
```

---

### 4. Mount Target

A Mount Target is:

```text
Network endpoint created inside a subnet
```

Purpose:

```text
Allows EC2 instances to connect to EFS
```

**Important:**

* At least one Mount Target per AZ.
* Recommended to create Mount Targets in all AZs where EC2 instances exist.

---

### 5. Security Group Configuration

Attach same Security Group to:

* EC2 Instances
* EFS Mount Targets

Required Rules:

| Type | Port | Source         |
| ---- | ---- | -------------- |
| SSH  | 22   | My IP          |
| NFS  | 2049 | Security Group |

---

### 6. EFS DNS Format

```text
fs-xxxxxxxx.efs.region.amazonaws.com
```

Example:

```text
fs-0123456789abcdef0.efs.us-east-1.amazonaws.com
```

---

### 7. Mount Command

Standard:

```bash
sudo mount -t efs fs-xxxx:/ /efs
```

Alternative NFS Mount:

```bash
sudo mount -t nfs4 -o nfsvers=4.1 fs-xxxx.efs.us-east-1.amazonaws.com:/ /efs
```

---

### 8. Persistent Mount

Add entry in:

```bash
/etc/fstab
```

Example:

```text
fs-xxxx:/ /efs efs defaults,_netdev 0 0
```

Test:

```bash
sudo mount -a
```

---

### 9. Verify EFS Mount

```bash
df -h
```

```bash
mount | grep efs
```

```bash
lsblk
```

**Note:**

* EFS does NOT appear as a local disk in `lsblk`.
* EFS is mounted over the network.

---

### 10. Real-Time Synchronization Test

Server-1:

```bash
echo "Hello AWS EFS" > /efs/test.txt
```

Server-2:

```bash
cat /efs/test.txt
```

File appears instantly because both servers share the same storage.

---

### 11. Amazon Linux 2023 Package

Install EFS Utilities:

```bash
sudo yum install amazon-efs-utils -y
```

Verify:

```bash
rpm -qa | grep efs
```

---

### 12. Performance Modes

#### General Purpose (Default)

* Lowest latency
* Most workloads
* Web Applications
* CMS
* Shared Storage

#### Max I/O

* Higher throughput
* Higher latency
* Large-scale analytics

---

### 13. Throughput Modes

#### Elastic Throughput (Recommended)

Automatically scales throughput.

#### Provisioned Throughput

Manually define throughput.

---

### 14. Storage Classes

#### Standard

* Frequently accessed files.

#### Infrequent Access (IA)

* Lower cost.
* Retrieval charges apply.

#### Archive

* Lowest cost.
* Long-term retention.

---

### 15. Encryption

#### At Rest

```text
AWS KMS
```

#### In Transit

```bash
sudo mount -t efs -o tls fs-xxxx:/ /efs
```

Recommended:

```bash
sudo mount -t efs -o tls fs-xxxx:/ /efs
```

---

### 16. High Availability

EFS is:

✅ Multi-AZ

✅ Highly Available

✅ Managed by AWS

No manual replication required.

---

### 17. Common Troubleshooting Commands

Check Mount:

```bash
mount | grep efs
```

Check Port:

```bash
nc -zv EFS-DNS 2049
```

DNS Resolution:

```bash
nslookup EFS-DNS
```

Verify Mount Targets:

```bash
aws efs describe-mount-targets
```

---

### 18. Common Errors

#### Mount Timeout

Cause:

```text
Port 2049 blocked
```

Fix:

```text
Allow NFS traffic
```

---

#### DNS Resolution Failed

Cause:

```text
DNS Hostnames disabled in VPC
```

Fix:

```text
Enable DNS Hostnames and DNS Resolution
```

---

#### Access Denied

Cause:

```text
Security Group issue
```

Fix:

```text
Verify NFS rule on port 2049
```

---

### 19. EFS with Kubernetes (EKS)

Use:

```text
AWS EFS CSI Driver
```

Benefits:

* Shared Persistent Volumes
* Multiple Pods access same data
* Dynamic Provisioning

---

### 20. Important AWS Services Integration

| Service    | Support |
| ---------- | ------- |
| EC2        | Yes     |
| EKS        | Yes     |
| ECS        | Yes     |
| Lambda     | Yes     |
| AWS Backup | Yes     |
| DataSync   | Yes     |

---

## Quick Revision (Must Remember)

```text
Storage Type      : File Storage
Protocol          : NFS v4.1
Port              : 2049
Multi EC2 Access  : Yes
Multi-AZ          : Yes
Auto Scaling      : Yes
Mount Target      : Required
Encryption        : KMS/TLS
Persistent Mount  : /etc/fstab
Kubernetes        : EFS CSI Driver
```

## Production Best Practices

- ✅ Enable encryption at rest and in transit
- ✅ Create Mount Targets in every AZ
- ✅ Use Elastic Throughput
- ✅ Enable AWS Backup
- ✅ Use Lifecycle Policies (Standard → IA → Archive)
- ✅ Use Security Group references instead of CIDR ranges
- ✅ Monitor using Amazon CloudWatch
- ✅ Use IAM + EFS Access Points for application-level access control
- ✅ Mount EFS using TLS (`-o tls`) in production environments.
