# Banksie App Deployment

## ❓ Overview
In this project, I successfully deployed a multi-tier cloud-based banking app in AWS using S3, ECS, and RDS. 

## 🏗️ Architecture Setup

### 🖥️ Frontend
   1. Registered a domain using a domain registrar (banksie.app)
   2. Created a private S3 bucket with the same name as the root domain and added frontend files
   3. Used Route 53 as the DNS service for the domain 
      - Created a hosted zone (named hosted zone the root domain: banksie.app)
      - Added Route 53 nameservers for the hosted zone to the domain registrar used to register domain
      - Made sure nameservers propagated (Can take up to 48 hours)
   4. Created a CloudFront distribution with the origin as the S3 bucket 
   5. Added root domain (banksie.app) and subdomain (www.banksie.app) as alternate domains for CF distribution
   6. Requested SSL Certificate from AWS Certificate Manager
      - Included root domain and subdomain in certificate
      - ACM added CNAME records and A/AAAA records to the hosted zone, but this can be done manually
      - Once certificate was issued, added certificate to CF distribution
      - Added index.html as the default root object in the CF distribution
      - Wait for CF distribution to redeploy
     
### 🛢️ Database
   1. Create a database in RDS
      - Choose full configuration
      - PostgreSQL
      - Free tier
      - Created username and password
      - Choose instance configuration (Burstable classes / db.t3.micro)
      - Storage type: General Purpose SSD, 20 GiB
      - Choose VPC (must be the same that ECS tasks use)
      - Created new VPC security group to be configured later (flask-rds-sg)
      - Port: 5432
   2. Added the database endpoint to Flask app
   3. Created environment variable in AWS Secrets Manager console for database URL
      - Added a secret for the database URL
      - Choose "other type of secret"
      - Added database URL in plain text
     
### ⚙️ Backend
   1. Created an IAM user to handle ECR and ECS 
      1. Attached policies:
         - AmazonEC2ContainerRegistryFullAccess - allows user to use AWS ECR
         - AmazonECS_FullAccess - allows user to use AWS ECS
      2. Generated access key for the user to be user later
   3. Created repository for ECS image in AWS ECR
   4. Created an ECS Cluster
   5. Created a Task Definition
      - Name the task definition family (banksie-task)
      - Launch type = Fargate
      - CPU = .5 vCPU
      - Memory 2 GB
      - Create default task execution role
      - Container
        - Name container (banksie-container)
        - Add image URI (add :latest at the end)
        - Define port mapping = TCP Port 80 HTTP (Gunicorn is listening on port 80 in flask app)
   6. Revised the task definition with JSON to include the secret made using the secret arn
   7. Created and attached an inline policy to the task execution role to read secrets from AWS Secrets Manager
   10. Created ALB in EC2 console
      - Added listener for HTTP and HTTPS
        - Requested new ACM certificate for api.banksie.app
        - Added A record to Route 53 for api.banksie.app (will need time to propagate)
      - Created a security group for ALB (flask-app-alb-sg)
        - Allow inbound HTTP and HTTPS traffic from anywhere
        - Allow all outbound traffic to ECS tasks
      - create target group for ecs tasks (For an IP but do not add any targets)
        - Add health check path as /health
        - in Flask app add route for /health to return code 200
      - Add ALB url to frontend code files
      - Add new files to S3 bucket
   11. Created security group to be used by ECS tasks (banksie-sg)
       - Allow HTTP traffic from ALB created on port 80
       - Allow outbound traffic to RDS database
       - Modified the ALB security group to allow outbound traffic to this security group
   13. Create a service
      - Add task definition previously created
      - Choose capacity provider strategy (FARGATE)
      - Desired tasks = 1
      - Networking
        - Choose VPC (same as database)
        - Choose at least 2 subnets
      - Create security group that allows HTTP/HTTPS traffic from ALB on port 80
      - Add ALB that was previously created
      - Service will have an error until we push an image
   14. In EC2 console, go to security groups
      - Choose the RDS security group made previously
      - Add inbound rule so the task security group can access the RDS
   15. Push Docker image to ECR via AWS CLI
      - Configure AWS CLI credentials
      - Log into ECR 
      - Build/tag docker image
      - Push image
   16. Force new deployment for your service with the new image and revised task to include the image
   17. Set up Github Actions for automatic deployments
       - Add secrets and variables to the Github repository
       - Create workflow (.yml file)
