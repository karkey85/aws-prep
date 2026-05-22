
[ Internet ] ---> [ EC2 Instance ] ---> [ Port 3306 ] ---> [ RDS Instance ]
                  (Has SG-Web)                             (Has SG-DB)
                                                           (Inbound rule: Allow SG-Web)

The Core Concept
Instead of whitelisting specific IP addresses, you will create a rule that says: "Allow traffic into the database from any resource that carries
the Web Server's Security Group."

**Step 1: Create the Two Security Groups**
1. Open the Amazon EC2 Console and select Security Groups from the left sidebar.
2. Click Create security group to build the EC2 Firewall:
  Security group name: SG-Web-Server
  Description: Allow SSH from my IP.
  VPC: Select your Default VPC.
  Inbound rules: Add a rule for SSH (Port 22) and set the Source to My IP.
  Click Create security group.
3. Click Create security group again to build the RDS Firewall:
  Security group name: SG-Database
  Description: Allow database traffic from the web server group.
  VPC: Select the same Default VPC.
  Inbound rules: Leave this completely blank for now. (We will link it in Step 3).
  Click Create security group.

**Step 2: Launch the Resources with Their Respective Groups**

Now, tie these firewalls to your new compute and database components.

**Launch the EC2 Instance**
Go to the EC2 Console -> Launch Instance -> Name it Manual-Web-Server and pick Amazon Linux 2023 -> Under Network settings, click Edit->Instead of creating a new group, choose Select existing security group and check the box next to SG-Web-Server.
->Launch the instance.

**Launch the RDS Database**
Go to the RDS Console -> Create database (Standard Create -> MySQL -> Free Tier)->
Scroll down to the Connectivity section.->
For Compute resource, explicitly choose Don't connect to an EC2 compute resource. (This forces the manual configuration path). ->
Under Existing VPC security groups, remove the default group and select SG-Database.->
Create the database.

**Step 3: Link the Security Groups (The Magic Step)**
Copy the Web Group ID in SG-Webserver and paste in the source of SG-Database in a new rule with Aurora/Mysql and custom type.

**Step 4: Verify via Command Line**
Once your RDS database status changes to Available, copy its endpoint address like karthik-database-2.xxx.ap-south-1.rds.amazonaws.com 
SSH into your EC2 instance.

# 1. Update system libraries
sudo dnf update -y

# 2. Install the database client tools
sudo dnf install mariadb105 -y

# 3. Connect using the endpoint string
mysql -h your-rds-endpoint.amazonaws.com -u admin -p

If your configurations are correct, you will instantly be prompted for your password and drop straight into the SQL prompt.
