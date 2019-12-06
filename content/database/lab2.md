---
title: "Lab 2: Set up your RDS database"
weight: 200
---

### Create an RDS subnet group

Your DB subnet group identifies the subnets that your DB cluster uses from the VPC that you created in the previous steps. Your DB subnet group must include at least one subnet in at least two of the Availability Zones in the AWS Region where you want to deploy your DB cluster.

To create a DB subnet group browse to your RDS dashboard, click on **Subnet groups** in the panel on your left, click on the **Create DB Subnet Group** button and use the following details:


![Figure 1](/images/rds1.png)

Scroll down and add the two Data subnets created earlier (one for each AZ) to your new subnet group:

![Figure 2](/images/rds2.png)

### Create the Aurora database cluster

Once the subnet group has been created, we can go ahead and create the Aurora cluster. To do that please navigate to your RDS dashboard, click on Create database and use the following details:

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

Once you have created the database cluster, click Modify to see the associated security group:

![Figure 9](/images/rds9.png)

Then browse to the VPC dashboard and modify the group to allow traffic from **Application Subnet A** and **Application Subnet B** on port 3306/tcp (MySQL). 

![Figure 10](/images/rds10.png)
 

---

Wordpress uses its database to house articles, users, and configuration information.  This can result in a lot of unnecessary queries against the database which consistently return the same data set.  To offset the strain on the database and improve performance of the Wordpress web application let's add a caching layer for common SQL requests.  In the next lab you will deploy a database caching layer for use by Wordpress.
