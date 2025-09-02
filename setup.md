# 2-Tier Website Deployment on AWS
This guide provides a step-by-step process to deploy a 2-Tier Website on AWS, including networking, EC2 web tier, RDS database tier, ALB, auto-scaling, and Route 53.

---

## 1. Prerequisites
- AWS account with proper permissions (EC2, RDS, VPC, ALB, Auto Scaling, Route 53)  
- Key pair for EC2 SSH access  
- Domain name (Route 53)  

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
- **AMI:** Amazon Linux 2 
- **Key Pair:** Use existing key  
- **Security Group:** Web SG  
- **Subnet:** Public subnet  
- **Auto-assign Public IP:** Yes  

**User Data Script (Linux)**

  #!/bin/bash  
  yum update -y  
  yum install -y httpd php  
  systemctl enable httpd  
  systemctl start httpd  
  echo "<h1>Welcome to 2-Tier Website</h1>" > /var/www/html/index.html  

---

## 4. RDS – Database Tier
- **Engine:** MySQL 
- **Instance Type:** `db.t3.micro`  
- **Storage:** 20 GB (GP2 SSD)  
- **VPC:** Same VPC  
- **Subnet Group:** Private subnet  
- **Security Group:** DB SG  
- **Database Name:** `mydb`  
- **Master Username:** `admin`  
- **Master Password:** `Password123!`  

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

## 8. Route 53
1.  Create Hosted Zone for your domain.
2.  Add Record
     Type: A or CNAME
     Target: ALB DNS name
3. Wait for DNS propagation (~minutes)
4. Access your website using the domain name
  
---

## 9. Verification
1. Open browser → go to your domain (Route 53)
2. Verify page shows: “Welcome to 2-Tier Website”

