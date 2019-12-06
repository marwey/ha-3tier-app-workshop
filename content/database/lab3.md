---
title: "Lab 3: Set up Elasticache for Memcached"
weight: 300
---

Memcached is a server-side caching mechanism that reduces the database load by caching common database queries, leading to a faster loading Wordpress website.

To get started with Elasticache, open its dashboard by searching for the service in the AWS console, then click on **Memcached** and finally click on **Create** to spin up your first cluster:

![Figure 1](/images/ec1.png)

Then click on **Advanced Memcached settings** and make sure you create a new subnet group consisting of the Data subnets of both availability zones:

![Figure 2](/images/ec2.png)

Finally, click on **Create** to start building your cluster. Once the cluster has been built, click on it to expand it and write down the **Configuration Endpoint** parameter as you will need it a few steps later:

![Figure 3](/images/ec3.png)

---

You have now created a resilient active / standby database deployment with a caching layer that is distributed across multiple data centers.  In the next set of labs you will build the application tier which will deploy Wordpress to begin using the data tier and serving your customers.
