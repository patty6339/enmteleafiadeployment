### AWS Deployment Architecture for TeleAfia (Containerized with Free-Tier Focus)

#### 1. Frontend Deployment
**Services:**  
- **Amazon ECS (Elastic Container Service) for React App**  
- **S3 for Static File Storage**  
- **Route 53 for DNS Management**  
- **Certificate Manager for SSL/TLS**  

**Benefits:**  
- Scalable, containerized frontend architecture  
- Free-tier eligible for S3 storage and Route 53  
- HTTPS security with Certificate Manager  

**AWS Configuration Steps:**

1. **Create S3 Bucket for Frontend Static Files**  
   ```bash
   aws s3 mb s3://teleafia-frontend
   aws s3 website s3://teleafia-frontend --index-document index.html --error-document index.html
   ```

2. **Upload React Build Files to S3**  
   ```bash
   cd Frontend_react
   npm run build
   aws s3 sync build/ s3://teleafia-frontend
   ```

3. **Route 53 Configuration for DNS**  
   ```bash
   aws route53 create-hosted-zone --name teleafia.com --caller-reference $(date +%s)
   aws route53 change-resource-record-sets --hosted-zone-id Z3P5QSUBK4POT --change-batch '{"Changes": [{"Action": "CREATE", "ResourceRecordSet": {"Name": "teleafia.com", "Type": "A", "TTL": 60, "ResourceRecords": [{"Value": "your-s3-bucket-region.amazonaws.com"}]}}]}'
   ```

---

#### 2. Backend Deployment
**Services:**  
- **Amazon ECS for Node.js App**  
- **EC2 Instances with Auto Scaling Group for Backend**  
- **Application Load Balancer (ALB)**  
- **RDS for MySQL Database**  

**Benefits:**  
- Containerized backend with ECS  
- Free-tier eligible EC2 instances and RDS  
- ALB for load balancing  

**AWS Configuration Steps:**

1. **Create ECS Cluster and Task Definition for Backend Node.js App**  
   ```json
   {
     "family": "teleafia-backend-task",
     "containerDefinitions": [
       {
         "name": "backend-container",
         "image": "your-repository/backend-app:latest",
         "memory": 1024,
         "cpu": 512,
         "essential": true,
         "portMappings": [
           {
             "containerPort": 5500,
             "hostPort": 5500
           }
         ],
         "environment": [
           {
             "name": "DB_HOST",
             "value": "your-rds-endpoint.region.rds.amazonaws.com"
           },
           {
             "name": "DB_USER",
             "value": "admin"
           },
           {
             "name": "DB_NAME",
             "value": "teleafia_db"
           }
         ]
       }
     ]
   }
   ```

2. **Deploy Backend Node.js App via ECS**  
   ```bash
   aws ecs create-service --cluster teleafia-backend-cluster --service-name teleafia-backend-service --task-definition teleafia-backend-task --desired-count 2 --launch-type EC2 --network-configuration "awsvpcConfiguration={subnets=[subnet-id],securityGroups=[sg-id],assignPublicIp='ENABLED'}"
   ```

3. **Create Application Load Balancer (ALB)**  
   ```bash
   aws elbv2 create-load-balancer --name teleafia-alb --subnets subnet-abc123,subnet-def456 --security-groups sg-xyz789 --type application --scheme internet-facing
   ```

4. **Target Group for ECS Services**  
   ```bash
   aws elbv2 create-target-group --name teleafia-target-group --protocol HTTP --port 5500 --vpc-id vpc-abc123
   ```

---

#### 3. Database Deployment
**Services:**  
- **Amazon RDS for MySQL**  
- **Multi-AZ Deployment with Free-tier Benefits**  
- **Automated Backups and Point-in-Time Recovery**  

**Database Configuration Steps:**

1. **RDS MySQL Setup**  
   ```sql
   CREATE DATABASE teleafia_db;
   USE teleafia_db;
   SOURCE setup_db.sql;
   ```

2. **Environment Variables for ECS**  
   ```plaintext
   DB_HOST=your-rds-endpoint.region.rds.amazonaws.com
   DB_USER=admin
   DB_NAME=teleafia_db
   DB_PORT=3306
   ```

---

#### 4. File Storage
**Services:**  
- **S3 for User Uploads**  
- **EC2 for Backend Storage (Optional)**  

---

#### 5. Security
**Services:**  
- **WAF (Web Application Firewall)**  
- **Shield for DDoS Protection (optional)**  
- **IAM for Access Management**  
- **Secrets Manager for Credentials**  

---

#### 6. Monitoring Setup (CloudWatch Configuration)

```json
{
  "metrics": {
    "namespace": "TeleAfia",
    "metrics_collected": {
      "ecs": {
        "cpu": {
          "measurement": ["cpu_usage_idle", "cpu_usage_user", "cpu_usage_system"]
        },
        "memory": {
          "measurement": ["mem_used_percent"]
        },
        "disk": {
          "measurement": ["disk_used_percent"]
        }
      },
      "alb": {
        "httpCode_5XX": {
          "measurement": ["HTTP_5XX_count"]
        }
      }
    }
  }
}
```

---

#### 7. Scaling and Cost Optimization
- **Horizontal Scaling**: ECS services, ALB with auto-scaling, and RDS Multi-AZ setup.  
- **Cost Optimization**:  
  - Development environment: $50-100 per month (t3.micro EC2, RDS t3.micro, minimal S3 usage).  
  - Production environment: $200-500 per month (t3.small/medium EC2, RDS Multi-AZ, increased S3 usage).

---

