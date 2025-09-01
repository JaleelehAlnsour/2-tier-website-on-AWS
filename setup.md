# 2-Tier Website Deployment on AWS

This guide provides a step-by-step process to deploy a 2-Tier Website on AWS, including networking, EC2 web tier, RDS database tier, ALB, auto-scaling, and security configurations.

---

## 1. Prerequisites

- AWS account with proper permissions (EC2, RDS, VPC, ALB, Auto Scaling, CloudWatch)  
- Key pair for EC2 SSH access  
- Domain name (optional)  

---

## 2. VPC & Networking

1. **Create VPC**  
   - CIDR: `10.0.0.0/16`  

2. **Create Subnets**  
   - Public (Web): `10.0.1.0/24`  
   - Private (DB): `10.0.2.0/24`  

3. **Internet Gateway**  
   - Attach to VPC  

4. **Route Tables**  
   - Public subnet: `0.0.0.0/0` → Internet Gateway  
   - Private subnet: no internet (or NAT Gateway if needed)  

5. **Security Groups**  
   - Web SG: Inbound `HTTP(80)`, `SSH(22)` from your IP  
   - DB SG: Inbound `MySQL(3306)` from Web SG
     
---

## 3. EC2 – Web Tier

- **Instance Type:** `t3.micro`  
- **AMI:** Amazon Linux 2 / Ubuntu  
- **Key Pair:** Use existing key  
- **Security Group:** Web SG  
- **Subnet:** Public subnet  
- **Auto-assign Public IP:** Yes  

**User Data Script (Linux)**
---

## 4. RDS – Database Tier

- **Engine:** MySQL / PostgreSQL  
- **Instance Type:** `db.t3.micro`  
- **Storage:** 20 GB (GP2 SSD)  
- **VPC:** Same VPC  
- **Subnet Group:** Private subnet  
- **Security Group:** DB SG  
- **Database Name:** `mydb`  
- **Master Username:** `admin`  
- **Master Password:** `ChangeMe123!`  

> DB should be private; accessible only from Web Tier.

---

## 5. Application Configuration

- Connect Web server to RDS using the endpoint:  
  `mydb.xxxxxxx.us-east-1.rds.amazonaws.com`  
- Use database credentials defined above.  

---

## 6. Application Load Balancer (ALB)

1. **Create ALB**  
   - Internet-facing, in public subnets  

2. **Security Group**  
   - Allow inbound `HTTP(80)`  

3. **Target Group**  
   - Type: Instances  
   - Protocol: HTTP 80  
   - Register EC2 instances  

4. **Listener**  
   - HTTP 80 → Target Group  

---

## 7. Auto Scaling

- **Launch Template:** Use same AMI & Security Group as Web EC2  

- **Auto Scaling Group (ASG):**  
  - Subnets: Public  
  - Desired: 2  
  - Min: 1  
  - Max: 4  
  - Attach to ALB Target Group  

- **Scaling Policy:**  
  - Scale out if CPU > 70%  
  - Scale in if CPU < 30%  

---

## 8. Monitoring & Alarms

- **CloudWatch:** Monitor EC2 CPU, RDS CPU, ALB 5xx errors  
- **Optional IAM Role:** Grant EC2 access to S3 or other AWS services  

**Example Alarms:**  
- EC2 CPU > 80%  
- RDS CPU > 70%  
- ALB HTTP 5xx errors > threshold  

---

## 9. Verification

1. Open browser → `http://ALB-DNS`  
2. Verify page shows: *“Welcome to 2-Tier Website”*  
3. Optional: Test DB connection with PHP script




systemctl start httpd
echo "<h1>Welcome to 2-Tier Website</h1>" > /var/www/html/index.html
