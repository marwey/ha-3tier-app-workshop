---
title: "Lab 2: Set up your RDS database"
weight: 200
---

### Create database security groups

Visit the [AWS VPC console](https://console.aws.amazon.com/vpc/home) and create 2 security groups.  The first security group should be named something like *WP Database Client SG* and the second security group should be named *WP Database SG*.  

With both groups created edit the **Inbound Rules** of the *WP Database SG* security group and create a rule of type *MySQL / Aurora* which allows traffic on port `3306` from the *WP Database Client SG*.

Now you are ready to create your RDS database.

### Create an RDS subnet group

Amazon RDS provides managed database deployments.  When you use Amazon RDS to deploy a database in a highly available fashion it will create 2 instances in 2 different availability zones.  To do this, when you create a database deployment you specify a subnet group which tells RDS in which subnets it can deploy your database instances.  

To create a DB subnet group browse to the [Amazon RDS console](https://console.aws.amazon.com/rds/home), click on **Subnet groups** in the panel on your left, click on the **Create DB Subnet Group** button and use the following details:


![Figure 1](/images/rds1.png)

Scroll down and add the two Data subnets created earlier (one for each AZ) to your new subnet group:

![Figure 2](/images/rds2.png)

### Create the Aurora database cluster

Once the subnet group has been created you are ready to launch your RDS-managed database.  Click **Databases** on the left and click **Create database** and use the following details:

![Figure 3](/images/rds3.png)

Select the MySQL-compatible Aurora engine:

![Figure 4](/images/rds4.png)

When prompted about the Master password, you can either click on Auto generate a password or create your own – in either case please make sure you write down the password as it will be required a few steps later when setting up the DB connectivity of the Wordpress instances.

![Figure 5](/images/rds5.png)

Select the DB instance size together with a Multi-AZ deployment, required for high availability:

![Figure 6](/images/rds6.png)

In the connectivity section, make sure you select the correct VPC, together with the DB subnet group, and security group created earlier:

![Figure 7](/images/rds7.png)

Expand **Additional configuration** and specify an **Initial database name** of *wordpress*.

Finally, click on **Create database** to start building the cluster:

![Figure 8](/images/rds8.png)

The database will take a few minutes to be provisioned and made available.  

---

You're active / passive database should now be available and running in two different availability zones, waiting for connections from any EC2 resource with the client security group associated to it.  

Wordpress uses its database to house articles, users, and configuration information.  This can result in a lot of unnecessary queries against the database which consistently return the same data set.  To offset the strain on the database and improve performance of the Wordpress web application let's add a caching layer for common SQL requests.  In the next lab you will deploy a database caching layer for use by Wordpress.
