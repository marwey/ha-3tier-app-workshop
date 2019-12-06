---
title: "Lab 4: Create a shared filesystem"
weight: 200
---

Amazon Elastic Filesystem (EFS) provides you with a managed NFS cluster which is compatible with NFS v4.1.  In this lab you will create an EFS cluster that will provide a shared filesystem for your web application servers.

## Create filesystem security groups

Visit the [AWS VPC console](https://console.aws.amazon.com/vpc/home) and create 2 security groups.  The first security group should be named something like *WP FS Client SG* and the second security group should be named *WP FS SG*.  

With both groups created edit the **Inbound Rules** of the *WP FS SG* security group and add a rule of type `NFS` which allows traffic on port `2049` from the *WP FS Client SG*.

Now you are ready to create your EFS cluster.

## Create the EFS cluster

To create an EFS cluster visit the [Amazon EFS console](https://console.aws.amazon.com/efs/home) and click **Create file system**.

From the VPC drop down select your VPC and then choose the two subnets you created for the application tier.  You are now defining mount targets in each of your availability zones.  For each mount target, on the right under *Security groups*, associate the *WP FS SG* security group created above to each mount target and remove the association with the *default* security group. 

Click *Next Step*.

On the next page review and accept the defaults by clicking **Next Step**.

To complete the setup of your EFS cluster click **Create File System**.

This will create two mount targets in your subnets and after a few minutes the targets will reach a state of *Available*. 

---

You now have an EFS cluster ready and available for your use.  In the next lab you will create the application servers that will connect to and use these resources to serve your clients.
