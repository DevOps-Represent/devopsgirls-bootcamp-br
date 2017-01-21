# Load Balancing and High Availability.

## Key Concepts

Before we get started, here's a couple of loose concepts:

 - Load Balancers forward your traffic to one or more service providers. What this essentially means is that you can have multiple EC2 instances serving the same kind of content, being represented as one service.

 - High Availability means making it so that your system remains operational for a desirably long length of time. This means making it more durable to system failures - having failovers, in this case.

 - Service Separation refers to making sure that if you have multiple systems dependent on each other, that they exist separately and share fewer failure modes.

 - Artifacts are tangible byproducts of our systems development. In this case, our artifact will be the modified Wordpress package that's special to only you.

Now that that's out of the way, let's get started!

## What we're going to do

In this practical session, we will:

 - Create an Elastic Load Balancer
 - Reinstall Wordpress, but using an external database
 - Create an Artifact of your modified Wordpress, then upload it to S3
 - Create another instance, using the artifact as a base


## Creating an Elastic Load Balancer

### 1.) Create your load balancer

Go to Services > EC2, then look for "Load Balancers". Click on "Create Load Balancer"

### 2.) Select the type of load balancer

In the next page, make sure you select *Classic Load Balancer*.

### 3.) Name your ELB

Put *myname*-elb under *Load Balancer Name*. Under *Create ELB Inside*, select *My Default VPC*.

### 4.) Set your load balancing protocol

We'll leave the listener configuration intact for the time being - but do click on "Load Balancer Protocol". You should see a couple of options - load balancers are powerful!

### 5.) Create security groups for your ELB

On the next page, choose the option "Create a new Security Group". Take note that it's similar to the security groups you made for your instance earlier - similar to a "firewall" where you can say what kind of traffic is allowed. 

We'll leave things default for now, except for "Source" - for which we'll choose "My IP".

### 6.) Skip the next page, as we have no SSL settings.

### 7.) Configure your health checks

On the next page, we'll be configuring our *health checks*. Health checks essentially verify if the instance is "healthy" - if it's accepting traffic, or it's not receiving errors. Have a look at the tooltips for each of the options.

For now, we'll choose *TCP* as the Ping Protocol, using *80* as the Ping Port. What this is saying, is that the Load Balancer will check port 80 of any attached instances, and mark it as "healthy" if it responds without errors. 

### 8.) Set the instances to forward to

On the instances page, we'll select the instance that you created in the previous module. This essentially means that we'll be forwarding any traffic that hits the load balancer to your instance.

### 9.) Add tags

As usual, tags. We'll put a Key of *Name* and a Value of *myname*-elb.

### 10.) Finish up! 

Review all the settings you've had, then click *Create*

### 11.) See your ELB details

Click on the link that goes to your ELB. As with the instance, you'll see a pane that details the properties of your ELB. Make note of the *DNS Name*, marked below.


## Reinstall Wordpress using RDS

Because Wordpress keeps its URL data in the database, we'll need to reset your Wordpress installation. If your SSH session is still open then you're good to go - otherwise, go to the SSH instructions here.

### 12.) Reset Wordpress

Using your Terminal or Putty, go to the Wordpress directory, then remove the wp-config.php file:

```
cd /var/www/html/
rm wp-config.php
```

### 13.) Reinstall Wordpress

Now, open your browser and paste the *DNS Name* that you had in Step 11. This should show you an installation page. Proceed with the installation, but when you get to the panel that asks you for your database details, put in the following:

```
Database Host: rds.internal.devopsgirls
Database User: devopsgirls
Database Password: devopsgirlsrds
Database Prefix: firstname.lastname
```

Make sure you replace 'firstname.lastname' with your first name and last name. For example, 'Banana Smith' would have the database prefix of 'banana.smith'.

### 14.) Finish the installation

Finish the install, until it forwards you to the Blog Post page. Now that things are ready, we can start rolling our Wordpress artifact.

## Creating the artifact

### 15.) Create a copy of your Wordpress directory

If the installation went well, then you're going to want to create a copy of your installed Wordpress directory, so you won't have to run the install again. We do this via the Terminal or Putty application. We're going to use the following commands:

```
cd /var/www/html/
tar cvfz ~/firstname.lastname-wordpress.tgz .
```

With this command, changing directories to */var/www/html* - where Wordpress is installed. Then, we create a compressed tar file - firstname.lastname-wordpress.tgz - containing the contents of our current directory (.)

### 16.) Upload the file to S3

S3 is an object store - essentially allowing you to upload files to a directory that you can share either within the account, or to the world. We do this by running the following command:

```
aws s3 ~/firstname.lastname-wordpress.tgz s3://devopsgirls-training/firstname.lastname-wordpress.tgz
```

### 17.) Confirm the file exists using the web console:

Go to *Services > S3*. Click on the bucket called *devopsgirls-training*. If you uploaded your file correctly, then it should be there!

## Create a second instance

So: now we have a Load Balancer, and an Artifact. What we can do now is make it so that you have multiple instances, so that your service can still be up if one of the instances is down.

### 18.) Create a second instance

Go to *Services > EC2* in the web console. As with the first module, we're creating another instance.

