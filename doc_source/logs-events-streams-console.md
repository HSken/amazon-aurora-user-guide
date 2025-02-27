# Viewing logs, events, and streams in the Amazon RDS console<a name="logs-events-streams-console"></a>

Amazon RDS integrates with AWS services to show information about logs, events, and database activity streams in the RDS console\.

The **Logs & events** tab for your Aurora DB cluster shows the following information:
+ **Auto scaling policies and activities** – Shows policies and activities relating to the Aurora Auto Scaling feature\. This information only appears in the **Logs & events** tab at the cluster level\. 
+ **Amazon CloudWatch alarms** – Shows any metric alarms that you have configured for the DB instance in your Aurora cluster\. If you haven't configured alarms, you can create them in the RDS console\. 
+ **Recent events** – Shows a summary of events \(environment changes\) for your Aurora DB instance or cluster\. For more information, see [Viewing Amazon RDS events](USER_ListEvents.md)\.
+ **Logs** – Shows database log files generated by a DB instance in your Aurora cluster\. For more information, see [Monitoring Amazon Aurora log files](USER_LogAccess.md)\.

The **Configuration** tab displays information about database activity streams\.

**To view logs, events, and streams for your Aurora DB cluster in the RDS console**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the navigation pane, choose **Databases**\.

1. Choose the name of the Aurora DB cluster that you want to monitor\.

   The database page appears\. The following example shows an Amazon Aurora PostgreSQL DB cluster named `apga`\.  
![\[Database page with monitoring tab shown\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/cluster-with-monitoring-tab.png)

1. Scroll down and choose **Configuration**\.

   The following example shows the status of the database activity streams for your cluster\.  
![\[Enhanced Monitoring\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/cluster-das.png)

1. Choose **Logs & events**\.

   The Logs & events section appears\.  
![\[Database page with Logs & events tab shown\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/cluster-logs-and-events-subpage.png)

1. Choose a DB instance in your Aurora cluster, and then choose **Logs & events** for the instance\.

   The following example shows that the contents are different between the DB instance page and the DB cluster page\. The DB instance page shows logs and alarms\.  
![\[Logs & events page\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/cluster-instance-logs-and-events-subpage.png)