---
title: "Lab 5: Autoscaling and load balancing"
weight: 300
---

### Create a launch configuration for the Auto Scaling Groups (ASG)

A launch configuration is an instance configuration template that an Auto Scaling Group uses to launch EC2 instances. When you create launch configurations you need to specify configuration information for the instances, including the ID of the Amazon Machine Image (AMI), the instance type, a key pair, one or more security groups, and a block device mapping. 

To get started select **Launch configurations** in your EC2 dashboard, then click on **Create launch configuration**. On the next screen click on the **AWS Marketplace** tab, search for **Wordpress** and select the AMI highlighted below:

![Figure 1](/images/asg1.png)

Select the instance type:

![Figure 2](/images/asg2.png)

Create your launch configuration:

![Figure 3](/images/asg3.png)

Paste the script below in the User Data field, while replacing the parameters accordingly:
```
#!/bin/bash
cd /opt/bitnami/apps/wordpress/htdocs
rm wp-config.php
sudo -u bitnami -i -- wp core config --dbname=workshop --dbuser=wpadmin --dbpass=<Your RDS Master password> --dbhost=<The endpoint of your DB cluster or writer node>
sudo -u bitnami -i -- wp db create
sudo -u bitnami -i -- wp core install --url=`curl http://169.254.169.254/latest/meta-data/public-ipv4` --title=Wordpress-Workshop --admin_user=wpadmin --admin_password=AlwaysCh00seAStrongPassw0rd --admin_email=admin@example.com
```

{{% notice info %}}
What follows is potential alternate user data based on reference architecture to be used with Amazon Linux (NOT Amazon Linux 2)
{{% /notice %}}
```bash
#!/bin/bash -xe

EFS_MOUNT="fs-60d43c38.efs.eu-central-1.amazonaws.com"

DB_NAME="wordpress"
DB_HOSTNAME="wpdbv2-instance-1.cdq1du7ewx3u.eu-central-1.rds.amazonaws.com"
DB_USERNAME="dbadmin"
DB_PASSWORD="DbPassword$"

WP_ADMIN="wpadmin"
WP_PASSWORD="WpPassword$"

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
       wp core install --url='http://www.example.com' --title='Wordpress on AWS' --admin_user="$WP_ADMIN" --admin_password="$WP_PASSWORD" --admin_email='admin@example.com' --allow-root
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

To create a new Application Load Balancer select **Load Balancers** in your EC2 dashboard, click on **Create Load Balancer** and follow the guidelines below:

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







