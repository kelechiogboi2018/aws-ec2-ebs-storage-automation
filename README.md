# AWS EBS Backup, Snapshot and Disaster Recovery Project

## Project Overview

This hands-on AWS project demonstrates practical management of **Amazon Elastic Block Store (EBS)** for an EC2 workload, including volume attachment, Linux filesystem configuration, persistent mounting, manual snapshots, automated snapshot management with **AWS Data Lifecycle Manager (DLM)**, and the disaster-recovery workflow for restoring an EBS volume from a snapshot.

The project was completed in the **Europe (London) – `eu-west-2`** AWS Region.

## Architecture

![AWS EBS Backup and Disaster Recovery Architecture](./aws-ebs-backup-disaster-recovery-architecture.png)

### Architecture Flow

`Administrator → EC2 → Attached EBS Volume → Snapshot → AWS DLM Automation → Recovery Volume → EC2`

The EC2 instance uses an attached EBS data volume. The Linux filesystem is mounted and configured in `/etc/fstab` so that it remains available after reboot. EBS snapshots provide point-in-time backups, while AWS Data Lifecycle Manager automates scheduled snapshot creation and retention. During recovery, a snapshot can be used to create a new EBS volume and attach it to an EC2 instance in a compatible Availability Zone.

## Project Objectives

- Provision and attach an EBS volume to an EC2 instance.
- Identify the attached Linux block device.
- Create a filesystem on the new volume.
- Create a mount point and mount the EBS filesystem.
- Configure `/etc/fstab` for persistent mounting.
- Verify that data survives an EC2 reboot.
- Create a manual EBS snapshot.
- Configure automated snapshots using AWS Data Lifecycle Manager.
- Understand snapshot retention and backup lifecycle management.
- Practise the EBS snapshot restoration workflow.
- Document the implementation as a portfolio-ready DevOps project.

## AWS Services and Technologies

| Technology | Purpose |
|---|---|
| Amazon EC2 | Linux compute instance used for the project |
| Amazon EBS | Persistent block storage attached to EC2 |
| EBS Snapshots | Point-in-time backups of the EBS volume |
| AWS Data Lifecycle Manager | Automated snapshot scheduling and lifecycle management |
| Amazon VPC | Network boundary for the EC2 workload |
| Security Groups | Instance-level traffic control |
| Linux | Filesystem, mounting and storage administration |
| SSH | Remote administration of the EC2 instance |

## Implementation

### 1. Attach the EBS Volume

A separate EBS volume was attached to the EC2 instance. The operating system was then checked to identify the new block device.

Example:

```bash
lsblk
```

This is important because the device name presented by Linux can differ from the device name selected in the AWS console.

### 2. Create the Filesystem

After confirming that the new volume did not already contain a filesystem, an `ext4` filesystem was created.

```bash
sudo mkfs -t ext4 /dev/xvdf
```

> Device names vary between EC2 instance types and configurations. Always verify the actual device with `lsblk` before formatting.

### 3. Create a Mount Point

```bash
sudo mkdir -p /data
```

The EBS filesystem was then mounted:

```bash
sudo mount /dev/xvdf /data
```

Verification commands included:

```bash
lsblk
df -h
```

### 4. Configure Persistent Storage

A normal Linux mount performed with `mount` does not automatically survive a reboot. Persistent mounting was therefore configured through:

```text
/etc/fstab
```

Example entry:

```text
/dev/xvdf  /data  ext4  defaults  0  2
```

In production, using the filesystem **UUID** in `/etc/fstab` is generally safer than depending on a device name.

Useful command:

```bash
sudo blkid
```

The configuration should be validated before rebooting:

```bash
sudo mount -a
```

### 5. Test EBS Persistence

Test data was stored on the mounted EBS filesystem and the EC2 instance was rebooted.

After reconnecting, the mount and test data were checked again. This demonstrated an important EBS property:

> EBS provides persistent block storage independent of the EC2 instance's running state.

### 6. Create an EBS Snapshot

A manual snapshot of the EBS volume was created through:

**EC2 → Elastic Block Store → Snapshots**

The snapshot provides a point-in-time backup from which another EBS volume can be created.

### 7. Automate Snapshots with AWS Data Lifecycle Manager

AWS Data Lifecycle Manager was configured with an **EBS snapshot policy** to automate the backup process.

The policy was configured for scheduled snapshot creation and enabled successfully.

This introduced an important production principle:

> Backups should be automated rather than relying entirely on manual administrator actions.

### 8. Disaster Recovery Workflow

The recovery process is:

1. Select an available EBS snapshot.
2. Choose **Create volume from snapshot**.
3. Create the replacement volume in the required Availability Zone.
4. Attach the restored volume to an EC2 instance.
5. Identify the restored block device.
6. Mount the filesystem.
7. Verify that the original data is present.

A key AWS constraint is that an **EBS volume can only be attached to an EC2 instance in the same Availability Zone**.

## Useful Linux Commands

```bash
# List block devices
lsblk

# Display mounted filesystem usage
df -h

# Inspect filesystem/device UUID information
sudo blkid

# Create an ext4 filesystem
sudo mkfs -t ext4 /dev/xvdf

# Create a mount point
sudo mkdir -p /data

# Mount the filesystem
sudo mount /dev/xvdf /data

# Test /etc/fstab entries
sudo mount -a

# Display persistent mount configuration
cat /etc/fstab
```

## Backup and Recovery Lifecycle

```text
EBS Data
   ↓
Snapshot
   ↓
DLM Scheduled Backup
   ↓
Retention Policy
   ↓
Recovery Requirement
   ↓
Create New EBS Volume from Snapshot
   ↓
Attach and Mount
   ↓
Recover Application/Data
```

## Key Learning Outcomes

Through this project I developed practical experience with:

- EC2 storage administration.
- EBS volume attachment and management.
- Linux block devices and filesystems.
- `ext4` filesystem creation.
- Linux mount points.
- Persistent mounts using `/etc/fstab`.
- Storage troubleshooting using `lsblk`, `df` and `blkid`.
- EBS snapshot creation.
- Automated backups using AWS Data Lifecycle Manager.
- Backup retention concepts.
- Snapshot-based disaster recovery.
- Availability Zone requirements for EBS.
- AWS storage resilience and operational best practices.

## DevOps Skills Demonstrated

This project demonstrates practical knowledge of:

**AWS | EC2 | EBS | EBS Snapshots | AWS DLM | Linux | SSH | Filesystems | Persistent Storage | Backup Automation | Disaster Recovery | Cloud Operations | Troubleshooting**

## Production Improvements

The project can be extended by:

- Using filesystem UUIDs in `/etc/fstab`.
- Managing infrastructure with Terraform.
- Encrypting EBS volumes and snapshots with AWS KMS.
- Implementing appropriate snapshot retention policies.
- Testing recovery procedures regularly.
- Adding monitoring and alerting with Amazon CloudWatch and Amazon SNS.
- Copying snapshots across Regions where the recovery strategy requires it.
- Applying least-privilege IAM permissions.
- Documenting Recovery Point Objective (RPO) and Recovery Time Objective (RTO).

## Conclusion

This project moved beyond simply attaching cloud storage to an EC2 instance. It covered the operational lifecycle of persistent storage: **provisioning, filesystem configuration, persistence, backup automation and disaster recovery**.

The implementation demonstrates hands-on AWS and Linux skills relevant to **Junior DevOps Engineer, Cloud Engineer and Platform Engineer** roles.
