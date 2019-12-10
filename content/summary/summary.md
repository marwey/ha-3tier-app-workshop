---
title: "Summary"
weight: 90
---

During this lab you used AWS VPC to create a software-defined network across multiple AWS availability zones in a single AWS region.  

Using Amazon RDS you created a managed active / standby multi-node MySQL database deployment that it automatically backed up and recoverable to within the last 5 minutes and is patched on your behalf.  In the event of a failure the database will automatically failover to the standby node which is synchronously replicated with the active node, ensuring you never lose your data.

You also created a Memcached-powered cache using Amazon ElastiCache.  This allows you to cache frequent database queries and reduce the number of queries you have to make against your HA MySQL database.

Then you created a distributed NFS share using Amazon Elastic Filesystem (EFS) to share a single Wordpress installation across multiple application servers.

Finally, using AWS auto scaling and the AWS Application Load Balancer you created a collection of virtual machines that will automatically scale out and back in based upon resource utilization and in response to the number of users hitting your application server.  As new servers come online the load balancer will automatically be updated with their information, as servers are removed the auto scaling group will inform the load balancer and drain connections from those servers.

One step that is missing is we have not yet configured Wordpress to use the Memcached deployment we created.  To do this you can follow the [Wordpress / Memcached instructions](https://aws.amazon.com/elasticache/memcached/wordpress-with-memcached/) on the AWS website.  The necessary plugins and libraries are installed by the user data earlier in Lab 5, however you need to manually configure them through the Wordpress Web UI to use your Memcached deployment.

All of this produces a highly-available, distributed, and fault tolerate web application.  The pattern demonstrated in this lab can be re-used for many other, common, web application technologies.  Anywhere where state can be externalized to a filesystem, cache, or database.

We hope that you have enjoyed this lab and will continue to learn to build resilient, fault tolerant systems on AWS!