# Deploying a 2-Tier Website on AWS (Amazon Linux AMI)

## Objective
Build a scalable and reliable two-tier web application using AWS.

## Tools & Services Used
- **EC2 (Amazon Linux AMI)**
- **Apache/Nginx**
- **MySQL RDS**
- **VPC**
- **Security Groups**

## Architecture
- **Web Tier:** Handles HTTP requests with Apache/Nginx on EC2 instances.
- **DB Tier:** Managed MySQL database using Amazon RDS.
- **Load Balancer:** Distributes traffic across the web tier for scalability and availability.

```
Diagram:  
ELB (Load Balancer) → Web Tier (EC2 + Apache/Nginx) → DB Tier (RDS MySQL)
```

![Architecture Diagram](architecture.png) <!-- Replace with your diagram file name -->

## Outcome
- Created a modular architecture separating web (frontend) and database (backend) layers
- Ensured better scalability and maintainability

## Getting Started
1. Launch EC2 instances in a new/existing VPC.
2. Set up Apache/Nginx on EC2.
3. Provision an RDS MySQL instance.
4. Configure Security Groups for each layer.
5. Deploy your application code.
6. Test end-to-end connectivity.

