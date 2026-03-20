# 🌍 Migration Project

🔗 **GitHub Repository:**
[https://github.com/adarsh0331/Project_26_Migration_Project.git](https://github.com/adarsh0331/Project_26_Migration_Project.git) 

## 📌 Overview

This project demonstrates a **multi-region migration strategy** for a sample Python web application using AWS services.

---

## 🚀 Migration Approach

The following approach was implemented:

1. Copied EC2 AMIs across regions
2. Used Terraform to deploy identical infrastructure in the target region
3. Enabled continuous database replication
4. Configured S3 Cross-Region Replication (CRR)
5. Used Route 53 weighted routing to shift traffic gradually

---

## ⚙️ Jenkins Setup (CI/CD)

### Step 1: Create EC2 Instance (Jenkins Server)

### Step 2: Install Dependencies

```bash
sudo yum update -y
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo dnf install java-17-amazon-corretto -y
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins

sudo mount -o remount,size=4G /tmp/
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Install Git & Docker

```bash
sudo yum install git -y

sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker jenkins
```

### Pipeline Flow

* Git clone
* Build Docker image
* Push to DockerHub
* Deploy container

Access app:

```
http://<public-ip>:5000
```

---

## 🏗️ Application Deployment (us-east-1)

### 1. Create EC2 Instance

* AMI: Amazon Linux 2023
* Instance Type: t2.micro
* Security Group:

  * SSH (22) → Your IP
  * HTTP (80) → Anywhere
  * TCP (5000) → Anywhere

### 2. Install Docker & Run App

```bash
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user

docker pull shilpa1819/python-flask-app:latest
docker run -d -p 5000:5000 shilpa1819/python-flask-app
```

---

### 3. Create AMI

* Create AMI from EC2 instance

---

### 4. Launch Template

* Select AMI
* Instance type
* Security group
* Key pair

---

### 5. Target Group

* Protocol: HTTP
* Port: 5000
* Register instances

---

### 6. Application Load Balancer

* Scheme: Internet-facing
* Listener: HTTP (80)
* Attach Target Group

---

### 7. Auto Scaling

* Desired: 2
* Min: 1
* Max: 3

---

### 8. Run App in New Instances

```bash
docker run -d -p 5000:5000 adarshbarkunta/python-flask-app
```

---

### 9. Verify

```
http://<public-ip>:5000
```

---

### 10. Copy AMI to Another Region

* Copy AMI → us-west-2

---

## 🌎 Deployment in us-west-2

1. Verify AMI copy
2. Repeat steps (Launch Template → ASG → ALB)
3. Register instances
4. Verify application

---

## 🌐 Domain Setup (Route 53)

### Steps

1. Create Hosted Zone
2. Update Nameservers in GoDaddy
3. Create Record:

* Type: A
* Alias → ALB DNS

---

## ⚖️ Weighted Routing (Traffic Shift)

### Region 1

* Weight: 100

### Region 2

* Weight: 0

Gradually shift traffic by adjusting weights.

---

## 🗄️ RDS Setup

### Create Database

* Engine: MySQL
* Instance: db.t3.micro
* Storage: 20GB

---

### Connect to RDS

```bash
sudo dnf install mysql -y
# If issue:
sudo dnf install mariadb105 -y

mysql -h <RDS-ENDPOINT> -u admin -p
```

---

### Create DB & Table

```sql
CREATE DATABASE devops_app;
USE devops_app;

CREATE TABLE visitors (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50),
  visit_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO visitors (name) VALUES ('shilpa');
SELECT * FROM visitors;
```

---

## 🔁 Cross-Region RDS Replication

### Steps

1. RDS → Databases
2. Select DB
3. Actions → Create Read Replica
4. Choose Region: us-west-2

---

### Verification

```sql
SELECT * FROM visitors;
```

---

## 🪣 S3 Cross-Region Replication (CRR)

### 1. Create Buckets

* Source: us-east-1
* Destination: us-west-2

---

### 2. Enable Versioning

* Enable on both buckets

---

### 3. Configure Replication

* Rule: replicate-to-west
* Destination: west bucket
* IAM role: auto-create

---

### 4. Test

Upload file → verify in replica bucket

---

## 🏛️ Architecture

```
Users
   ↓
Route53 (Weighted Routing)
   ↓
 ┌───────────────┐
 │               │
ALB East      ALB West
   ↓             ↓
 EC2           EC2
   ↓             ↓
Docker App   Docker App
   ↓             ↓
RDS Primary → RDS Replica
   ↓
S3 → Cross-Region Replication
```

---

## ✅ Final Outcome

* Multi-region deployment
* High availability
* Disaster recovery ready
* Gradual traffic shifting
* Automated replication
