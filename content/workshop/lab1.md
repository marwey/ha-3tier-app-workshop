---
title: "Lab 1: Create your Webserver"
weight: 10
---


## Introduction

The intent of this workshop is to provide a set of best practices for building highly resilient
Wordpress environments on Amazon Web Services (AWS)

## Configure the network environment

### Create a new Virtual Private Cloud (VPC)

As a starting point for the workshop you will need to login to your AWS account, select the
region of your choice and create a new VPC using the following settings:

```
Figure 1 - Create a new VPC
```
### Create public and private subnets in the new VPC..........................................................................

Once the VPC has been created, the next step is to create the subnets that will be used to
host the application across two different Availability Zones. We are going to create six
subnets in total, three for each AZ, as shown in the following diagram:

```
Figure 2 – Network environment
```

To create each of the six subnets please navigate to the VPC dashboard of your account,
select **Subnets** , then click on **Create subnet** and use the details below. Make sure to always
select the **Wordpress-workshop** VPC when creating the subnets.

```
Subnet Name Subnet CIDR Subnet location
Public Subnet A 192.168.0.0/24 eu-west-1a
Public Subnet B 192.168.1.0/24 eu-west-1b
Application Subnet A 192.168.2.0/24 eu-west-1a
Application Subnet B 192.168.3.0/24 eu-west-1b
Data Subnet B 192.168. 4 .0/24 eu-west-1a
Data Subnet B 192.168. 5 .0/24 eu-west-1b
```
_Figure 3 - Create the first public subnet in eu-west-1a_


_Figure 4 - Create the second public subnet in eu-west-1b_

_Figure 5 - Create the first application subnet in eu-west-1a_


_Figure 6 - Create the second application subnet in eu-west-1b_

_Figure 7 - Create the first data subnet in eu-west-1a_


_Figure 8 - Create the second data subnet in eu-west-1b_

At this point all the correct subnets have been created so we can proceed with the routing
and NAT configuration.

### Create NAT gateways across the public subnets

The Wordpress instances will need to be able to connect to the Internet and download
application or OS updates so we’re going to create two NAT gateways, one for each
availability zone where the application is deployed.

```
Figure 9 - NAT Gateways
```

Go to the VPC dashboard in your account, select **NAT Gateways** and create one gateway for
each of the two public subnets (i.e. Public Subnet A and Public Subnet B) Always make sure
you have selected the correct public subnet when creating the gateway.

_Figure 10 - Create a NAT gateway_

### Create an Internet Gateway and set up routing

The following steps will enable routing between the various subnets created earlier,
together with connectivity from the Internet to the public subnets and also connectivity
from the private subnets to the Internet.

First you need to create a new Internet Gateway from your VPC dashboard and attach it to
the **Wordpress-workshop** VPC:

```
Figure 11 - Create an Internet Gateway
```
```
Figure 12 – Attach the gateway to your VPC
```

The gateway will be used by instances and services hosted in the public subnets (e.g. Public
Subnet A and Public Subnet B) to communicate over the Internet.

Once the gateway is created you will need to create a new routing table and associate it with
the public subnets:

_Figure 13 - Create the routing table for public instances_

After creating the routing table select it from the **Route Tables** section of your VPC
dashboard, then click on **Edit routes** and add a default route via the Internet Gateway
created in the previous step:

_Figure 14 - Point your default route to the Internet Gateway_

Finally, you need to associate the newly created routing table with the public subnets. To do
that, click on the respective route table, then click on **Subnet Associations** and select the
two public subnets created earlier:


_Figure 15 - Associate the public routing table with the public subnets_

The process must be repeated for the private subnets as well, the main difference being that
routing to the Internet will be provided in this case by the NAT gateways instead of the
Internet Gateways:

_Figure 16 - Create a new route table for the Application subnets_

_Figure 17 - Edit the route table and add the default route via the NAT gateway_


_Figure 18 - Associate the route table with the Application subnets_

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

_Figure 19 – Select the AMI_


_Figure 20 - Select the instance type_

_Figure 21 - Configure details_

When reaching the **Configure security group** page, please select the default security group
for the VPC you have created earlier, then click **Review** to review and submit the final
configuration.


### Create the ASG for the back-end web servers

Once you have created the launch configuration you can proceed to creating the Autoscaling
group for the Wordpress web servers. To do that select **Auto Scaling Groups** in your EC
dashboard, click on **Create Auto Scaling Group** and select the previously created launch
configuration:

_Figure 22 - Select the Wordpress-workshop launch configuration_

In the next screen make sure that the correct VPC is selected, together with the correct
subnets for the web servers (i.e. Application Subnet A and Application Subnet B):


_Figure 23 - Configure the ASG_

Next, configure the scaling policies as follows:

_Figure 24 - Configure the scaling policies_

### Set up Elastic Load Balancing

A load balancer distributes incoming application traffic across multiple targets, such as EC
instances, in multiple Availability Zones, increasing the availability of the Wordpress
platform.

To create a new Elastic Load Balancer select **Load Balancers** in your EC2 dashboard, click on
**Create Load Balancer** and follow the guidelines below:


_Figure 25 - Select Application Load Balancer_

When configuring the load balancer make sure you choose the correct VPC, together with
the correct public subnets from each Availability Zone:

_Figure 26 - Configure the load balancer_

Then select the default security group associated with your VPC:


_Figure 27 - Select the security group_

Configure the target groups and routing policies:

_Figure 28 – Create a new target group_

Now add the two instances created by the autoscaling group in the previous step to the
ELB’s registered targets:


_Figure 29 - Register your instances as targets_

Once you submit the final configuration you will be able to see the load balancer in your EC
dashboard:

_Figure 30 - The Load Balancer has been successfully created_

As a last step, go back to the Autoscaling Groups, select **Wordpress-workshop-ASG** , then
click on **Actions** , **Edit** and add the ELB target group created earlier (i.e. Wordpress-
workshop) under the **Target Groups** setting:


_Figure 31 - Add the correct ELB target group_

### Install and configure Wordpress...................................................................................................

## Set up the Elastic File System (EFS) back-end

### [...]

### [...]

## Secure the environment

### [...]



