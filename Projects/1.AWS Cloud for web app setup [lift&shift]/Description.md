<h1>AWS Cloud for Web App Setup [lift & Shift]</h1> 

### **End-to-End Process for Web App Setup on AWS Cloud**

#### **Step 1: Login to AWS Account**
- Log in to your AWS Management Console using your credentials.  
- Ensure you have the necessary permissions to perform resource creation and management.

---

#### **Step 2: Create Key Pairs**
- Navigate to **EC2 Dashboard** → **Key Pairs**.
- Create a new key pair and download the `.pem` file for SSH access.
  - Example: `myapp-keypair.pem`
- Store the key securely; it will be required for accessing your EC2 instances.

---

#### **Step 3: Create Security Groups**
- Go to **EC2 Dashboard** → **Security Groups** and create a new security group.
- Configure inbound and outbound rules:
  - **Inbound Rules**:
    - Allow SSH (Port 22) from your IP.
    - Allow HTTP (Port 80) and HTTPS (Port 443) from the internet.
  - **Outbound Rules**:
    - Allow all traffic for outbound requests.
- Save the security group.

---

#### **Step 4: Launch EC2 Instances with User Data**
- In the **EC2 Dashboard**, launch a new instance.
- Choose an Amazon Machine Image (e.g., Amazon Linux 2).
- Select instance type (e.g., t2.micro).
- Add the created **key pair** and **security group**.
- Add a **User Data Script** (bash script) to automate initial setup, such as installing dependencies like Tomcat:
```bash
#!/bin/bash
yum update -y
yum install -y tomcat
systemctl enable tomcat
systemctl start tomcat
```
- Launch the instance.

---

#### **Step 5: Update IP-to-Name Mapping in Route 53**
- Go to **Route 53** → **Hosted Zones**.
- Create or use an existing hosted zone for your domain.
- Add a new **A record** (Alias record) to point the domain name to your EC2 instance's public IP.

---

#### **Step 6: Build Application from Source Code**
- Use your build tool (e.g., Maven) to package the application:
```bash
mvn clean package
```
- This creates the application artifact (e.g., `myapp.war`).

---

#### **Step 7: Upload Artifact to S3 Bucket**
- Create an S3 bucket in **S3 Dashboard**.
- Upload the application artifact (`myapp.war`) to the S3 bucket.
- Ensure proper permissions for downloading the artifact from the bucket.

---

#### **Step 8: Download Artifact to Tomcat EC2 Instance**
- SSH into the Tomcat EC2 instance using the key pair:
```bash
ssh -i myapp-keypair.pem ec2-user@<EC2_Public_IP>
```
- Download the artifact from the S3 bucket:
```bash
aws s3 cp s3://<bucket-name>/myapp.war /var/lib/tomcat/webapps/
```
- Restart Tomcat to deploy the application:
```bash
systemctl restart tomcat
```

---

#### **Step 9: Set Up ELB with HTTPS**
- Go to **EC2 Dashboard** → **Load Balancers**.
- Create an **Application Load Balancer (ALB)**.
  - Choose internet-facing and add your EC2 instances to the target group.
  - Use **HTTPS** as the listener protocol.
- Obtain an SSL certificate:
  - Go to **AWS Certificate Manager (ACM)**.
  - Request a public SSL/TLS certificate for your domain.
  - Validate the certificate via DNS or email.
- Attach the SSL certificate to the ALB.

---

#### **Step 10: Map ELB Endpoint to Website Name in GoDaddy DNS**
- Copy the **DNS name** of the ALB from the **EC2 Dashboard**.
- Log in to your GoDaddy account and navigate to your domain settings.
- Update the DNS settings to point the domain's **CNAME record** to the ALB DNS name.

---

#### **Step 11: Verify**
- Open your domain in a browser (e.g., `https://www.myappdomain.com`).
- Verify that:
  - The website loads successfully.
  - The application is functioning as expected.
  - HTTPS is active with the SSL certificate.

---

This completes the **lift-and-shift** process for deploying a web application on AWS Cloud.
