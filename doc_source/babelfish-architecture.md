# Babelfish architecture<a name="babelfish-architecture"></a>

When you create an Aurora PostgreSQL cluster with Babelfish turned on, Aurora provisions the cluster with a PostgreSQL database named `babelfish_db`\. This database is where all migrated SQL Server objects and structures reside\. 

**Note**  
In an Aurora PostgreSQL cluster, the `babelfish_db` database name is reserved for Babelfish\. Creating your own "babelfish\_db" database on a Babelfish DB cluster prevents Aurora from successfully provisioning Babelfish\. 

When you connect to the TDS port, the session is placed in the `babelfish_db` database\. From T\-SQL, the structure looks similar to being connected to a SQL Server instance\. You can see the `master`, `msdb`, and `tempdb` databases, and the `sys.databases` catalog\. You can create additional user databases and switch between databases with the USE statement\. When you create a SQL Server user database, it's flattened into the `babelfish_db` PostgreSQL database\. Your database retains cross\-database syntax and semantics equal to or similar to those provided by SQL Server\.

## Using Babelfish with a single database or multiple databases<a name="babelfish-single_vs_multi_db"></a>

When you create an Aurora PostgreSQL cluster to use with Babelfish, you choose between using a single SQL Server database on its own or multiple SQL Server databases together\. Your choice affects how the names of SQL Server schemas inside the `babelfish_db` database appear from Aurora PostgreSQL\. The migration mode is stored in the `migration_mode` parameter\. You can't change this parameter after creating your cluster\.

In single\-database mode, the schema names of the user database in the `babelfish_db` database remain the same as in SQL Server\. If you choose to move a single database, schemas are recreated inside of the database and can be referenced with the same name used with SQL Server\. For example, the `dbo` and `smith` schemas reside inside the `dbA` database*\.* 

![\[\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/babelfish_single_db_mode.png)

When connecting through TDS, you can run `USE dbA` to see schemas `dbo` and `smith` from T\-SQL, as you would in SQL Server\. The unchanged schema names are also visible from PostgreSQL\.

In multiple\-database mode, the schema names of user databases become `dbname_schemaname` when seen from PostgreSQL\. The schema names remain the same when seen from T\-SQL\.

![\[\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/images/babelfish_multi_db_mode.png)

As shown in the image, multiple\-database mode and single\-database mode are the same as SQL Server when connecting through the TDS port and using T\-SQL\. For example, `USE dbA` lists schemas `dbo` and  `smith` just as it does in SQL Server\. The mapped schema names, such as `dbA_dbo`  and `dbA_smith`, are visible from PostgreSQL\.

Each database still contains your schemas\. The name of each database is prepended to the name of the SQL Server schema, using an underscore as a delimiter, for example:
+ `dbA` contains `dbA_dbo` and `dbA_smith`\.
+ `dbB` contains `dbB_dbo` and `dbB_jones`\.
+ `dbC` contains `dbC_dbo` and `dbC_miller`\.

Inside the `babelfish_db` database, the T\-SQL user still needs to run `USE dbname` to change database context, so the look and feel remains similar to SQL Server\.

## Choosing a migration mode<a name="babelfish-choosing_single_vs_multi"></a>

Each migration mode has advantages and disadvantages\. Choose your migration mode based on the number of user databases you have, and your migration plans\. After you create a cluster for use with Babelfish, you can't change the migration mode\. When choosing a migration mode, consider the requirements of your user databases and clients\.

When you create a cluster for use with Babelfish, Aurora PostgreSQL creates the system databases, `master` and `tempdb`\. If you created or modified objects in the system databases \(`master` or `tempdb`\), make sure to recreate those objects in your new cluster\. Unlike SQL Server, Babelfish doesn't reinitialize `tempdb` after a cluster reboot\.

Use single database migration mode in the following cases:
+ If you are migrating a single SQL Server database\. In single database mode, migrated schema names are identical to the original SQL Server schema names\. When you migrate your application, you make fewer changes to your SQL code\.
+ If your end goal is a complete migration to native Aurora PostgreSQL\. Before migrating, consolidate your schemas into a single schema \(`dbo`\) and then migrate into a single cluster to lessen required changes\.

Use multiple database migration mode in the following cases:
+ If you are trying out Babelfish and you aren't sure of your future needs\. 
+ If multiple user databases need to be migrated together, and the end goal isn't to perform a fully native PostgreSQL migration\.
+ If you might be migrating multiple databases in the future\.