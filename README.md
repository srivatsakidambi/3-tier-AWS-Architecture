# 3-tier-AWS-Architecture
Please find below my first of a series of projects to design and configure a highly available 3 Tier architecture using AWS infrastructure. The 3 Tier are given below as shown 
- Tier 1 - User/Presentation Tier
- Tier 2 - Application Tier
- Tier 3 - Data Tier
  
See below the high level topology of 
![plot](./Tier3Topology.png)

Step 1: 
To begin with we will first setup the VPC which will host the entire 3 tier architecture.  In this step we will be creating a VPC and Subnets as well as routing and security groups. 
- Go to “Your VPCs” from the VPC service on the AWS management console and click on the orange “Create VPC” button. 
![plot](./CreateVPC.png)

- Only create a vpc here and give it a name. Provide your own name or use the same one as shown here.
![plot](./CreateVPC2.png)

Now we will create Subnets, go to Subnets on the left hand side of the VPC service and click on it. 
- Add your VPC ID (select from the drop down where you will find it already is existing and select it. )
![plot](./CreateSubnet1.png)
-	Assign it a name letting you know what it is your first public subnet
-	Put it in any availability zone and give it a CIDR of 192.168.1.0/24
![plot](./CreateSubnet2.png)
-	Add a second subnet and name it Private Subnet 1 or something to let you know it is your first private subnet
-	Put it in the same availability zone as the first subnet you made and give it a CIDR of 192.168.2.0/24
![plot](./CreateSubnet3.png)

-	Add a third subnet and assign a name letting you know it is the second private subnet you will be making
-	Put it in the same availability zone as your first public subnet and give it a CIDR of 192.168.3.0/24
![plot](./CreateSubnet4.png)

- Add a fourth and final subnet and give it a name letting you know it is the third private subnet
- Put it in a different availability zone from the rest of your subnets and give it a CIDR of 192.168.4.0/24
![plot](./CreateSubnet5.png)

- Set up for route tables 
- Allocate an Elastic IP address by going to Elastic IPs on the left hand side and click “Allocate Elastic IP address”
![plot](./Elastic_IP1.png)

- Everything should be good as default but make sure that you are in the same region you have been creating everything in and then press “Allocate”. You can also add a name tag if you wish but it isn’t necessary
![plot](./Elastic_IP2.png)

- We will now create an internet gateway and attach it to the VPC by going to Internet Gateways on the left hand side and clicking “Create Internet Gateway”
![plot](./Internet_Gateway1.png)

- We will name it something similar to what is below and then click “Create Internet Gateway”
![plot](./Internet_Gateway2.png)

- Once it is created attach it to your VPC by clicking “Attach to a VPC” on the top of the screen
![plot](./Internet_Gateway3.png)

- Click the drop down and select your vpc that you made

![plot](./Internet_Gateway4.png)

- Create a NAT Gateway by clicking on Nat Gateways on the left hand side and then clicking “Create NAT Gateway”

![plot](./Nat_Gateway1.png)

- Give it a name similar to the one below and assign it to a public subnet
- Click the drop down for Elastic IPs and click the one you created previously
- Click “Create NAT gateway"

![plot](./Nat_Gateway2.png)

- Create Route Tables by first heading to “Route Tables” on the left hand side
- Click “Create route table”
![plot](./Route_Table1.jpeg)

- Give it a name letting you know this is the public route table for your lab
- Assign your VPC to it and click “Create route table”
![plot](./Route_Table2.png)

- Make a second route table naming it something to let you know that this is the private route table for your lab and assign your VPC to it

![plot](./Route_Table3.png)

- Now associate your subnets with their respective route table
- Click on the public route table and click on “Subnet association” next to “Details”
- Click on “Edit subnet associations”
  
![plot](./Route_Table4.png)


-	Click on your public subnet and then click “Save associations”

![plot](./Route_Table5.png)

-	Now add a route to our public route table to get access to the internet gateway
-	Click on “Routes” next to “Details” and click “Edit routes”

-	Add a new route having a destination of anywhere and a target of your internet gateway and click “Save changes”

![plot](./Route_Table6.png)

- Do the same thing for your private route table by clicking on it and going to its subnet associations and editing them

![plot](./Route_Table7.png)

-	Click on all three of your private subnets and save the associations

![plot](./Route_Table8.png)

-	Go to edit the routes of the private table


![plot](./Route_Table9.png)
- Add a route to the private table that has a destination of anywhere and a target of your Nat gateway that you made earlier

![plot](./Route_Table10.png)

-	Now to create our security groups (One for our bastion host, web server, app server, and our database) we will head to Security Groups on the left and click “Create security group”

![plot](./Security_Group1.png)
-	Give it a name and description letting you know it is for a bastion host
- Assign your VPC to it
- Give it three inbound rules, one for SSH using your IP and one for HTTP using 0.0.0.0/0 as well as https using 0.0.0.0/0
  
![plot](./Security_Group2.png)

-	Create another security group
-	Give it a name and description letting you know it is for a Web server
-	Assign your VPC to it
-	Give it the same inbound rules as the Bastion Host security group

![plot](./Security_Group3.png)  

-	Create another security group
-	Give it a name and description letting you know it is for an app server
-	Assign your VPC to it
-	Give it an inbound rule for All ICMP -IPv4 with a source of your web server SG and another inbound rule for SSH with a source of your bastion host SG

![plot](./Security_Group4.png) 

-	Create one final security group
-	Please give it a name and description letting you know it is for a database server
-	Assign your VPC to it
-	Give it two inbound rules both for MYSQL/Aurora and give one of them a source of your app server SG and the other one a source of your bastion host SG

![plot](./Security_Group5.png) 

-	Go back to your bastion host inbound rules and add one more for MYSQL/Aurora and a source of your database SG
-	Go back to your web server inbound rules and add one more for All ICMP - IPv4 and a source of your app server SG
-	Go back to your app server inbound rules and add one more for MYSQL/Aurora and a source of your database SG and then an HTTP and HTTPS rule both with a source of 0.0.0.0/0 

Step 2: Create Servers
-	Create Bastion Host
-	Select Amazon Linux 2 AMI
![plot](./Bastion_Host_EC2_Instance1.jpeg)

- Select t2.micro

![plot](./Bastion_Host_EC2_Instance2.png)

- Put in your VPC and Public Subnet and enable auto-assign public IP

![plot](./Bastion_Host_EC2_Instance3.png)

- Storage leave default
- Add a name tag to let you know this is the Bastion Host

![plot](./RDS_Capture1.png)

- Select an existing group and select your Bastion Host SG

![plot](./RDS_Capture2.png)
-	Launch and choose an existing keypair. This can be downloaded from the lab page
-	To make the Web Server follow the same steps until you get to Step 3

![plot](./RDS_Capture3.png)
-	Follow along like previously and change your network, subnet, and enable auto assign public ip
- Then go to user data and type this into it to set up the web server
#!/bin/bash
sudo yum update -y
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
Sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd

