# Load Balancing and High Availability.

## Key Concepts

Before we get started, here's a couple of loose concepts:

 - Load Balancers forward your traffic to one or more service providers. What this essentially means is that you can have multiple EC2 instances serving the same kind of content, being represented as one service.

 - High Availability means making it so that your system remains operational for a desirably long length of time. This means making it more durable to system failures - having failovers, in this case.

 - Service Separation refers to making sure that if you have multiple systems dependent on each other, that they exist separately and share fewer failure modes.

 - Artifacts are tangible byproducts of our systems development. In this case, our artifact will be the modified Wordpress package that's special to only you.

Now that that's out of the way, let's get started!

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


## Clear your current instance
