---
title: "Lab 2: Set up your RDS database"
weight: 200
---

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

In the connectivity section, make sure you select the correct VPC, together with the DB subnet group created earlier:

![Figure 7](/images/rds7.png)

Finally, click on Create database to start building the cluster:

![Figure 8](/images/rds8.png)

The database will take a few minutes to be provisioned and made available.  While it is being setup you will modify the security group that you specified.  You will create a *database client* security group and only allow this security group to access your database.  To do this start by visiting the [AWS VPC console](https://console.aws.amazon.com/vpc/home).

Click on **Security Groups** in the left-hand side menu to get started.  Next create the client security group by clicking **Create security group** and give your client security group a name such as `WP Database Client SG`.  Give your security group a description and associate it with your VPC, then click **Create**.

Return to the list of security groups and select the security group that was created for your database.  If you review the **Inbound Rules** it will show a single rule allowing communication with the database from the IP address of the device you used to create the database.  Modify this rule to only allow communication from the security group you created in the previous step.

---

You're active / passive database should now be available and running in two different availability zones, waiting for connections from any EC2 resource with the client security group associated to it.  

Wordpress uses its database to house articles, users, and configuration information.  This can result in a lot of unnecessary queries against the database which consistently return the same data set.  To offset the strain on the database and improve performance of the Wordpress web application let's add a caching layer for common SQL requests.  In the next lab you will deploy a database caching layer for use by Wordpress.
