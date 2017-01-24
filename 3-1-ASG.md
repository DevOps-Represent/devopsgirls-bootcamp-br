# Launch Configurations and Auto Scaling Groups

## Key Concepts

Before we get started, here are a couple of loose concepts:

 - *Launch Configurations are a blueprint for how you want to make your instances*. This means that you configure it in the same way that you would configure an EC2 instance, with the difference that you're not actually making one - you're making a *plan* for your instance.

 - *Autoscaling Groups dictate how many instances you want to run.* It has many mechanisms for doing so - via manual intervention or system checks, but combined with Launch Configurations, you can automate the process of making instances.

 - *IAM Instance Profiles are machine users with limited permissions.* Think of it as a robot - it "impersonates" a person logging into your instance and doing things (in this case, installing Wordpress).

Now that's out of the way, let's get started!

## What we're going to do

In this practical session, we will:

 - Create a Launch Configuration

 - Specify UserData as we did with EC2 earlier

 - Attach it to a limited InstanceProfile

 - Create an Autoscaling Group

 - Configure Autoscaling group so it attaches instances to the ELB

 - Test scaling up or down
 

## Creating a Launch Configuration


### 1.) Creating your Launch Configuration

Go to Services > EC2, then look for *Launch Configurations* on the left-hand side. Click on *Create Launch Configuration*

### 2.) Set your Launch configuration size, and AMI

We will configure your Launch Configuration in the same way we configure your instance - with the following parameters:

```
 - AMI: Amazon Linux AMI
 - Instance Type: t2.micro
```

### 3.) Configure Launch Configuration details

We'll then set your Launch Configuration details. Name your Launch Config *myname-launchconfig*, then select the "DevOpsGirls-S3-Wordpress" IAM Role.


[image]

Note that this IAM role only allows read-only access to the contents of the S3 bucket you've placed your Wordpress package in - and nothing else. 

### 4.) Set Advanced Details

In the same panel, click on *Advanced Details*. In the *User Data* field, paste the following in:

```
#!/bin/bash
yum install -y mysql php php-mysql httpd
aws s3 cp s3://devopsgirls-training/firstname.lastname-wordpress.tgz /var/www/wordpress.tgz
tar xvfz /var/www/wordpress.tgz -C /var/www/html/
service httpd start
```

### 5.) Disable public IP addresses

In the same panel, select *Do not assign public IP*. This is an important security measure, and separates the instances from the public web - all interaction is via your Load Balancer.

### 6.) Set storage

We don't need to modify the storage, so we'll leave everything in *Add Storage* as default for now.

### 7.) Security Groups

We'll do the same thing that we did with your previous EC2 instance. In *Protocol*, select *HTTP* - then set *Source* to *"Custom IP"*. If you type out the name of the Load Balancer you created earlier, it should show up on the list.

The good thing about this is that for any instances we're creating, we're only ever expecting HTTP traffic *only from your Load Balancer*. This is a good thing.

### 8.) Verify your configuration, select the SSH KeyPair

We'll finish up with a review of your Launch Configuration. Check that everything is valid, then click on *Create Launch Configuration*

As with creating EC2 instances, this will ask you to specify a Key Pair. Specify the one you created earlier.

## Creating an AutoScaling Group

### 9.) Create an Autoscaling Group for your Launch Config

Once you click *Create Launch Configuration*, you should see another button called *Create an Autoscaling Group using this Launch Configuration*. Click on it.

### 10.) Set Autoscaling Group details

In the next panel, specify the GroupPame as *myname-asg*. Set Group Size to *2* instances.

### 11.) Select Network and Subnets

For this exercise, we'll use the VPC marked *default*. If you click on the *Subnet* input box, you should be given two to three subnets to choose from. Select *all of them* as destinations.

Subnets essentially set which Availability Zones to deply your instances under. Think of Availability Zones as datacenters - separate locations in Sydney where your instance will live. You'll want a good spread - this protects your service from outages if say, a datacenter in one Sydney location goes down.

### 12.) Set Load Balancer

If you click on *Advanced Details* under the same pane, you should see a tickbox called *Load Balancing*. This allows you to register the server as *live* and attaches it to the Load Balancer you created earlier.

An additional set of options should appear. If you click on the input box for *Classic Load Balancers*, you should be able to select the Load Balancer you created earlier.

### 13.) Keep scaling policies default

We're not going to be playing with Scaling Policies for the time being - so choose *Keep this group at its initial size*. Just keep in mind that you can use policies to watch for the CPU or memory use of your instances - so for example, if the CPU ever goes over 70% usage, you can choose to add more instances automatically.

### 14.) Skip notifications

We'll choose not to send notifications at this time. Think of this as a way of notifying someone if an event occurs - if say, an instance is launched or deleted.

### 15.) Configure tags

As with anything that we do, it's iportant to tag our to-be-created instances as soon as they're created. Set a Key of *Name*, and set the Value to your name.

### 16.) Review, and finish up!

You should be met with a section where you can review the changes that have occurred. Click *Create Auto Scaling Group* once you're ready, and it should kick things off.





## Testing your Autoscaling Group


