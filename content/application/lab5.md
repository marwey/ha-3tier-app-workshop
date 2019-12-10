---
title: "Lab 5: Autoscaling and load balancing"
weight: 300
---

You have created a software-defined network across multiple fault-isolated data centers, deployed an HA active / passive MySQL database, a managed Memcached instance, and a distributed NFS cluster for shared storage.  In this last and final lab you will deploy an HA application server running PHP to use these resources as part of a scalable Wordpress installation.  

---

### Create an IAM role

EC2 instances can be granted access to AWS APIs using temporary credentials granted through an [Instance Profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html).  An Instance Profile is an IAM role which can be assigned to an EC2 instance.  In the event that you wish to have shell-level access to the Amazon Linux operating system of your Wordpress application servers you can allow for SSH access to the instances or you can use [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html).  

Session Manager provides shell access via the AWS console and is easily configured when using Amazon Linux.  It has the added benefit of sending all keystrokes to CloudWatch logs so that you have an audit trail of activity on your instances.  To use Session Manager we will create an IAM role and reference it later when we create the Launch Configuration for our Wordpress application servers.

Access the [AWS IAM console](https://console.aws.amazon.com/iam/home) and click **Roles** on the left-hand side.  Then click **Create role** to begin creating a role for your Wordpress application servers.  

Select `AWS Service` -> `EC2` as the type of Trusted Entity and click **Next: Permissions**.  On the next screen search for the managed policy `AmazonSSMManagedInstanceCore`, tick the checkbox next to the policy and click **Next: Tags**.  Click **Next: Review**, give your role a name such as `Wordpress-Application-Role` and click **Create role**.

Later, when your EC2 instances are running with this role attached, if you need shell-level access to any of them you can make a note of the Instance ID and from the [Session Manager console](https://console.aws.amazon.com/systems-manager/session-manager/sessions) start a web-based shell session.

### Create load balancer and application security groups

Visit the [AWS VPC console](https://console.aws.amazon.com/vpc/home) and create 2 security groups.  The first security group should be named something like *WP Load Balancer SG* and the second security group should be named *WP Wordpress SG*.  

Edit the **Inbound Rules** for **WP Load Balancer SG** and allow HTTP traffic on port 80 from `0.0.0.0/0` (indicating from anywhere).

Edit the **Inbound Rules** for **WP Wordpress SG** and only allow HTTP traffic on port 80 from the **WP Load Balancer SG**.

### Create a load balancer

A load balancer distributes incoming application traffic across multiple targets, such as EC2 instances, in multiple Availability Zones, increasing the availability of the Wordpress platform.

From the EC2 console click **Load Balancers** on the left-hand menu and then click **Create Load Balancer**.  Under *Application Load Balancer* click **Create**.  

Give your load balancer a name and under **Availability Zones** select your VPC.  Then tick the checkbox for both availability zones and select your public subnets as created in the first lab.  

![Figure 8](/images/asg8.png)

Click **Next: Configure Security Groups**.

Select the **WP Load Balancer SG** created earlier and click **Next: Configure Routing**.

Give the target group a name and click **Next: Register Targets**.  Click **Next: Review** without defining any targets and then click **Create**.

![Figure 12](/images/asg12.png)

Make a note of the **DNS name** created for your load balancer as you will need this in the following steps.

### Create a launch configuration for the Auto Scaling Groups (ASG)

A launch configuration is an instance configuration template that an Auto Scaling Group uses to launch EC2 instances. When you create launch configurations you need to specify configuration information for the instances, including the ID of the Amazon Machine Image (AMI), the instance type, a key pair, one or more security groups, and a block device mapping. 

Select **Launch configurations** in your EC2 dashboard, then click on **Create launch configuration**. Choose the **Amazon Linux AMI** by clicking the corresponding **Select** button for the image.

Select the instance type:

![Figure 2](/images/asg2.png)

Create your launch configuration.  Specify the IAM role you created earlier so that you can access the EC2 instances created using Session Manager.

![Figure 3](/images/asg3.png)

Paste the script below in the User Data field.  Be sure and set the following values accordingly:

 - EFS_MOUNT
 - DB_NAME
 - DB_HOSTNAME
 - DB_USERNAME
 - DB_PASSWORD
 - WP_ADMIN
 - WP_PASSWORD
 - LB_HOSTNAME

```bash
#!/bin/bash -xe

EFS_MOUNT="<YOUR-EFS-HOSTNAME>"

DB_NAME="wordpress"
DB_HOSTNAME="<YOUR-DB-HOSTNAME>"
DB_USERNAME="<YOUR-DB-USERNAME>"
DB_PASSWORD="<YOUR-DB-PASSWORD>"

WP_ADMIN="wpadmin"
WP_PASSWORD="WpPassword$"

LB_HOSTNAME="<YOUR-ALB-HOSTNAME>"

yum update -y
yum install -y awslogs httpd24 mysql56 php55 php55-devel php55-pear php55-mysqlnd gcc-c++ php55-opcache

mkdir -p /var/www/wordpress
mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 $EFS_MOUNT:/ /var/www/wordpress

## create site config
cat <<EOF >/etc/httpd/conf.d/wordpress.conf
ServerName 127.0.0.1:80
DocumentRoot /var/www/wordpress/wordpress
<Directory /var/www/wordpress/wordpress>
  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted
</Directory>
EOF

## install cache client
pecl install igbinary-2.0.8
wget -P /tmp/ https://s3.amazonaws.com/aws-refarch/wordpress/latest/bits/AmazonElastiCacheClusterClient-1.0.1-PHP55-64bit.tgz
tar -xf '/tmp/AmazonElastiCacheClusterClient-1.0.1-PHP55-64bit.tgz'
cp 'AmazonElastiCacheClusterClient-1.0.0/amazon-elasticache-cluster-client.so' /usr/lib64/php/5.5/modules/
if [ ! -f /etc/php-5.5.d/50-memcached.ini ]; then
    touch /etc/php-5.5.d/50-memcached.ini
fi
echo 'extension=igbinary.so;' >> /etc/php-5.5.d/50-memcached.ini
echo 'extension=/usr/lib64/php/5.5/modules/amazon-elasticache-cluster-client.so;' >> /etc/php-5.5.d/50-memcached.ini

## install wordpress
if [ ! -f /bin/wp/wp-cli.phar ]; then
   curl -o /bin/wp https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
   chmod +x /bin/wp
fi
# make site directory
if [ ! -d /var/www/wordpress/wordpress ]; then
   mkdir -p /var/www/wordpress/wordpress
   cd /var/www/wordpress/wordpress
   # install wordpress if not installed
   # use public alb host name if wp domain name was empty
   if ! $(wp core is-installed --allow-root); then
       wp core download --version='4.9' --locale='en_GB' --allow-root
       wp core config --dbname="$DB_NAME" --dbuser="$DB_USERNAME" --dbpass="$DB_PASSWORD" --dbhost="$DB_HOSTNAME" --dbprefix=wp_ --allow-root
       wp db create --allow-root
       wp core install --url="http://$LB_HOSTNAME" --title='Wordpress on AWS' --admin_user="$WP_ADMIN" --admin_password="$WP_PASSWORD" --admin_email='admin@example.com' --allow-root
       wp plugin install w3-total-cache --allow-root
       # sed -i \"/$table_prefix = 'wp_';/ a \\define('WP_HOME', 'http://' . \\$_SERVER['HTTP_HOST']); \" /var/www/wordpress/wordpress/wp-config.php
       # sed -i \"/$table_prefix = 'wp_';/ a \\define('WP_SITEURL', 'http://' . \\$_SERVER['HTTP_HOST']); \" /var/www/wordpress/wordpress/wp-config.php
       # enable HTTPS in wp-config.php if ACM Public SSL Certificate parameter was not empty
       # sed -i \"/$table_prefix = 'wp_';/ a \\# No ACM Public SSL Certificate \" /var/www/wordpress/wordpress/wp-config.php
       # set permissions of wordpress site directories
       chown -R apache:apache /var/www/wordpress/wordpress
       chmod u+wrx /var/www/wordpress/wordpress/wp-content/*
       if [ ! -f /var/www/wordpress/wordpress/opcache-instanceid.php ]; then
         wget -P /var/www/wordpress/wordpress/ https://s3.amazonaws.com/aws-refarch/wordpress/latest/bits/opcache-instanceid.php
       fi
   fi
   RESULT=$?
   if [ $RESULT -eq 0 ]; then
       touch /var/www/wordpress/wordpress/wordpress.initialized
   else
       touch /var/www/wordpress/wordpress/wordpress.failed
   fi
fi

## install opcache
if [ ! -d /var/www/.opcache ]; then
    mkdir -p /var/www/.opcache
fi
# enable opcache in /etc/php-5.5.d/opcache.ini
sed -i 's/;opcache.file_cache=.*/opcache.file_cache=\/var\/www\/.opcache/' /etc/php-5.5.d/opcache.ini
sed -i 's/opcache.memory_consumption=.*/opcache.memory_consumption=512/' /etc/php-5.5.d/opcache.ini
# download opcache-instance.php to verify opcache status
if [ ! -f /var/www/wordpress/wordpress/opcache-instanceid.php ]; then
    wget -P /var/www/wordpress/wordpress/ https://s3.amazonaws.com/aws-refarch/wordpress/latest/bits/opcache-instanceid.php
fi

chkconfig httpd on
service httpd start
```

Then click **Next: Configure Security Groups** and select the following security groups:

 - WP Cache Client SG
 - WP DB Client SG
 - WP EFS Client SG
 - WP Wordpress SG

Click **Review** to review and submit the final configuration.  You can disregard warnings about being able to SSH into the server and can also choose *Proceed without keypair* as you will not need to remotely access these servers.

### Create the ASG for the back-end web servers

Once you have created the launch configuration you can proceed to creating the Autoscaling group for the Wordpress web servers. To do that select **Auto Scaling Groups** in your EC2 dashboard, click on **Create Auto Scaling Group** and select the previously created launch configuration:

![Figure 4](/images/asg4.png)

In the next screen make sure that the correct VPC is selected, together with the correct subnets for the web servers (i.e. Application Subnet A and Application Subnet B):

![Figure 5](/images/asg5.png)

Under **Advanced Details** tick the box next to **Load Balancing** and for **Target Groups** select the target group you created earlier.

Click **Next: Configure scaling policies** and configure the scaling policies as follows:

![Figure 6](/images/asg6.png)

Click through and accept the remaining defaults to complete the creation of your auto scaling group.

The autoscaling group will now begin creating the desired number of EC2 instances based on the launch configuration you created.  As the systems come online the target group is updated with the instance details for your EC2 instances and the load balancer will begin distributing traffic across the instances.  As instances are added or removed the autoscaling group and load balancer will work in concert with one another to ensure that only healthy instances receive traffic.

![Healthy Targets](/images/healthy_targets.png)

When your targets are deemed healthy in your target group you can open the DNS name for your Application Load Balancer in your web browser to view your newly created Wordpress installation.

![Wordpress Screenshot](/images/wp_screenshot.png)

---

You have now created a highly-available auto-scaling deployment of Wordpress that will scale in and out in response to client traffic hitting the website.  Lets now review, end to end, what you have accomplished.
