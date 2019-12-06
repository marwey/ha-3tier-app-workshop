---
title: "Lab 1: Configure the network"
weight: 400
---

### Create a new Virtual Private Cloud (VPC)

As a starting point for the workshop you will need to login to your AWS account, select the region of your choice and create a new VPC.

To do this click on **Your VPCs** on the left hand side of the console and click **Create VPC**.  Enter a name for your VPC and a CIDR range such as the one below.  When you're fiinished click **Create**.

![Figure 1 - Create a new VPC](/images/figure1.png)


### Create public and private subnets in the new VPC

Once the VPC has been created, the next step is to create the subnets that will be used to
host the application across two different Availability Zones. We are going to create six
subnets in total, three for each AZ, as shown in the following diagram:

![Figure 2](/images/figure2.png)

To create each of the six subnets please navigate to the VPC dashboard of your account,
select **Subnets** , then click on **Create subnet** and use the details below. Make sure to always
select the **Wordpress-workshop** VPC when creating the subnets.

![Figure 3](/images/figure3.png)

{{% notice info %}}
The screenshots below were taken in Ireland (eu-west-1), if you are building in a different AWS region please just ensure that you create your subnets in 2 different availability zones in the same region, such as **us-west-2a** and **us-west-2b**.
{{% /notice %}}

For each subnet specify a name and a CIDR range for the subnet.  Be sure and create a public, application, and data subnet in each of two availability zones as detailed in the table below.

![Table 1](/images/table1.png)

At this point all the correct subnets have been created and they can route network traffic between them.  In the next set of steps you will create an Internet Gateway, allowing communication between your VPC and the Internet.  You will also configure your routing tables to only allow internet communication with your public subnets and not the private application or data subnets.

### Create an Internet Gateway and set up routing

The following steps will allow connectivity from the Internet to the public subnets and also connectivity from the private subnets to the Internet via NAT gateways.

First you need to create a new Internet Gateway (IGW) from your VPC dashboard and attach it to the **Wordpress-workshop** VPC.  Start by clicking **Internet Gateways** on the left hand side of the VPC console and then click the **Create internet gateway** button.  Enter a name for your IGW such as `WP Internet Gateway` and click **Create**.  After the IGW has been created you need to associate it with your VPC by attaching it to your VPC:

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

### Create NAT gateways across the public subnets

The Wordpress instances will need to be able to connect to the Internet and download application or OS updates so we’re going to create two NAT gateways, one for each availability zone where the application is deployed.

To do this you will create a NAT Gateway for each availability zone, then create a routing table for the application subnet in each availability zone, update the routing table with a path to the NAT gateway, and then associate it with the application subnet.

![Figure 9](/images/figure9.png)

Go to the VPC dashboard in your account, select **NAT Gateways** and create one gateway in 
each of the two public subnets (i.e. Public Subnet A and Public Subnet B) Always make sure
you have selected the correct public subnet when creating the gateway.

![Figure 10](/images/figure10.png)

Now we need to create route tables for each of the two Application subnets and use the NAT gateways created earlier as the default gateway:

![Figure 16](/images/figure16.png)

Edit the route table and add the default route via the NAT gateway in Application subnet A:

![Figure 17](/images/figure17.png)

Associate the route table with Application Subnet A:

![Figure 18](/images/figure18.png)

Repeat the last three steps to also create a route table for Application Subnet B which uses the NAT gateway deployed in the second availability zone.

---

You have now created a virtual private cloud network across 2 availability zones within an AWS region.  You have created 6 subnets, 3 in each availability zone, and have configured a route so that the internet can communicate with resources in the public subnets and vice versa.  The application subnets have been configured, via routing table, to communicate with the internet via NAT gateways in the public subnets, and the data subnets can only communicate with resources in the 6 subnets, but not the internet.  You are now ready to start creating the database cluster.
