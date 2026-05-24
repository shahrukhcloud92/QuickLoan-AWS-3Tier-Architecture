# QuickLoan: Scalable & High-Availability Three-Tier Web Application on AWS

## Project Overview

QuickLoan is a production-grade loan application platform engineered to demonstrate a robust three-tier web architecture on Amazon Web Services (AWS). In single-server architecture deployments, platforms face systemic vulnerabilities including Single Points of Failure (SPOF) and an inability to adapt to unpredictable traffic spikes.

This architecture solves those constraints by completely decoupling the application into distinct logical tiers:
* 
**Presentation Layer (Global Routing):** Manages end-user requests, offloads connection handshakes, and abstracts internal infrastructure scaling via a Public Application Load Balancer and Dynamic DNS.
* 
**Application Layer (Elastic Compute):** A stateless fleet of web nodes running Nginx and PHP-FPM inside isolated public routing subnets that expand and contract horizontally according to user demand.
* 
**Data & Asset Storage Layer (Persistence):** Completely isolates dynamic transaction data inside a protected, private relational database subnet group and shifts static media storage globally to high-durability object storage.
---

## Project Metadata

* **Prepared By:** Shahrukh Kausar Shaikh
* **Guided By:** Lekharaj Sir
* **Cloud Platform:** Amazon Web Services (AWS)
* **Region Used:** us-west-2 (Oregon)
 
**Core Stack:** Nginx, PHP 8.2 (PHP-FPM), MySQL 8.0, Git, HTML5, CSS3, JavaScript.
---

## Table of Contents

1. System Architecture & Traffic Flow
2. Technical Stack Blueprint
3. Phase 1: Isolated Network Infrastructure Deployment
4. Phase 2: Security Groups & Stateful Firewall Rules
5. Phase 3: Compute Tier & Core Instance Provisioning
6. Phase 4: Web Stack Configuration & Application Deployment
7. Phase 5: Object Storage Static Asset Offloading
8. Phase 6: Managed Relational Database Engine Setup
9. Phase 7: Dynamic DNS Domain Configuration
10. Phase 8: High Availability & Load Balancing Configuration
11. Phase 9: Auto Scaling Infrastructure & CloudWatch Automation
12. Phase 10: Infrastructure Stress Testing & Event Verification
13. Enterprise Resource Cleanup Protocol

---

## 1. System Architecture & Traffic Flow
The transactional workflow loops through the environment via distinct pathways:

1. **Public Entrance:** End users target the custom host pointer quickloann.hopto.org. Dynamic DNS checks route queries directly to the active Application Load Balancer (ALB) footprint.
2. 
**Load Apportionment:** The ALB evaluates targets across availability zones us-west-2a and us-west-2b and targets the stateless application web server fleet on Port 80.
3. 
**Media Stream Optimization:** Frontend scripts force client browsers to pull structural graphics (logos and UI images) directly from an Amazon S3 bucket policy over secure HTTPS connections, conserving critical application processing cycles.
4. 
**Protected Query Processing:** Form submissions are parsed by PHP-FPM processes on active nodes, which establish an authenticated internal database connector socket across internal network lanes on Port 3306 to target the isolated Amazon RDS MySQL database.
5. 
**Administrative Bridge:** System administrators access infrastructure components strictly via a single hardened Jump Server (Bastion Host) using secure terminal keys, pivoting internally across private subnet IPs.
---

## 2. Technical Stack Blueprint
* 
**Networking (AWS VPC):** Core software isolation container separating the virtual datacenter from other cloud tenants.
* 
**Subnets:** Zones public request access points completely away from secure backend relational systems.
* 
**Public Gateway (Internet Gateway):** Implements bidirectional external edge routing maps for edge nodes.
* 
**Private Gateway (NAT Gateway + Elastic IP):** Provides a one-way secure internet escape route for private nodes to download update patches.
* 
**Web Host (Nginx Open Source):** Serves as a highly efficient reverse-proxy and client connection engine.
* 
**Processing Stack (PHP 8.2 & PHP-FPM):** Interprets application business logic and converts payloads into relational queries.
* 
**Compute Engine (Amazon EC2):** On-demand virtual machines providing scalable CPU/RAM for system runtimes.
* 
**Image Standard (Custom AMI):** A baked system image containing preset system stacks used for horizontal machine cloning.
* 
**Data Core (Amazon RDS MySQL 8.0):** A managed relational data warehouse that handles automated maintenance and backups.
* 
**Object Storage (Amazon S3 Standard):** Highly durable file repository utilized to serve static site assets globally via HTTPS.
* 
**Load Balancer (Application Load Balancer):** Distributes web layers uniformly across independent physical data centers.
* 
**Elastic Scaling (Auto Scaling Group):** Dynamically manages computing capacity based on real-time traffic profiles.
* 
**Monitoring (Amazon CloudWatch Alarms):** Continuously monitors instance metrics to trigger horizontal scaling boundaries.
* 
**Alert Notification (Amazon SNS):** Pushes real-time infrastructure event states and scaling alerts straight to admin inboxes.
* 
**DNS Resolution (No-IP Host Configuration):** Binds a production-ready web domain name directly to elongated cloud entry strings.

---

## 3. Phase-Wise Implementation Steps

### Phase 1: Isolated Network Infrastructure Deployment

* **Step 1.1: Virtual Private Cloud (VPC) Subsystem**
* Configuration parameters: Select 'VPC Only', IPv4 CIDR Block = 10.10.0.0/16. Once provisioned, open the Actions panel, select Edit VPC settings, and explicitly enable both DNS resolution and DNS hostnames.
<img width="1366" height="768" alt="vpc" src="https://github.com/user-attachments/assets/b70d86a7-9334-45d7-b311-ee218d6919fb" />

* Technical Context: This establishes your isolated software data center. Enabling DNS hostnames is required to ensure that frontend servers can resolve the dynamic database endpoints provisioned by Amazon RDS.

* **Step 1.2: Redundant Multi-Availability Zone Subnet Architecture**
* Configuration parameters: Allocate three subnets bounded inside your custom VPC:
<img width="1366" height="768" alt="subnets" src="https://github.com/user-attachments/assets/5560967a-3717-4078-b858-e00370e8b43e" />

* public-subnet1 inside Availability Zone us-west-2a with CIDR block 10.10.1.0/24.
* public-subnet2 inside Availability Zone us-west-2b with CIDR block 10.10.2.0/24.
* private-subnet3 inside Availability Zone us-west-2c with CIDR block 10.10.3.0/24 (configured as private-subnet3 and RDS-Pvt-subnet-2).
* Technical Context: Distributing subnets structurally ensures your application can withstand a complete localized data center power outage without interrupting customer banking journeys.

* **Step 1.3: Gateway and Route Table Execution Maps**
* Configuration parameters: Instantiate an Internet Gateway (IGW-quickloan) and mount it to your VPC. Create a NAT Gateway (NAT-quickloan) placed explicitly inside public-subnet1, binding it to an Allocated Elastic IP.
<img width="1366" height="768" alt="internet gateway" src="https://github.com/user-attachments/assets/d64448c5-69e3-4f44-ae5b-adf1331385d5" />
<img width="1366" height="768" alt="nat gateway" src="https://github.com/user-attachments/assets/984c4dee-a695-45d6-9801-c903ef861402" />


* Technical Context: Create two Route Tables: public-RT mapped with destination 0.0.0.0/0 targeting your IGW (associated explicitly to public subnets 1 and 2) ; and private-RT mapped with destination 0.0.0.0/0 targeting your NAT Gateway (associated to private subnet 3).

### Phase 2: Security Groups & Stateful Firewall Rules

* **Step 2.1: Designing Inter-Tier Security Group Handshakes**
* Configuration parameters: Generate an edge firewall named APP-JUMP-SG matching your VPC. Inject inbound authorization constraints: allow Port 80 from 0.0.0.0/0, allow Port 22 from your specific administrator workstation IP address, and allow Port 3306 internally from 10.10.0.0/16.
<img width="1366" height="768" alt="security group inbound app server" src="https://github.com/user-attachments/assets/1d66889c-e5e6-4b03-a3c2-51c6470da64e" />

* Technical Context: Create a second firewall group named DATABASE-SG. Under inbound settings, select Type as MySQL/Aurora (3306) and choose the Source as Custom, binding it directly to the security group ID of your APP-JUMP-SG. This process is known as Security Group Referencing, which ensures no outside internet requests can reach the data boundary.

### Phase 3: Compute Tier & Core Instance Provisioning

* **Step 3.1: Server Infrastructure Allocation**
* Configuration parameters: Deploy two distinct EC2 instances using the Amazon Linux 2023 AMI standard on a t3.micro architecture template, using your unique key pair configuration (LTKP.pem):
<img width="1366" height="768" alt="instances" src="https://github.com/user-attachments/assets/4a8dbc97-bb80-4a91-a315-a30be758f6bf" />
* APP-SERVER: Placed inside public-subnet1 with Auto-assign Public IP enabled, bound to APP-JUMP-SG.
* JUMP-SERVER: Placed inside public-subnet1 with Auto-assign Public IP enabled, bound to APP-JUMP-SG.

* Technical Context: The Jump Server serves as a secure gateway, ensuring administrators can securely bridge into internal networks without exposing backend instances to direct external risks.

### Phase 4: Web Stack Configuration & Application Deployment

* **Step 4.1: Compilation and Deployment of the App Tier**
* Technical Process: Connect to the staging web instance via SSH. Update the package registry and install the core web engine and application runtimes:
sudo yum update -y
sudo yum install -y nginx php8.2 php-fpm php-mysqlnd php-pdo php-mbstring
sudo systemctl start nginx php-fpm
sudo systemctl enable nginx php-fpm


* Application Code Deployment: Upload the application codebase files using a secure transfer tool. Move your code files out of home storage and relocate them structurally inside Nginx's active default location layout pathways:
sudo mv /home/ec2-user/includes /usr/share/nginx/html/
sudo mv /home/ec2-user/public /usr/share/nginx/html/
sudo chown -R nginx:nginx /usr/share/nginx/html/public
sudo chmod -R 755 /usr/share/nginx/html/public

### Phase 5: Object Storage Static Asset Offloading

* **Step 5.1: Offloading Presentation Assets onto Global Storage**
* Configuration parameters: Create an S3 Bucket named aws-project-bucket-5052026 inside your local deployment zone. Disable Block all public access settings entirely. Paste an enterprise global public read resource rule into your bucket policy panel to grant display streaming capabilities directly to user browsers:
{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "PublicReadGetObject",
"Effect": "Allow",
"Principal": "*",
"Action": "s3:GetObject",
"Resource": "arn:aws:s3:::aws-project-bucket-5052026/*"
}
]
}

<img width="1366" height="768" alt="S3 object list" src="https://github.com/user-attachments/assets/1709291d-5dd7-4028-b465-4e2c3ebdc64c" />

* Code-Level Asset Alignment: Upload your graphic assets folder directly into the bucket. Open /usr/share/nginx/html/public/index.html and modify your local image source parameters (src="images/...") to point explicitly to the global S3 Object URL endpoints.


### Phase 6: Managed Relational Database Engine Setup

* **Step 6.1: Provisioning the Relational Engine**
* Configuration parameters: Create a DB Subnet Group choosing vpc-quickloan and map it across your regional AZ subnets. Create an Amazon RDS database engine selecting MySQL (Engine 8.0) under a Free Tier template layer named customer-db. Set master user as admin and password as admin123. Disable Public Access, apply your DATABASE-SG group firewall, and capture the dynamic engine connection endpoint string.
<img width="1366" height="768" alt="rds database" src="https://github.com/user-attachments/assets/14636a4c-99d5-4ba3-a656-beb117af2579" />


* Database Code Configuration: Edit the internal environment mapping file on your web node (/usr/share/nginx/html/includes/db_connect.php) and replace placeholder entries with your exact password, username, and RDS endpoint host string.

* **Step 6.2: Connecting via the Jump Bridge & Schema Instantiation**
* Technical Process: Connect to your public JUMP-SERVER via SSH. Install a standard client utility: sudo yum install mariadb105 -y. Run an authorization challenge bridge command to log into the private database space:
mysql -h customer-db.xxxx.us-west-2.rds.amazonaws.com -u admin -p

* Once authorized into the live query engine shell, run database initialization commands to prepare your dynamic persistence layers:
CREATE DATABASE quickloan_db;
USE quickloan_db;
CREATE TABLE applications (
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(255) NOT NULL,
contact VARCHAR(20) NOT NULL,
email VARCHAR(255) NOT NULL,
loan_type VARCHAR(50) NOT NULL
);

### Phase 7: Dynamic DNS Domain Configuration

* **Step 7.1: Binding Infrastructure to Human-Readable Domains**
* Configuration parameters: Register a free account at No-IP.com. Create a dynamic hostname target named quickloann.hopto.org mapped as an initial A Record pointing directly to your staging web instance's public IP address.

<img width="1366" height="768" alt="no ip cname dns" src="https://github.com/user-attachments/assets/2746cf02-6bcc-44b3-b803-4db1a3042a95" />

* Virtual Host Configuration: Edit your application configuration file (/usr/share/nginx/html/nginx/quickloan.conf) and update your server directive line to match your custom domain: server_name quickloann.hopto.org;. Move the completed configuration file into the active system virtual host routing pathway and clear the engine memory profiles:
sudo mv /usr/share/nginx/html/nginx/quickloan.conf /etc/nginx/conf.d/
sudo nginx -t
sudo systemctl restart nginx

### Phase 8: High Availability & Load Balancing Configuration

* **Step 8.1: Creating the Golden Image (AMI) Snapshot**
* Configuration parameters: Select your fully configured APP-SERVER, click the Actions menu, and navigate to Image and templates > Create image. Name the resource app-server-image. Once the AMI status changes to Available, safe-stop your original staging application server instance.
<img width="1366" height="768" alt="AMI available detail" src="https://github.com/user-attachments/assets/03c08a84-2fa0-4d20-9ceb-7d0f6f0e8a6f" />

* **Step 8.2: Base Launch Template Construction**
* Configuration parameters: Navigate to Launch Templates and create a template named LT1 (or LT-15march). Pick your custom app-server-image AMI standard, configure instance type as t3.micro, bind to key pair LTKP, and apply the APP-JUMP-SG security group. Under advanced options, ensure Auto-assign public IP is set to Enable.

<img width="1366" height="768" alt="launch template detail" src="https://github.com/user-attachments/assets/782e383e-045b-4feb-825c-067042995283" />

* **Step 8.3: Target Group and Application Load Balancer Execution**
* Configuration parameters: Create an Instance-type Target Group named TG-quickloan over HTTP Port 80, pointing health evaluations directly to path /public/index.html.
<img width="1366" height="768" alt="targey group health status" src="https://github.com/user-attachments/assets/5866b182-ebc1-41f4-ad8f-c6ec8208f514" />


* Create an internet-facing Application Load Balancer named LB-quickloan mapped across your public availability subnets. Select the APP-JUMP-SG group firewall, and configure listener routing targets to forward Port 80 requests directly to TG-quickloan. Copy the generated long ALB DNS name string.

<img width="1366" height="768" alt="load balancer dns" src="https://github.com/user-attachments/assets/0ac67055-ed2b-465c-b3e4-db25b4e25c96" />

* **Step 8.4: Converting Domain Mappings to Load Structures**
* Configuration parameters: Return to your open No-IP domain configuration panel. Modify the hostname record type for quickloann.hopto.org, switching its parameter tracking value from an A Record to a CNAME Record. Paste your Application Load Balancer DNS name string into the target text box field and click update.

<img width="1366" height="768" alt="no ip cname dns" src="https://github.com/user-attachments/assets/b5fbec46-9882-4dc4-9912-8ea46b4a5877" />

### Phase 9: Auto Scaling Infrastructure & CloudWatch Automation

* **Step 9.1: Deploying the Resilient Auto Scaling Group**
* Configuration parameters: Create an Auto Scaling Group named ASG-quickloan bound to your LT1 template blueprint. Choose your custom VPC and bind it across both your public subnet networks. Attach to your existing load balancer target group (TG-quickloan) and check the box to enable ELB health checks. Define capacity metrics as Desired = 2, Minimum = 1, Maximum = 5.
<img width="1366" height="768" alt="cloudwatch alarms" src="https://github.com/user-attachments/assets/6b28cbef-ebfa-439b-b3ce-ab29f301189e" />

* **Step 9.2: SNS Alerts & CloudWatch Automated Scaling Rules**
* Configuration parameters: Create a Standard Amazon SNS Topic named quickloan-alert (or AWS-PROJECT-NOTIFICATION) and add an Email subscription pointing directly to your primary admin inbox monitor.
<img width="1366" height="768" alt="sns topic email" src="https://github.com/user-attachments/assets/87168aef-3921-4caa-a47a-0be4d215781f" />

<img width="1366" height="768" alt="ASG policies" src="https://github.com/user-attachments/assets/6c7fdeca-1ffc-46a7-8e94-5c6d34a06304" />

* Open CloudWatch Alarms and configure two custom rules : Quickloan-Highcpu-Alarm configured to trigger when average CPU utilization across the group goes Greater than 70% for 1 minute ; and Quickloan-Lowcpu-Alarm configured to trigger when average CPU drops Lower than 20% for 1 minute. Link both alarms to send alerts directly to your SNS topic.
<img width="1366" height="768" alt="cloudwatch alarms" src="https://github.com/user-attachments/assets/615f6ef6-641d-4f2a-b009-37a8a9c5228e" />


* Policy Integration: In the ASG console panel, go to the automatic scaling rules tab and apply two simple scaling actions : create a policy to add 1 capacity unit when Quickloan-Highcpu-Alarm goes active; and create a removal policy to subtract 1 capacity unit when Quickloan-Lowcpu-Alarm goes active.

<img width="1366" height="768" alt="ASG policies" src="https://github.com/user-attachments/assets/81fbb6c8-9c9e-46e0-92f9-7c6ca334900f" />

### Phase 10: Infrastructure Stress Testing & Event Verification

* **Step 10.1: Simulating Heavy Production Loads**
* Technical Process: Use your Jump server to log into one of your newly active Auto Scaling application nodes using its internal Private IP address. Install the system performance load utility tool: sudo yum install -y stress-ng. Run a multi-core workload emulation process to simulate high traffic load:
stress-ng --cpu 4 --timeout 600s --metrics-brief

* **Step 10.2: Validating System Response & Failover Execution**
* System Event Monitoring: Within a minute of running the stress test, check your inbox for an automated notification confirming that the high CPU limit has been breached. Check your active EC2 instances console panel to watch the Auto Scaling Group automatically spin up a third application instance clone to alleviate system load.
<img width="1366" height="768" alt="Screenshot 2026-05-05 214650" src="https://github.com/user-attachments/assets/c7389716-1bd7-444d-b613-da591609ae68" />

<img width="1366" height="768" alt="stress test email high" src="https://github.com/user-attachments/assets/13573387-a3d8-4dbd-908a-98a778fcefb0" />
<img width="1366" height="768" alt="stress test instance launch automatic" src="https://github.com/user-attachments/assets/c76a32f5-d819-40f4-9ab2-a8de4594e632" />

* **Step 10.3: Functional End-to-End Application Verification**
* Technical Process: Open a web browser window and open your registered domain path: quickloann.hopto.org/public/index.html. Fill out and submit the loan application form fields with test parameters.

<img width="1366" height="768" alt="loan apply form" src="https://github.com/user-attachments/assets/f634873e-3f0a-45d5-b25d-7d8b79e40140" />

* Backend Validation: Re-authenticate into your private RDS MySQL instance from your terminal bridge shell panel and execute a query verification command: SELECT * FROM applications;. The table must successfully display the values you just entered into the web form, proving end-to-end data flow integrity.

<img width="1366" height="768" alt="database entry verification" src="https://github.com/user-attachments/assets/f2a9cd07-5e65-42d6-a6d3-46cc932b2a77" />

---

## 🛑 4. Enterprise Resource Cleanup Protocol

To maintain maximum cost control and ensure no lingering or unassociated components incur charges, follow this exact, structural reverse-deletion order when tearing down the platform:

1. **Auto Scaling Group Fleet Removal:** Select ASG-quickloan and click Delete. This terminates all active instances managed by the scaling infrastructure and stops new machines from spinning up during teardown.
2. **Launch Templates:** Remove your custom configurations by deleting template LT1.
3. **Application Load Balancer & Target Groups:** Delete the active ALB (LB-quickloan) followed by the corresponding target group tracking layer (TG-quickloan).
4. **Amazon RDS MySQL Instance:** Delete your database engine customer-db. Uncheck the box to create a final snapshot, confirm the acknowledgment boxes, and enter 'delete me' into the confirmation panel field.
5. **Standalone EC2 Staging Machines:** Return to EC2 Dashboard > Instances, select both your staging APP-SERVER and administrative JUMP-SERVER, click Instance state > Terminate instance, and confirm.
6. **NAT Gateways & Public Elastic IP Allocations:** Locate and delete your active NAT-quickloan instance. Once its state updates to Deleted, navigate to your Elastic IP Dashboard, select the IP associated with the NAT Gateway, click Actions, and select Release Elastic IP address. Keeping an unassociated Elastic IP address active in an idle cloud footprint incurs continuous hourly charges.
7. **Amazon S3 Object Storage Bucket:** Select your bucket name. Choose Empty to completely purge all file structures from storage, then select Delete bucket to clear out the parent directory entirely.
8. **Custom AMIs & Block Storage Snapshots:** Navigate to AMIs, select your custom app-server-image, click Actions, and choose Deregister. Move to the Snapshots Dashboard, locate the underlying snapshot block associated with that specific image, and delete it.
9. **Virtual Private Cloud (VPC) Sandbox Container:** Finally, go to Your VPCs, select your custom vpc-quickloan network instance, and click Delete VPC. This clean master cleanup process automatically clears away your custom subnets, routing tables, network ACLs, and attached gateways for you.

```

```
