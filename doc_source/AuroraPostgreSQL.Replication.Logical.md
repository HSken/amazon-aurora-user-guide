# Using PostgreSQL logical replication with Aurora<a name="AuroraPostgreSQL.Replication.Logical"></a>

PostgreSQL logical replication provides fine\-grained control over replicating and synchronizing parts of a database\. For example, you can use logical replication to replicate an individual table of a database\. 

Following, you can find information about how to work with PostgreSQL logical replication and Amazon Aurora\. For more detailed information about the PostgreSQL implementation of logical replication, see [Logical replication](https://www.postgresql.org/docs/current/logical-replication.html) and [Logical decoding concepts](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html) in the PostgreSQL documentation\.

**Note**  
Logical replication is available with Aurora PostgreSQL version 2\.2\.0 \(compatible with PostgreSQL 10\.6\) and later\.

Following, you can find information about how to work with PostgreSQL logical replication and Amazon Aurora\.

**Topics**
+ [Configuring logical replication](#AuroraPostgreSQL.Replication.Logical.Configure)
+ [Example of logical replication of a database table](#AuroraPostgreSQL.Replication.Logical.PostgreSQL-Example)
+ [Logical replication using the AWS Database Migration Service](#AuroraPostgreSQL.Replication.Logical.DMS-Example)
+ [Stopping logical replication](#AuroraPostgreSQL.Replication.Logical.Stop)

## Configuring logical replication<a name="AuroraPostgreSQL.Replication.Logical.Configure"></a>

To use logical replication, you first set the `rds.logical_replication` parameter for a cluster parameter group\. You then set up the publisher and subscriber\.

Logical replication uses a publish and subscribe model\. *Publishers* and *subscribers* are the nodes\. A *publication* is a set of changes generated from one or more database tables\. You specify a publication on a publisher\. A *subscription* defines the connection to another database and one or more publications to which it subscribes\. You specify a subscription on a subscriber\. The publication and subscription make the connection between the publisher and subscriber\. The replication process uses a *replication slot* to track the progress of a subscription\.

**Note**  
Following are requirements for logical replication:  
To set up logical replication, you need to have `rds_superuser` permissions\.
The source instance \(publisher node\) and the receiving instance \(subscriber node\) must each have automated backups turned on\. For more information, see [Enabling automated backups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html#USER_WorkingWithAutomatedBackups.Enabling) in the *Amazon RDS User Guide*\.

**To enable PostgreSQL logical replication with Aurora**

1. Create a new DB cluster parameter group to use for logical replication, as described in [Creating a DB cluster parameter group](USER_WorkingWithDBClusterParamGroups.md#USER_WorkingWithParamGroups.CreatingCluster)\. Use the following settings:
   + For **Parameter group family**, choose your version of Aurora PostgreSQL, such as **aurora\-postgresql12**\. 
   + For **Type**, choose **DB Cluster Parameter Group**\. 

1. Modify the DB cluster parameter group, as described in [Modifying parameters in a DB cluster parameter group](USER_WorkingWithDBClusterParamGroups.md#USER_WorkingWithParamGroups.ModifyingCluster)\. Set the `rds.logical_replication` static parameter to 1\. 

   Enabling the `rds.logical_replication` parameter affects the DB cluster's performance\. 

1. Review the `max_replication_slots`, `max_wal_senders`, `max_logical_replication_workers`, and `max_worker_processes` parameters in your DB cluster parameter group based on your expected usage\. If necessary, modify the DB cluster parameter group to change the settings for these parameters, as described in [Modifying parameters in a DB cluster parameter group](USER_WorkingWithDBClusterParamGroups.md#USER_WorkingWithParamGroups.ModifyingCluster)\.

   Follow these guidelines for setting the parameters:
   + `max_replication_slots` – A *replication slot* tracks the progress of a subscription\. Set the value of the `max_replication_slots` parameter to the total number of subscriptions that you plan to create\. If you are using AWS DMS, set this parameter to the number of AWS DMS tasks that you plan to use for change data capture from this DB cluster\.
   + `max_wal_senders` and `max_logical_replication_workers` – Ensure that `max_wal_senders` and `max_logical_replication_workers` are each set at least as high as the number of logical replication slots that you intend to be active, or the number of active AWS DMS tasks for change data capture\. Leaving a logical replication slot inactive prevents the vacuum from removing obsolete tuples from tables, so we recommend that you don't keep inactive replication slots for long periods of time\.
   + `max_worker_processes` – Ensure that `max_worker_processes` is at least as high as the combined values of `max_logical_replication_workers`, `autovacuum_max_workers`, and `max_parallel_workers`\. Having a high number of background worker processes might affect application workloads on small DB instance classes, so monitor the performance of your database if you set `max_worker_processes` higher than the default value\.

**To configure a publisher for logical replication**

1. Set the publisher's cluster parameter group:
   + To use an existing Aurora PostgreSQL DB cluster for the publisher, the engine version must be 10\.6 or later\. Do the following:

     1. Modify the DB cluster parameter group to set it to the group that you created when you enabled logical replication\. For details about modifying an Aurora PostgreSQL DB cluster, see [Modifying an Amazon Aurora DB cluster](Aurora.Modifying.md)\.

     1. Restart the DB cluster for static parameter changes to take effect\. The DB cluster parameter group includes a change to the static parameter `rds.logical_replication`\.
   + To use a new Aurora PostgreSQL DB cluster for the publisher, create the DB cluster using the following settings\. For details about creating an Aurora PostgreSQL DB cluster, see [Creating a DB cluster](Aurora.CreateInstance.md#Aurora.CreateInstance.Creating)\.

     1. Choose the **Amazon Aurora** engine and choose the **PostgreSQL\-compatible** edition\.

     1. For **Engine version**, choose an Aurora PostgreSQL engine that is compatible with PostgreSQL 10\.6 or greater\.

     1. For **DB cluster parameter group**, choose the group that you created when you enabled logical replication\.

1. Modify the inbound rules of the security group for the publisher to allow the subscriber to connect\. Usually, you do this by including the IP address of the subscriber in the security group\. For details about modifying a security group, see [Security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) in the *Amazon Virtual Private Cloud User Guide*\. 

## Example of logical replication of a database table<a name="AuroraPostgreSQL.Replication.Logical.PostgreSQL-Example"></a>

To implement logical replication, use the PostgreSQL commands `CREATE PUBLICATION` and `CREATE SUBSCRIPTION`\. 

For this example, table data is replicated from an Aurora PostgreSQL database as the publisher to a PostgreSQL database as the subscriber\. Note that a subscriber database can be an RDS PostgreSQL database or an Aurora PostgreSQL database\. A subscriber can also be an application that uses PostgreSQL logical replication\. After the logical replication mechanism is set up, changes on the publisher are continually sent to the subscriber as they occur\. 

To set up logical replication for this example, do the following:

1. Configure an Aurora PostgreSQL DB cluster as the publisher\. To do so, create a new Aurora PostgreSQL DB cluster, as described when configuring the publisher in [Configuring logical replication](#AuroraPostgreSQL.Replication.Logical.Configure)\.

1. Set up the publisher database\. 

   For example, create a table using the following SQL statement on the publisher database\. 

   ```
   CREATE TABLE LogicalReplicationTest (a int PRIMARY KEY);
   ```

1. Insert data into the publisher database by using the following SQL statement\.

   ```
   INSERT INTO LogicalReplicationTest VALUES (generate_series(1,10000));
   ```

1. Create a publication on the publisher by using the following SQL statement\. 

   ```
   CREATE PUBLICATION testpub FOR TABLE LogicalReplicationTest;
   ```

1. Create your subscriber\. A subscriber database can be either of the following:
   + Aurora PostgreSQL database version 2\.2\.0 \(compatible with PostgreSQL 10\.6\) or later\. 
   + Amazon RDS for PostgreSQL database with the PostgreSQL DB engine version 10\.4 or later\.

   For this example, we create an Amazon RDS for PostgreSQL database as the subscriber\. For details on creating a DB instance, see [Creating a DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html) in the *Amazon RDS User Guide\.* 

1. Set up the subscriber database\. 

   For this example, create a table like the one created for the publisher by using the following SQL statement\. 

   ```
   CREATE TABLE LogicalReplicationTest (a int PRIMARY KEY);
   ```

1. Verify that there is data in the table at the publisher but no data yet at the subscriber by using the following SQL statement on both databases\.

   ```
   SELECT count(*) FROM LogicalReplicationTest;
   ```

1. Create a subscription on the subscriber\. 

   Use the following SQL statement on the subscriber database and the following settings from the publisher cluster: 
   + **host** – The publisher cluster's writer DB instance\.
   + **port** – The port on which the writer DB instance is listening\. The default for PostgreSQL is 5432\.
   + **dbname** – The DB name of the publisher cluster\.

   ```
   CREATE SUBSCRIPTION testsub CONNECTION 
      'host=publisher-cluster-writer-endpoint port=5432 dbname=db-name user=user password=password' 
      PUBLICATION testpub;
   ```

   After the subscription is created, a logical replication slot is created at the publisher\.

1. To verify for this example that the initial data is replicated on the subscriber, use the following SQL statement on the subscriber database\.

   ```
   SELECT count(*) FROM LogicalReplicationTest;
   ```

Any further changes on the publisher are replicated to the subscriber\.

## Logical replication using the AWS Database Migration Service<a name="AuroraPostgreSQL.Replication.Logical.DMS-Example"></a>

You can use the AWS Database Migration Service \(AWS DMS\) to replicate a database or a portion of a database\. Use AWS DMS to migrate your data from an Aurora PostgreSQL database to another open source or commercial database\. For more information about AWS DMS, see the [AWS Database Migration Service User Guide](https://docs.aws.amazon.com/dms/latest/userguide/)\.

The following example shows how to set up logical replication from an Aurora PostgreSQL database as the publisher and then use AWS DMS for migration\. This example uses the same publisher and subscriber that were created in [Example of logical replication of a database table](#AuroraPostgreSQL.Replication.Logical.PostgreSQL-Example)\.

To set up logical replication with AWS DMS, you need details about your publisher and subscriber from Amazon RDS\. In particular, you need details about the publisher's writer DB instance and the subscriber's DB instance\.

Get the following information for the publisher's writer DB instance:
+ The virtual private cloud \(VPC\) identifier
+ The subnet group
+ The Availability Zone \(AZ\)
+ The VPC security group
+ The DB instance ID

Get the following information for the subscriber's DB instance:
+ The DB instance ID
+ The source engine

**To use AWS DMS for logical replication with Aurora PostgreSQL**

1. Prepare the publisher database to work with AWS DMS\. 

   To do this, PostgreSQL 10\.x and later databases require that you apply AWS DMS wrapper functions to the publisher database\. For details on this and later steps, see the instructions in [Using PostgreSQL version 10\.x and later as a source for AWS DMS](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.PostgreSQL.html#CHAP_Source.PostgreSQL.v10) in the *AWS Database Migration Service User Guide\.*

1. Sign in to the AWS Management Console and open the AWS DMS console at [https://console.aws.amazon.com/dms/v2](https://console.aws.amazon.com/dms/v2)\. At top right, choose the same AWS Region in which the publisher and subscriber are located\.

1. Create an AWS DMS replication instance\.

   Choose values that are the same as for your publisher's writer DB instance\. These include the following settings:
   + For **VPC**, choose the same VPC as for the writer DB instance\.
   + For **Replication Subnet Group**, choose a subnet group with the same values as the writer DB instance\. Create a new one if necessary\.
   + For **Availability zone**, choose the same zone as for the writer DB instance\.
   + For **VPC Security Group**, choose the same group as for the writer DB instance\.

1. Create an AWS DMS endpoint for the source\. 

   Specify the publisher as the source endpoint by using the following settings: 
   + For **Endpoint type**, choose **Source endpoint**\. 
   + Choose **Select RDS DB Instance**\.
   + For **RDS Instance**, choose the DB identifier of the publisher's writer DB instance\.
   + For **Source engine**, choose **postgres**\.

1. Create an AWS DMS endpoint for the target\. 

   Specify the subscriber as the target endpoint by using the following settings:
   + For **Endpoint type**, choose **Target endpoint**\. 
   + Choose **Select RDS DB Instance**\.
   + For **RDS Instance**, choose the DB identifier of the subscriber DB instance\.
   + Choose a value for **Source engine**\. For example, if the subscriber is an RDS PostgreSQL database, choose **postgres**\. If the subscriber is an Aurora PostgreSQL database, choose **aurora\-postgresql**\.

1. Create an AWS DMS database migration task\. 

   You use a database migration task to specify what database tables to migrate, to map data using the target schema, and to create new tables on the target database\. At a minimum, use the following settings for **Task configuration**:
   + For **Replication instance**, choose the replication instance that you created in an earlier step\.
   + For **Source database endpoint**, choose the publisher source that you created in an earlier step\.
   + For **Target database endpoint**, choose the subscriber target that you created in an earlier step\.

   The rest of the task details depend on your migration project\. For more information about specifying all the details for DMS tasks, see [Working with AWS DMS tasks](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.html) in the *AWS Database Migration Service User Guide\.*

After AWS DMS creates the task, it begins migrating data from the publisher to the subscriber\. 

## Stopping logical replication<a name="AuroraPostgreSQL.Replication.Logical.Stop"></a>

You can stop using logical replication\.

**To stop using logical replication**

1. Drop all replication slots\.

   To drop all of the replication slots, connect to the publisher and run the following SQL command

   ```
   SELECT pg_drop_replication_slot(slot_name) FROM pg_replication_slots 
      WHERE slot_name IN (SELECT slot_name FROM pg_replication_slots);
   ```

   The replication slots can't be active when you run this command\.

1. Modify the DB cluster parameter group associated with the publisher, as described in [Modifying parameters in a DB cluster parameter group](USER_WorkingWithDBClusterParamGroups.md#USER_WorkingWithParamGroups.ModifyingCluster)\. Set the `rds.logical_replication` static parameter to 0\. 

1. Restart the publisher DB cluster for the change to the `rds.logical_replication` static parameter to take effect\.