# Daabase on AWS

- Table Of Contents
    - [Relational Database Service (RDS)](#RDS)
    - [Dynamo DB](#dynamo)
    - [Excercise](#Excercise)

<a name='RDS'></a>

##  Amazon Relational Database Service (RDS)
### What Is Amazon RDS?
Amazon RDS enables you to create and manage relational databases in the cloud without the operational burden of traditional database management. For example, if you sell healthcare equipment and your goal is to be the number-one seller in the Pacific Northwest, building out a database doesn’t directly help you achieve that goal though having a database is necessary to achieve the goal.   Amazon RDS helps you offload some of this unrelated work of creating and managing a database. You can focus on the tasks that differentiate your application, instead of infrastructure-related tasks such as provisioning, patching, scaling, and restoring.  Amazon RDS supports most of the popular relational database management systems, ranging from commercial options, open source options, and even an AWS-specific option. Here are the supported Amazon RDS engines. 
- Commercial: Oracle, SQL Server
- Open Source: MySQL, PostgreSQL, MariaDB
- Cloud Native: Amazon Aurora

![RDS-Vision](/images/rds.png)
<br>
Note: The cloud native option, Amazon Aurora, is a MySQL and PostgreSQL-compatible database built for the cloud. It is more durable, more available, and provides faster performance than the Amazon RDS version of MySQL and PostgreSQL. 

### Understand DB Instances
Just like the databases that you would build and manage yourself, Amazon RDS is built off of compute and storage. The compute portion is called the DB (database) instance, which runs the database engine. Depending on the engine of the DB instance you choose, the engine will have different supported features and configurations. A DB instance can contain multiple databases with the same engine, and each database can contain multiple tables.  Underneath the DB instance is an EC2 instance. However, this instance is managed through the Amazon RDS console instead of the Amazon EC2 console. When you create your DB instance, you choose the instance type and size. Amazon RDS supports three instance families.
- Standard, which include general-purpose instances
- Memory Optimized, which are optimized for memory-intensive applications
- Burstable Performance, which provides a baseline performance level, with the ability to burst to full CPU usage.
![rds-2](/images/rds-2.png)
<br>

The DB instance you choose affects how much processing power and memory it has. Not all of the options are available to you, depending on the engine that you choose. You can find more information about the DB instance types in the resources section of this unit.  Much like a regular EC2 instance, the DB instance uses Amazon Elastic Block Store (EBS) volumes as its storage layer. You can choose between three types of EBS volume storage.
- General purpose (SSD)
- Provisioned IOPS (SSD)
- Magnetic storage (not recommended)
![rds-3](/images/rds-3.png)
<br>

### Work with Amazon RDS in an Amazon Virtual Private Cloud
When you create a DB instance, you select the Amazon Virtual Private Cloud (VPC) that your databases will live in. Then, you select the subnets that you want the DB instances to be placed in. This is referred to as a DB subnet group. To create a DB subnet group, you specify:
- The Availability Zones (AZs) that include the subnets you want to add
- The subnets in that AZ where your DB instance are placed

The subnets you add should be private so they don’t have a route to the internet gateway. This ensures your DB instance, and the cat data inside of it, can only be reached by the app backend.  Access to the DB instance can be further restricted by using network access control lists (ACLs) and security groups. With these firewalls, you can control, at a granular level, what type of traffic you want to allow into your database.  Using these controls provide layers of security for your infrastructure. It reinforces that only the backend instances have access to the database.


### Back Up Your Data
You don’t want to lose any of that precious cat information. To take regular backups of your RDS instance, you can use: 
- Automatic backups
- Manual snapshots

#### Automatic Backups 

Automated backups are turned on by default. This backs up your entire DB instance (not just individual databases on the instance), and your transaction logs. When you create your DB instance, you set a backup window that is the period of time that automatic backups occur. Typically, you want to set these windows during a time when your database experiences little activity because it can cause increased latency and downtime.  

You can retain your automated backups between 0 and 35 days. You might ask yourself, “Why set automated backups for 0 days?” The 0 days setting actually disables automatic backups from happening. Keep in mind that if you set it to 0, it will also delete all existing automated backups. This is not ideal, as the benefit of having automated backups is having the ability to do point-in-time recovery.    
![rds-4](/images/rds-4.png)
<br>

If you restore data from an automated backup, you have the ability to do point-in-time recovery. Point-in-time recovery creates a new DB instance using data restored from a specific point in time. This restoration method provides more granularity by restoring the full backup and rolling back transactions up to the specified time range.  


#### Manual Snapshots 

If you want to keep your automated backups longer than 35 days, use manual snapshots. Manual snapshots are similar to taking EBS snapshots, except you manage them in the RDS console. These are backups that you can initiate at any time, that exist until you delete them.  

For example, to meet a compliance requirement that mandates you to keep database backups for a year, you would need to use manual snapshots to ensure those backups are retained for that period of time.  

If you restore data from a manual snapshot, it creates a new DB instance using the data from the snapshot.
![rds-5](/images/rds-5.png)
<br>

### Which Backup Option Should I Use?
The answer, almost always, is both. Automated backups are beneficial for the point-in-time recovery. Manual snapshots allow you to retain backups for longer than 35 days.  

### Get Redundancy with Amazon RDS Multi-AZ
When you enable Amazon RDS Multi-AZ, Amazon RDS creates a redundant copy of your database in another AZ. You end up with two copies of your database: a primary copy in a subnet in one AZ and a standby copy in a subnet in a second AZ.   

The primary copy of your database provides access to your data so that applications can query and display that information.   

The data in the primary copy is synchronously replicated to the standby copy. The standby copy is not considered an active database, and does not get queried by applications.  

To improve availability, Amazon RDS Multi-AZ ensures that you have two copies of your database running and that one of them is in the primary role. If there’s an availability issue, such as the primary database losing connectivity, Amazon RDS triggers an automatic failover.  

When you create a DB instance, a domain name system (DNS) name is provided. AWS uses that DNS name to failover to the standby database. In an automatic failover, the standby database is promoted to the primary role and queries are redirected to the new primary database.   


<a name='dynamo'></a>

##  Introduction to Amazon DynamoDB

### What Is Amazon DynamoDB?
Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. DynamoDB lets you offload the administrative burdens of operating and scaling a distributed database so that you don't have to worry about hardware provisioning, setup and configuration, replication, software patching, or cluster scaling. 

With DynamoDB, you can create database tables that can store and retrieve any amount of data and serve any level of request traffic. You can scale up or scale down your tables' throughput capacity without downtime or performance degradation. You can use the AWS Management Console to monitor resource utilization and performance metrics.

DynamoDB automatically spreads the data and traffic for your tables over a sufficient number of servers to handle your throughput and storage requirements, while maintaining consistent and fast performance. All of your data is stored on solid-state disks (SSDs) and is automatically replicated across multiple Availability Zones in an AWS Region, providing built-in high availability and data durability. 


### Core Components of Amazon DynamoDB
In DynamoDB, tables, items, and attributes are the core components that you work with. A table is a collection of items, and each item is a collection of attributes. DynamoDB uses primary keys to uniquely identify each item in a table and secondary indexes to provide more querying flexibility. 

The following are the basic DynamoDB components:

Tables – Similar to other database systems, DynamoDB stores data in tables. A table is a collection of data. For example, see the example table called People that you could use to store personal contact information about friends, family, or anyone else of interest. You could also have a Cars table to store information about vehicles that people drive.

Items – Each table contains zero or more items. An item is a group of attributes that is uniquely identifiable among all of the other items. In a People table, each item represents a person. For a Cars table, each item represents one vehicle. Items in DynamoDB are similar in many ways to rows, records, or tuples in other database systems. In DynamoDB, there is no limit to the number of items you can store in a table.

Attributes – Each item is composed of one or more attributes. An attribute is a fundamental data element, something that does not need to be broken down any further. For example, an item in a People table contains attributes called PersonID, LastName, FirstName, and so on. For a Department table, an item might have attributes such as DepartmentID, Name, Manager, and so on. Attributes in DynamoDB are similar in many ways to fields or columns in other database systems.

<a name='Excercise'></a>

## Exercise 6: Setting up the Database
In this scenario, part of your responsibility is to keep the employee database up to date.

In this exercise, you first launch another EC2 instance. Then, you create the DynamoDB table for the employee directory application.

### Task 1: Launching an EC2 instance
In this task, you will launch another EC2 instance.

1. If needed, log in to the AWS Management Console as your Admin user.
2. Open the Amazon EC2 console by searching for and choosing EC2.
3. In the navigation pane, choose Instances.
4. If needed, select the check box for the employee-directory-app-s3 instance, which should be in the Stopped state.
5. Choose Actions and then choose Image and templates, Launch more like this.
6. For Name and at the end of the Value, append -exercise6.
    - Example:
    ```
    employee-directory-app-exercise6
    ```
7. For Key pair name, select app-key-pair.
8. Under Network settings and Auto-assign Public IP, choose Enable.
9. Choose Launch instance.
10. Choose View all instances.
    - The instance should now be in the Instances list.
11. Wait for the Instance state to change to Running and the Status check to change to 2/2 checks passed.
12. If needed, clear the check box for the employee-directory-app-s3 instance.
13. Select the check box for employee-directory-app-exercise6.
14. Copy the Public IPv4 address.
    - Note: Make sure that you only copy the address instead of choosing the open address link.
15. In a new browser window, paste the IP address that you copied. Make sure to remove the ‘S’ after HTTP so you are using only HTTP instead.

### Task 2: Creating the DynamoDB table
To connect the application to a database, you first need to create one! In this task, you will create a database by using DynamoDB.
1. Return to the console, and search for and open DynamoDB.
2. In the navigation pane, choose Tables.
3. Choose Create table and configure the following settings.
    - Table name: Employees
    - Partition key: id
4. Choose Create table.

### Task 3: Testing the application
In this task, you will test whether the application works by using it to create an employee entry and upload a photo.
1. Return to the Amazon EC2 console by searching for and opening EC2.
2. In the instance list, select the check box for the employee-directory-app-exercise6 instance.
3. On the Details tab, copy the Public IPv4 address and in a new browser window, paste the IP address.
4. In the application interface, choose Add.
5. Create a new employee entry by entering a name, location, and job title, and by selecting some attributes.
6. Upload a picture by choosing Browse and uploading a picture of your choice.
7. Choose Save.
8. Create and save a few employee entries.

### Task 4: Viewing the item in the database
In this task, you will see how the employee entries are stored in DynamoDB.
1. Return to the console, and search for and open DynamoDB.
2. In the navigation pane, choose Tables.
3. Open the table details by choosing the Employees link.
4. Choose Explore table items.
5. In the Items returned list, you should now see the entries in the database that you made by using the application on the EC2 instance.

Congratulations! You launched an EC2 instance that uses the S3 bucket, and is connected to the DynamoDB table.

### Task 5: Stopping your EC2 instance
In this task, you will now stop the instance to prevent future costs from incurring.

Note: Do not terminate the instance because you will use it in the next exercise.
1. Return to the Amazon EC2 console by searching for and opening EC2.
2. If needed, in the navigation pane, choose Instances.
3. Select the check box for employee-directory-app-exercise6.
4. Choose Instance state and then choose Stop instance.
5. In the dialog box, choose Stop.
6. The Instance state will eventually go into the Stopped state.

