---
title: "Lab 3: Set up Elasticache for Memcached"
weight: 300
---

Memcached is a server-side caching mechanism that reduces the database load by caching common database queries, leading to a faster loading Wordpress website.

In this lab you will create a managed deployment of Memcached.  To start you will need to create a client and a server security group to protect your Memcached instances.  

## Create cache security groups

Visit the [AWS VPC console](https://console.aws.amazon.com/vpc/home) and create 2 security groups.  The first security group should be named something like *WP Cache Client SG* and the second security group should be named *WP Cache SG*.  

With both groups created edit the **Inbound Rules** of the *WP Cache SG* security group and allow traffic on port `11211` from the *WP Cache Client SG*.

Now you are ready to create your Memcached instance.

## Create an ElastiCache Memcached instance

To create an ElastiCache instance open the [Amazon ElastiCache console](https://console.aws.amazon.com/elasticache/home) and click on **Memcached**, and click on **Create** to spin up your first cluster. Ensure that **Memcached** is selected and you have given your cache a name.

![Figure 1](/images/ec1.png)

Then click on **Advanced Memcached settings** and make sure you create a new subnet group consisting of the Data subnets of both availability zones:

![Figure 2](/images/ec2.png)

{{% notice info %}}
Edit the security group and select the security group you created earlier for your cache instance.  Deselect the default security group, leaving only your *WP Cache SG* selected.
{{% /notice %}}

Finally, click on **Create** to start building your cluster. Once the cluster has been built, click on it to expand it and write down the **Configuration Endpoint** parameter as you will need it a few steps later:

![Figure 3](/images/ec3.png)

After a few minutes your Memcached instance will show with a status of *Available*.

---

You have now created a resilient active / standby database deployment with a caching layer that is distributed across multiple data centers.  In the next set of labs you will build the application tier which will deploy Wordpress to begin using the data tier and serving your customers.
