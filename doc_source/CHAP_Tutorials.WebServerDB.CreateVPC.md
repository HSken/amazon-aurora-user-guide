# Tutorial: Create an Amazon VPC for use with a DB instance<a name="CHAP_Tutorials.WebServerDB.CreateVPC"></a>

A common scenario includes a DB instance in an Amazon VPC, that shares data with a web server that is running in the same VPC\. In this tutorial you create the VPC for this scenario\. 

The following diagram shows this scenario\. For information about other scenarios, see [Scenarios for accessing a DB instance in a VPC](USER_VPC.Scenarios.md)\. 

 

![\[Single VPC Scenario\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/con-VPC-sec-grp.png)

Because your DB instance only needs to be available to your web server, and not to the public Internet, you create a VPC with both public and private subnets\. The web server is hosted in the public subnet, so that it can reach the public internet\. The DB instance is hosted in a private subnet\. The web server is able to connect to the DB instance because it is hosted within the same VPC, but the DB instance is not available to the public Internet, providing greater security\.

This tutorial configures an additional public and private subnet in a separate Availability Zone\. These subnets aren't used by the tutorial\. An RDS DB subnet group requires a subnet in at least two Availability Zones\. The additional subnet makes it easy to switch to a Multi\-AZ DB instance deployment in the future\.

This tutorial describes configuring a VPC for Amazon Aurora DB clusters\. For more information about Amazon VPC, see [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/) and [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\. 

**Note**  
For a tutorial that shows you how to create a web server for this VPC scenario, see [Tutorial: Create a web server and an Amazon Aurora DB cluster](TUT_WebAppWithRDS.md)\.

## Create a VPC with private and public subnets<a name="CHAP_Tutorials.WebServerDB.CreateVPC.VPCAndSubnets"></a>

Use the following procedure to create a VPC with both public and private subnets\. 

**To create a VPC and subnets**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the top\-right corner of the AWS Management Console, choose the Region to create your VPC in\. This example uses the US West \(Oregon\) Region\.

1. In the upper\-left corner, choose **VPC dashboard**\. To begin creating a VPC, choose **Create VPC**\.

1. For **Resources to create** under **VPC settings**, choose **VPC and more**\.

1. For the **VPC settings**, set these values:
   + **Name tag auto\-generation** – **tutorial**
   + **IPv4 CIDR block** – **10\.0\.0\.0/16**
   + **IPv6 CIDR block** – **No IPv6 CIDR block**
   + **Tenancy** – **Default**
   + **Number of Availability Zones \(AZs\)** – **2**
   + **Customize AZs** – Keep the default values\.
   + **Number of public subnet** – **2**
   + **Number of private subnets** – **2**
   + **Customize subnets CIDR blocks** – Keep the default values\.
   + **NAT gateways \($\)** – **None**
   + **VPC endpoints** – **None**
   + **DNS options** – Keep the default values\.
**Note**  
Amazon RDS requires at least two subnets in two different Availability Zones to support Multi\-AZ DB instance deployments\. This tutorial creates a Single\-AZ deployment, but the requirement makes it easy to convert to a Multi\-AZ DB instance deployment in the future\.

1. Choose **Create VPC**\.

## Create a VPC security group for a public web server<a name="CHAP_Tutorials.WebServerDB.CreateVPC.SecurityGroupEC2"></a>

Next you create a security group for public access\. To connect to public instances in your VPC, you add inbound rules to your VPC security group that allow traffic to connect from the internet\. 

**To create a VPC security group**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. Choose **VPC Dashboard**, choose **Security Groups**, and then choose **Create security group**\. 

1. On the **Create security group** page, set these values: 
   + **Security group name:** **tutorial\-securitygroup**
   + **Description:** **Tutorial Security Group**
   + **VPC:** Choose the VPC that you created earlier, for example: **vpc\-*identifier* \(tutorial\-vpc\)** 

1. Add inbound rules to the security group\.

   1. Determine the IP address to use to connect to instances in your VPC using Secure Shell \(SSH\)\. To determine your public IP address, in a different browser window or tab, you can use the service at [https://checkip\.amazonaws\.com](https://checkip.amazonaws.com)\. An example of an IP address is `203.0.113.25/32`\.

      If you are connecting through an Internet service provider \(ISP\) or from behind your firewall without a static IP address, you need to find out the range of IP addresses used by client computers\.
**Warning**  
  
If you use `0.0.0.0/0` for SSH access, you enable all IP addresses to access your public instances using SSH\. This approach is acceptable for a short time in a test environment, but it's unsafe for production environments\. In production, you'll authorize only a specific IP address or range of addresses to access your instances using SSH\.

   1. In the **Inbound rules** section, choose **Add rule**\.

   1. Set the following values for your new inbound rule to allow SSH access to your Amazon EC2 instance\. If you do this, you can connect to your Amazon EC2 instance to install the web server and other utilities, and to upload content for your web server\. 
      + **Type:** **SSH**
      + **Source:** The IP address or range from Step a, for example: **203\.0\.113\.25/32**\.

   1. Choose **Add rule**\.

   1. Set the following values for your new inbound rule to allow HTTP access to your web server\. 
      + **Type:** **HTTP**
      + **Source:** **0\.0\.0\.0/0**

1. To create the security group, choose **Create security group**\.

   Note the security group ID because you need it later in this tutorial\.

## Create a VPC security group for a private DB instance<a name="CHAP_Tutorials.WebServerDB.CreateVPC.SecurityGroupDB"></a>

To keep your DB instance private, create a second security group for private access\. To connect to private instances in your VPC, you add inbound rules to your VPC security group that allow traffic from your web server only\. 

**To create a VPC security group**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. Choose **VPC Dashboard**, choose **Security Groups**, and then choose **Create security group**\.

1. On the **Create security group** page, set these values:
   + **Security group name:** **tutorial\-db\-securitygroup**
   + **Description:** **Tutorial DB Instance Security Group**
   + **VPC:** Choose the VPC that you created earlier, for example: **vpc\-*identifier* \(tutorial\-vpc\)**

1. Add inbound rules to the security group\.

   1. In the **Inbound rules** section, choose **Add rule**\.

   1. Set the following values for your new inbound rule to allow MySQL traffic on port 3306 from your Amazon EC2 instance\. If you do this, you can connect from your web server to your DB instance to store and retrieve data from your web application to your database\. 
      + **Type:** **MySQL/Aurora**
      + **Source:** The identifier of the **tutorial\-securitygroup** security group that you created previously in this tutorial, for example: **sg\-9edd5cfb**\.

1. To create the security group, choose **Create security group**\.

## Create a DB subnet group<a name="CHAP_Tutorials.WebServerDB.CreateVPC.DBSubnetGroup"></a>

A DB subnet group is a collection of subnets that you create in a VPC and that you then designate for your DB instances\. A DB subnet group allows you to specify a particular VPC when creating DB instances\.

**To create a DB subnet group**

1. Identify the private subnets for your database in the VPC\.

   1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

   1. Choose **VPC Dashboard**, and then choose **Subnets**\.

   1. Note the subnet IDs of the subnets named **tutorial\-subnet\-private1\-us\-west\-2a** and **tutorial\-subnet\-private2\-us\-west\-2b**\.

      You will need the subnet IDs when you create your DB subnet group\.

1. Open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.
**Note**  
Make sure you connect to the Amazon RDS console, not to the Amazon VPC console\.

1. In the navigation pane, choose **Subnet groups**\.

1. Choose **Create DB subnet group**\.

1. On the **Create DB subnet group** page, set these values in **Subnet group details**:
   + **Name:** **tutorial\-db\-subnet\-group**
   + **Description:** **Tutorial DB Subnet Group**
   + **VPC:** **tutorial\-vpc \(vpc\-*identifier*\)** 

1. In the **Add subnets** section, choose the **Availability Zones** and **Subnets**\.

   For this tutorial, choose **us\-west\-2a** and **us\-west\-2b** for the **Availability Zones**\. For **Subnets**, choose the private subnets you identified in the previous step\.

1. Choose **Create**\. 

   Your new DB subnet group appears in the DB subnet groups list on the RDS console\. You can click the DB subnet group to see details, including all of the subnets associated with the group, in the details pane at the bottom of the window\.

**Note**  
If you created this VPC to complete [Tutorial: Create a web server and an Amazon Aurora DB cluster](TUT_WebAppWithRDS.md), create the DB cluster by following the instructions in [Create an Amazon Aurora DB cluster](CHAP_Tutorials.WebServerDB.CreateDBCluster.md)\.

## Deleting the VPC<a name="CHAP_Tutorials.WebServerDB.CreateVPC.Delete"></a>

After you create the VPC and other resources for this tutorial, you can delete them if they are no longer needed\.

**Note**  
If you added resources in the Amazon VPC you created for this tutorial, such as Amazon EC2 instances or Amazon RDS DB instances, you might need to delete these resources before you can delete the VPC\. For more information, see [Delete your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#VPC_Deleting) in the *Amazon VPC User Guide*\.

**To delete a VPC and related resources**

1. Delete the DB subnet group\.

   1. Open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

   1. In the navigation pane, choose **Subnet groups**\.

   1. Select the DB subnet group you want to delete, such as **tutorial\-db\-subnet\-group**\.

   1. Choose **Delete**, and then choose **Delete** in the confirmation window\.

1. Note the VPC ID\.

   1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

   1. Choose **VPC Dashboard**, and then choose **VPCs**\.

   1. In the list, identify the VPC you created, such as **tutorial\-vpc**\.

   1. Note the **VPC ID** of the VPC you created\. You will need the VPC ID in subsequent steps\.

1. Delete the security groups\.

   1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

   1. Choose **VPC Dashboard**, and then choose **Security Groups**\.

   1. Select the security group for the Amazon RDS DB instance, such as **tutorial\-db\-securitygroup**\.

   1. From **Actions**, choose **Delete security groups**, and then choose **Delete** on the confirmation page\.

   1. On the **Security Groups** page, select the security group for the Amazon EC2 instance, such as **tutorial\-securitygroup**\.

   1. From **Actions**, choose **Delete security groups**, and then choose **Delete** on the confirmation page\.

1. Delete the NAT gateway\.

   1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

   1. Choose **VPC Dashboard**, and then choose **NAT Gateways**\.

   1. Select the NAT gateway of the VPC you created\. Use the VPC ID to identify the correct NAT gateway\.

   1. From **Actions**, choose **Delete NAT gateway**\.

   1. On the confirmation page, enter **delete**, and then choose **Delete**\.

1. Delete the VPC\.

   1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

   1. Choose **VPC Dashboard**, and then choose **VPCs**\.

   1. Select the VPC you want to delete, such as **tutorial\-vpc**\.

   1. From **Actions**, choose **Delete VPC**\.

      The confirmation page shows other resources that are associated with the VPC that will also be deleted, including the subnets associated with it\.

   1. On the confirmation page, enter **delete**, and then choose **Delete**\.

1. Release the Elastic IP addresses\.

   1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

   1. Choose **Amazon EC2 Dashboard**, and then choose **Elastic IPs**\.

   1. Select the Elastic IP address you want to release\.

   1. From **Actions**, choose **Release Elastic IP addresses**\.

   1. On the confirmation page, choose **Release**\.