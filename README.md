# Deployment Plan


Here’s a comprehensive guide to deploying your **MERN stack** app on the cloud (AWS specifically), while integrating the instructions from your notes:

---

## **1. Architecture**
A simple and cost-effective architecture can be built using **AWS EC2, RDS, and S3**:

- **Frontend (React)**:  
  - Deploy the static React build files to **Amazon S3** (Object Storage).  
  - Serve them via **AWS CloudFront** for faster content delivery globally.

- **Backend (Node.js/Express.js)**:  
  - Host the backend on an **EC2 instance**. Use an **Application Load Balancer (ALB)** to route traffic to the backend.  
  - Optionally use **Auto Scaling** to handle traffic spikes efficiently.  

- **Database (MySQL)**:  
  - Use **Amazon RDS** for a managed MySQL database to improve scalability, backups, and security.  

### High-Level Setup:
1. **React** → S3 + CloudFront (Static Hosting)  
2. **Node.js/Express.js** → EC2 + ALB  
3. **MySQL** → Amazon RDS  
4. **Logs/Backups** → CloudWatch and S3  

---

## **2. API**
- **What are you using?**  
  - Your API uses **Node.js/Express.js** to handle incoming HTTP requests.  
  - **REST API model** for communication between frontend and backend.

- **How does the API work?**  
  - Requests from the React frontend are sent to the Node.js server via the ALB.  
  - Backend interacts with the **RDS MySQL database** to fetch/store data.

- **Integration**  
  - Does it integrate with other AWS services?  
    - Yes: CloudWatch (monitoring), S3 (storage), and IAM roles for security.

---

## **3. Deployment**
### Steps:
1. **Frontend Deployment** (React):  
   - Build your React app:  
     ```bash
     npm run build
     ```  
   - Upload the `build/` folder to an S3 bucket.  
   - Enable public access for the bucket and configure it as a static website.  
   - Use **CloudFront** to distribute the React app globally.

2. **Backend Deployment** (Node.js/Express.js):  
   - Launch an **EC2 instance**:  
     - Install Node.js and configure the environment.  
     - SSH into the instance and pull your backend code:  
       ```bash
       git clone <repo-url>
       cd backend-folder
       npm install
       ```  
   - Use **PM2** to keep the Node.js app running:  
       ```bash
       npm install -g pm2  
       pm2 start server.js  
       ```  
   - Configure the EC2 instance security group to allow traffic on port 80/443.  
   - Attach an **Application Load Balancer (ALB)** for better traffic distribution.  

3. **Database Setup** (MySQL):  
   - Use **Amazon RDS** to create a MySQL database instance.  
   - Connect the Node.js backend to RDS using its endpoint.  
   - Example connection string in `server.js`:  
     ```javascript
     const mysql = require('mysql');
     const db = mysql.createConnection({
       host: 'your-rds-endpoint',
       user: 'your-username',
       password: 'your-password',
       database: 'your-database'
     });
     ```

4. **Build and Deploy Process**:  
   - Use **CI/CD pipelines** (e.g., GitHub Actions, AWS CodePipeline) for automating deployment.  
   - Steps:  
     - Push to GitHub.  
     - Build → Deploy to S3 (frontend) or EC2 (backend).  

---

## **4. Security**
- **Implement Key Measures**:
   - Use **IAM roles** for access permissions (least privilege).  
   - Enable **Security Groups** to restrict EC2 traffic.  
   - **Rotate secrets**: Store database credentials securely using **AWS Secrets Manager**.  
   - Monitor security vulnerabilities using **AWS Inspector** and **CloudWatch Logs**.  
   - Enable automated backups for RDS.  

- **Security vulnerabilities backup process**:  
   - Set up automated **snapshots** for RDS.  
   - Use S3 versioning to back up React static files.  

---

## **5. Cost**
- Use **AWS Pricing Calculator** to estimate monthly costs.  
- Example breakdown:  
   - **EC2**: ~$20/month (t2.micro for backend)  
   - **RDS**: ~$15/month (db.t3.micro for MySQL)  
   - **S3 + CloudFront**: ~$5/month for static hosting  
   - **Load Balancer**: ~$16/month  

- Monitor costs with **AWS Cost Explorer** and set **budget alerts**.

---

## **6. Challenges**
1. **Scaling**:  
   - Use **Auto Scaling Groups** for EC2.  
   - Enable **Read Replicas** in RDS for high traffic.  

2. **Performance**:  
   - Use CloudFront for faster frontend delivery.  
   - Use caching (e.g., Redis) for backend optimization.  

---


==============================================================================

Here’s a **detailed step-by-step deployment guide** for your MERN stack app on AWS, covering the setup of React, Node.js/Express, MySQL, and automation with a CI/CD pipeline.

---

## **1. Frontend Deployment (React on AWS S3 + CloudFront)**

### **Steps**:
1. **Build the React App**:  
   Run this command to create production-ready static files:  
   ```bash
   npm run build
   ```
   The output will be in the `build/` folder.

2. **Create an S3 Bucket**:  
   - Go to the **AWS Management Console → S3**.  
   - Create a new bucket (e.g., `my-react-frontend`).  
   - Enable public access and configure it as a static website.  

3. **Upload Files to S3**:  
   - Upload the contents of the `build/` folder to the S3 bucket.  
   - Set the permissions to allow public read access.  

4. **Configure CloudFront (CDN)**:  
   - Go to **CloudFront** in AWS Console and create a new distribution.  
   - Set the **Origin** to your S3 bucket.  
   - Use the default settings for caching and distribution.  
   - Copy the **CloudFront URL** to access your React app globally.

---

## **2. Backend Deployment (Node.js/Express on EC2 + ALB)**

### **Steps**:
1. **Launch an EC2 Instance**:  
   - Go to **EC2 → Launch Instance**.  
   - Choose Amazon Linux 2 (or Ubuntu 20.04) as the AMI.  
   - Select **t2.micro** for free-tier eligibility.  
   - Configure the Security Group to allow:  
     - **HTTP/HTTPS (ports 80/443)** for web traffic.  
     - **SSH (port 22)** for secure login.

2. **Set Up Node.js on the EC2 Instance**:  
   - SSH into the EC2 instance:  
     ```bash
     ssh -i your-key.pem ec2-user@your-ec2-public-ip
     ```
   - Install Node.js, npm, and Git:  
     ```bash
     sudo yum update -y
     sudo yum install -y nodejs npm git
     ```

3. **Deploy Your Backend Code**:  
   - Clone your backend code repository:  
     ```bash
     git clone <your-repo-url>
     cd <backend-folder>
     npm install
     ```
   - Start the app using **PM2** (keeps the server alive):  
     ```bash
     sudo npm install -g pm2
     pm2 start server.js
     pm2 save
     ```

4. **Set Up Application Load Balancer (ALB)**:  
   - Go to **EC2 → Load Balancers → Create ALB**.  
   - Configure the ALB to forward traffic to your EC2 instance.  
   - Update the **Security Group** to allow inbound traffic to the ALB.

---

## **3. Database Setup (Amazon RDS MySQL)**

### **Steps**:
1. **Create an RDS MySQL Database**:  
   - Go to **RDS → Create Database**.  
   - Select **MySQL** as the engine.  
   - Choose **db.t3.micro** (free-tier eligible).  
   - Configure the database name, username, and password.  

2. **Allow EC2 Access to RDS**:  
   - In the **RDS Security Group**, allow inbound traffic from your EC2 instance.  

3. **Connect Backend to RDS**:  
   Update your backend’s database configuration:  
   ```javascript
   const mysql = require('mysql');
   const db = mysql.createConnection({
     host: 'your-rds-endpoint',
     user: 'your-username',
     password: 'your-password',
     database: 'your-database'
   });

   db.connect((err) => {
     if (err) throw err;
     console.log("Connected to RDS MySQL!");
   });
   ```

---

## **4. CI/CD Pipeline (Automating Deployment with GitHub Actions)**

### **Steps**:
#### **For Frontend**:
1. **Create a GitHub Action Workflow**:  
   In your frontend repo, add a `.github/workflows/deploy.yml` file:  
   ```yaml
   name: Deploy React to S3
   on:
     push:
       branches:
         - main

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v3

         - name: Install dependencies and build
           run: |
             npm install
             npm run build

         - name: Deploy to S3
           uses: jakejarvis/s3-sync-action@master
           with:
             args: --acl public-read --delete
           env:
             AWS_S3_BUCKET: "your-s3-bucket-name"
             AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
             AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
             AWS_REGION: "your-region"
   ```

2. **Configure Secrets in GitHub**:  
   - Add `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` under **Repo Settings → Secrets**.

#### **For Backend**:
1. **Add Workflow for EC2 Deployment**:  
   In your backend repo, create `.github/workflows/deploy.yml`:  
   ```yaml
   name: Deploy Node.js to EC2
   on:
     push:
       branches:
         - main

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v3

         - name: Deploy to EC2
           uses: appleboy/scp-action@v0.1.4
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             source: "."
             target: "/home/ec2-user/backend"

         - name: Restart PM2
           uses: appleboy/ssh-action@master
           with:
             host: ${{ secrets.EC2_HOST }}
             username: ${{ secrets.EC2_USER }}
             key: ${{ secrets.EC2_SSH_KEY }}
             script: |
               cd /home/ec2-user/backend
               npm install
               pm2 restart all
   ```

2. **Configure EC2 SSH Access**:  
   - Add `EC2_HOST`, `EC2_USER`, and `EC2_SSH_KEY` as GitHub Secrets.

---

## **5. Security Best Practices**
1. **Use IAM Roles**: Attach roles to EC2 to access S3 and RDS securely.  
2. **Encrypt Data**: Enable encryption at rest for S3 and RDS.  
3. **Enable Backups**: Configure RDS automated backups.  
4. **Rotate Secrets**: Use AWS Secrets Manager for database credentials.  
5. **Monitor Logs**: Use CloudWatch for log monitoring and alarms.

---

## **6. Cost Management**
- Use the **AWS Pricing Calculator** to estimate monthly costs.  
- Enable **AWS Budgets** to set cost alerts.  

### Sample Estimate:
| Service       | Cost Estimate (Monthly) |
|---------------|-------------------------|
| EC2 (t2.micro)| $10–$20                |
| RDS (MySQL)   | $15–$20                |
| S3 + CloudFront| ~$5                   |
| Load Balancer | ~$16                   |

---
