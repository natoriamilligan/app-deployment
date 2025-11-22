# App Deployment Steps
1. Register a domain using a Domain Registrar
2. Create an S3 bucket with the same name as the root domain
3. Use Route 53 as the DNS service for the domain 
   1. Create a hosted zone (name hosted zone the root domain)
   2. Add Route 53 nameservers for your hosted zone to the Domain Registrar used to register domain
   3. Make sure nameservers have propogated. (Can take up to 48 hours)
4. Create a CloudFront distribution with the origin as S3 bucket (will add bucket policy onto root domain S3 bucket)
6. Request SSL Certificate from AWS Certificate Manager
   1. Include root domain and (www) subdomain in certificate
   2. Add CNAMES to hosted zone in Route 53 (ACM did this automatically and added A and AAAA records to hosted zones. or do manually)
   3. Once certificate is issued, add certificate to CF distribution
   4. Add root domain and subdomain as alternate domains for CF distribution
   5. add index.html as the default root object
   6. Wait for CF ditribution to redeploy






## Services Used
* Amazon Route 53 ($0.50/m)
* Amazon Cloud Front (First 1TB free/m --)


## What I Learned
- How to use Route 53 DNS service for an existing domain
- How to create alias records in Route 53 to access AWS resources
- How to redirect an S3 bucket to another S3 bucket
