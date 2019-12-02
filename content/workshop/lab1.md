---
title: "Lab 1: Setup your environment"
weight: 10
---


## Introduction

The intent of this workshop is to provide a set of best practices for building highly resilient
Wordpress environments on Amazon Web Services (AWS)

## Configure the network environment

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

## Configure the application back-end

### Create a launch configuration for the Auto Scaling Groups (ASG)

A launch configuration is an instance configuration template that an Auto Scaling Group uses
to launch EC2 instances. When you create launch configurations you need to specify
configuration information for the instances, including the ID of the Amazon Machine Image
(AMI), the instance type, a key pair, one or more security groups, and a block device
mapping.

To get started select **Launch configurations** in your EC2 dashboard, then click on **Create
launch configuration**. On the next screen select the second Amazon Linux AMI, as
highlighted below:

Select the AMI:

![Figure 19](/images/figure19.png)

Select the instance type:

![Figure 20](/images/figure20.png)

Configure details:

![Figure 21](/images/figure21.png)

When reaching the **Configure security group** page, please select the default security group
for the VPC you have created earlier, then click **Review** to review and submit the final
configuration.


### Create the ASG for the back-end web servers

Once you have created the launch configuration you can proceed to creating the Autoscaling
group for the Wordpress web servers. To do that select **Auto Scaling Groups** in your EC
dashboard, click on **Create Auto Scaling Group** and select the previously created launch
configuration:

Select the Wordpress-workshop launch configuration:

![Figure 22](/images/figure22.png)

In the next screen make sure that the correct VPC is selected, together with the correct
subnets for the web servers (i.e. Application Subnet A and Application Subnet B):

![Figure 23](/images/figure23.png)

Next, configure the scaling policies as follows:

![Figure 24](/images/figure24.png)

### Set up Elastic Load Balancing

A load balancer distributes incoming application traffic across multiple targets, such as EC
instances, in multiple Availability Zones, increasing the availability of the Wordpress
platform.

To create a new Elastic Load Balancer select **Load Balancers** in your EC2 dashboard, click on
**Create Load Balancer** and follow the guidelines below:

Select Application Load Balancer:

![Figure 25](/images/figure25.png)

When configuring the load balancer make sure you choose the correct VPC, together with
the correct public subnets from each Availability Zone:

![Figure 26](/images/figure26.png)

Then select the default security group associated with your VPC:

![Figure 27](/images/figure27.png)

Configure the target groups and routing policies:

![Figure 28](/images/figure28.png)

Now add the two instances created by the autoscaling group in the previous step to the
ELB’s registered targets:

![Figure 29](/images/figure29.png)

Once you submit the final configuration you will be able to see the load balancer in your EC
dashboard:

![Figure 30](/images/figure30.png)

As a last step, go back to the Autoscaling Groups, select **Wordpress-workshop-ASG** , then
click on **Actions** , **Edit** and add the ELB target group created earlier (i.e. Wordpress-
workshop) under the **Target Groups** setting:

![Figure 31](/images/figure31.png)


### Install and configure Wordpress

## Set up the Elastic File System (EFS) back-end

### [...]

### [...]

## Secure the environment

### [...]



