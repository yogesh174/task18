# WordPress with Amazon RDS

## Intro

✒ WordPress is a free and open-source content management system written in PHP and paired with a MySQL or MariaDB database. It is used to build amazing looking websites in a short period of time due to its ease of use and some of its major features such as plugins and templates. But one must connect a database to WordPress for maintaining all the site's data.

✒ Amazon Relational Database Service (Amazon RDS) is a web service that makes it easier to set up, operate, and scale a relational database in the AWS Cloud. It provides cost-efficient, resizable capacity for an industry-standard relational database and manages common database administration tasks. It is a fully managed database service provided by Amazon.

✒ With Amazon RDS we can use different kinds of database products such as MySQL, MariaDB, PostgreSQL etc...

✒ In this blog, we will see how to integrate WordPress with Amazon RDS

## Outline

✔ Setting up a MySQL server using Amazon RDS

✔ Launch an EC2 Instance with Apache web server and WordPress setup

✔ Configure WordPress

## Implementation


### Setting up a MySQL server using Amazon RDS

✒ Configure a Security Group with the below Inbound rules (we will use this later for the database instance) 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611658565429/fHstL1RRMx.png)

✒ Go to the Databases tab under Amazon RDS service in the AWS console and click on create a database

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611493137823/lrEPybV6x.png)

✒ Choose the standard create:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611491631724/rhE1qpjGC.png)

✒ Choose a DB engine and a version (I'll be using MySQL - 5.7.31 as it is already tested for WordPress)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611491744898/yUUusXIxb.png)

✒ Choose the free tier template for testing.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611491799093/Fr3_z5A_I.png)

✒ In the settings assign a name for the database instance and credentials for the database

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611491885279/YzBbJO4_K.png)

 ✒ In the instance family type db.t2.micro is the only type which is available for the free tier.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611491989728/cyQxHCGrn.png)

✒ Set the storage of the instance/EBS, for testing purposes the following will be enough:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611492122503/r0T2s7YPb.png)

✒ Next comes the connectivity of Instance i.e., where do you want to launch the instance. Choose a VPC, Subnet, whether should it be publicly accessible or not, Security group(Attach the previously created Security Group) and finally AZ.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611492278513/r5dubigfr.png)

✒ In Database Options give the initial database name as "wordpress" and click on create a database which will then launch your database instance.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611492928535/E3JERvxWK.png)

✒ There are some other configurations as well such as backup and delete protection which I'm going to leave as it is. 

✒ It will take some time to launch the database instance

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611660096383/QN67mPcoR.png)

✒ After the RDS is ready, note the endpoint (we will use this endpoint while setting up WordPress)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611660511235/CgqlPpZxX.png)


### Launch an EC2 Instance with Apache web server and WordPress setup

✒ Go to the Instances tab under Amazon EC2 service in the AWS console and click on launch an Instance

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611661297017/3s9E82dSe.png)

✒ I will be going with the RHEL 8 AMI

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611661454972/JipLyk3pe.png)

✒ Now choose t2.micro instance (this will be sufficient for the demo)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611661555036/fZvN47HHg.png)

✒ Now add the following user data (Essentially a script which executes while the Instace boots up)

```
#!/usr/bin/bash

# Install the necessary s/w

dnf install php php-mysqlnd php-fpm httpd tar curl php-json -y

# Download and setup WordPress

curl https://wordpress.org/latest.tar.gz --output wordpress.tar.gz
tar xf wordpress.tar.gz
cp -r wordpress/* /var/www/html/

# Give necessary permissions

setenforce 0
chown -R apache:apache /var/www/html/wordpress
chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R

# Start and enable webserver

systemctl start httpd
systemctl enable httpd
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611662728238/LiQI1S4nz.png)

✒ Leave the default configurations for storage as it is and proceed further

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611662933143/iA1JxcXqN.png)

✒ Create a name tag and assign it a value(for management purposes)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611664618829/ahLw0qMLN.png)

✒ Create a Security Group with the following Inbound rules for webserver -

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611664855313/B_p4GzRfm0.png)

✒ Choose a KeyPair and launch the Instance

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611664960554/IEYJMGUv_.png)

✒ If you get something like this:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611681444370/vNEnGeGLB.png)

Then you are probably using https protocol or there is a problem in Security Group. To resolve the former issue all we need to do is to give the complete URL i.e., "http://<IP or domain>"

### Configure WordPress

✒ After this you will be redirected to:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611681347322/nwDp18ZsK.png)

Click on Let's go

✒ Then give your database credentials which you have assigned while creating database using Amazon RDS service:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611683191369/WYnWzr3pH.png)

✒ Click on Run Installation:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611683197583/LkoxBQuqs.png)

✒ Then set the credentials for the WordPress website and click on Install WordPress:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611683507520/YjpufIiF9.png)

✒ Then login using the credentials that you've set in the previous step:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611683547077/UynMYDzRK.png)

✒ Customize the website according to your requirements:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611683589820/04rUhKkq3.png)

✒ Viola we now have our WordPress site up and running with RDS database service!

![temp.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1611725204826/nyjdKtIu5.png)

Thank You!
