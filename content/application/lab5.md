---
title: "Lab 5: Autoscaling and load balancing"
weight: 300
---

### Create a launch configuration for the Auto Scaling Groups (ASG)

A launch configuration is an instance configuration template that an Auto Scaling Group uses to launch EC2 instances. When you create launch configurations you need to specify configuration information for the instances, including the ID of the Amazon Machine Image (AMI), the instance type, a key pair, one or more security groups, and a block device mapping. 

To get started select **Launch configurations** in your EC2 dashboard, then click on **Create launch configuration**. On the next screen select the second Amazon Linux AMI, as highlighted below:

![Figure 1](/images/asg1.png)

Select the instance type:

![Figure 2](/images/asg2.png)

Create your launch configuration:

![Figure 3](/images/asg3.png)

When reaching the **Configure security group** page, please select the default security group for the VPC you have created earlier, then click **Review** to review and submit the final configuration.

### Create the ASG for the back-end web servers

Once you have created the launch configuration you can proceed to creating the Autoscaling group for the Wordpress web servers. To do that select **Auto Scaling Groups** in your EC2 dashboard, click on **Create Auto Scaling Group** and select the previously created launch configuration:

![Figure 4](/images/asg4.png)

In the next screen make sure that the correct VPC is selected, together with the correct subnets for the web servers (i.e. Application Subnet A and Application Subnet B):

![Figure 5](/images/asg5.png)

Next, configure the scaling policies as follows:

![Figure 6](/images/asg6.png)

### Set up Elastic Load Balancing

A load balancer distributes incoming application traffic across multiple targets, such as EC2 instances, in multiple Availability Zones, increasing the availability of the Wordpress platform.

To create a new Elastic Load Balancer select **Load Balancers** in your EC2 dashboard, click on **Create Load Balancer** and follow the guidelines below:

![Figure 7](/images/asg7.png)

When configuring the load balancer make sure you choose the correct VPC, together with the correct public subnets from each Availability Zone:

![Figure 8](/images/asg8.png)

Then select the default security group associated with your VPC:

![Figure 9](/images/asg9.png)

Configure the target groups and routing policies:

![Figure 10](/images/asg10.png)

Now add the two instances created by the autoscaling group in the previous step to the ELB’s registered targets:

![Figure 11](/images/asg11.png)

Once you submit the final configuration you will be able to see the load balancer in your EC2 dashboard:

![Figure 12](/images/asg12.png)

As a last step, go back to the Autoscaling Groups, select **Wordpress-workshop-ASG**, then click on **Actions, Edit** and add the ELB target group created earlier (i.e. Wordpress-workshop) under the **Target Groups** setting:

![Figure 13](/images/asg13.png)







