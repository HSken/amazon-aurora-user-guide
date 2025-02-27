# Amazon Virtual Private Cloud VPCs and Amazon Aurora<a name="USER_VPC"></a>

Amazon Virtual Private Cloud \(Amazon VPC\) enables you to launch AWS resources, such as Aurora DB clusters, into a virtual private cloud \(VPC\)\. 

When you use an Amazon VPC, you have control over your virtual networking environment: you can choose your own IP address range, create subnets, and configure routing and access control lists\. There is no additional cost to run your DB instance in an Amazon VPC\. 

Accounts that support only the *EC2\-VPC* platform have a default VPC\. All new DB instances are created in the default VPC unless you specify otherwise\.

**Topics**
+ [Working with a DB instance in a VPC](USER_VPC.WorkingWithRDSInstanceinaVPC.md)
+ [Scenarios for accessing a DB instance in a VPC](USER_VPC.Scenarios.md)
+ [Tutorial: Create an Amazon VPC for use with a DB instance](CHAP_Tutorials.WebServerDB.CreateVPC.md)

This documentation only discusses VPC functionality relevant to Amazon Aurora DB clusters\. For more information about Amazon VPC, see [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/) and [Amazon VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)\. For information about using a network address translation \(NAT\) gateway, see [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) in the *Amazon Virtual Private Cloud User Guide*\. 