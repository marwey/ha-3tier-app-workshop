---
title: "Lab 1: Configure the network"
weight: 10
---

### Create a new Virtual Private Cloud (VPC)

As a starting point for the workshop you will need to login to your AWS account, select the
region of your choice and create a new VPC using the following settings:

![Figure 1 - Create a new VPC](/images/figure1.png)

### Create public and private subnets in the new VPC

Once the VPC has been created, the next step is to create the subnets that will be used to
host the application across two different Availability Zones. We are going to create six
subnets in total, three for each AZ, as shown in the following diagram:

![Figure 2](/images/figure2.png)

To create each of the six subnets please navigate to the VPC dashboard of your account,
select **Subnets** , then click on **Create subnet** and use the details below. Make sure to always
select the **Wordpress-workshop** VPC when creating the subnets.

![Table 1](/images/table1.png)

Create the first public subnet in eu-west-1a:

![Figure 3](/images/figure3.png)

Create the second public subnet in eu-west-1b:

![Figure 4](/images/figure4.png)

Create the first application subnet in eu-west-1a:

![Figure 5](/images/figure5.png)

Create the second application subnet in eu-west-1b:

![Figure 6](/images/figure6.png)

Create the first data subnet in eu-west-1a:

![Figure 7](/images/figure7.png)

Create the second data subnet in eu-west-1b:

![Figure 8](/images/figure8.png)

At this point all the correct subnets have been created so we can proceed with the routing
and NAT configuration.

### Create NAT gateways across the public subnets

The Wordpress instances will need to be able to connect to the Internet and download
application or OS updates so we’re going to create two NAT gateways, one for each
availability zone where the application is deployed.

![Figure 9](/images/figure9.png)

Go to the VPC dashboard in your account, select **NAT Gateways** and create one gateway for
each of the two public subnets (i.e. Public Subnet A and Public Subnet B) Always make sure
you have selected the correct public subnet when creating the gateway.

![Figure 10](/images/figure10.png)

### Create an Internet Gateway and set up routing

The following steps will enable routing between the various subnets created earlier,
together with connectivity from the Internet to the public subnets and also connectivity
from the private subnets to the Internet.

First you need to create a new Internet Gateway from your VPC dashboard and attach it to
the **Wordpress-workshop** VPC:

![Figure 11](/images/figure11.png)

Attach the gateway to your VPC:

![Figure 12](/images/figure12.png)

The gateway will be used by instances and services hosted in the public subnets (e.g. Public
Subnet A and Public Subnet B) to communicate over the Internet.

Once the gateway is created you will need to create a new routing table and associate it with
the public subnets:

![Figure 13](/images/figure13.png)

After creating the routing table select it from the **Route Tables** section of your VPC
dashboard, then click on **Edit routes** and add a default route via the Internet Gateway
created in the previous step:

![Figure 14](/images/figure14.png)

Finally, you need to associate the newly created routing table with the public subnets. To do
that, click on the respective route table, then click on **Subnet Associations** and select the
two public subnets created earlier:

![Figure 15](/images/figure15.png)

The process must be repeated for the private subnets as well, the main difference being that
routing to the Internet will be provided in this case by the NAT gateways instead of the
Internet Gateways:

![Figure 16](/images/figure16.png)

Edit the route table and add the default route via the NAT gateway:

![Figure 17](/images/figure17.png)

Associate the route table with the Application subnets:

![Figure 18](/images/figure18.png)

At this point we have finished building the network environment so we’re ready to start
creating the web servers and autoscaling groups.

