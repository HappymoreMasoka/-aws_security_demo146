# Secure Multi-VPC AWS Architecture with IAM & S3 Access

This project demonstrates a **secure AWS architecture** using:

* Two **non-overlapping VPCs**
* **IAM Roles** for secure access
* **VPC Peering** for private communication
* **Security Groups** and **Network ACLs** for access control
* **S3 bucket** with encryption and HTTPS-only policy

All services used are within the **AWS Free Tier**.

---

## 📐 Architecture Overview

```
[VPC-A: Public Web Server]           <--- VPC Peering --->         [VPC-B: Private S3 Zone]

- EC2 Instance (Free Tier, Ubuntu)
- IAM Role: AmazonS3ReadOnlyAccess
- Security Group: SSH, HTTPS
- Route to VPC-B

                            Peering Connection

- S3 Bucket (Encrypted, Private)
- Bucket Policy: HTTPS only
- No internet gateway in VPC-B
```

---

## 🔧 Services Used

| Service         | Purpose                              |
| --------------- | ------------------------------------ |
| VPC             | Isolated networks                    |
| EC2             | Simulated public web server          |
| S3              | Simulated private database zone      |
| IAM Role        | Access S3 from EC2 securely          |
| VPC Peering     | Secure communication between VPCs    |
| Security Groups | Control inbound/outbound EC2 traffic |
| NACL            | Extra subnet-level traffic control   |

---

## 🪜 Step-by-Step Setup

### 1. VPC Creation

* VPC-A: `10.0.0.0/16`
* VPC-B: `10.1.0.0/16`
* Subnets: `/24` in each VPC

### 2. EC2 Instance in VPC-A

* Ubuntu Free Tier AMI
* SSH access enabled from your IP only
* Assign IAM Role (see below)

### 3. IAM Role for EC2

* Trusted entity: EC2
* Policy: `AmazonS3ReadOnlyAccess`
* Attach to your EC2 instance

### 4. S3 Bucket

* Name: `my-private-data-zone`
* Encryption: Enabled (SSE-S3)
* Block public access: Enabled
* Upload a file (e.g. `data.csv`)

### 5. Bucket Policy to Force HTTPS

`s3-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::my-private-data-zone/*",
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

### 6. VPC Peering

* Create peering connection between VPC-A and VPC-B
* Update route tables in both VPCs

### 7. Security Groups and NACLs

* EC2 SG: Allow SSH from your IP, HTTPS to S3
* NACL: Allow only necessary traffic in each direction

### 8. Test from EC2:

```bash
aws s3 ls s3://my-private-data-zone
```

✅ You should see the uploaded file if IAM and peering work correctly

---



