<h2>Achieving high availability with AWS Auto Scaling Group (ASG) and Application Load Balancer (ALB) Integration</h2>

<h3>Below is the step by step guide to auto-scale your system:</h3>

***Step-1:*** Go to VPC console and create a VPC.

![vpc](/assets/images/vpc1.png)

![vpc](/assets/images/vpc2.png)

***Step-2:*** Create 3 public subnets, where our EC2 instances will be dynamically stored.

![subnet](/assets/images/subnet1.png)

![subnet](/assets/images/subnet2.png)

![subnet](/assets/images/subnet3.png)

![subnet](/assets/images/subnet4.png)

![subnet](/assets/images/subnet5.png)

> [!IMPORTANT]
> Don’t forget to check the ‘Enable auto-assign public IPv4 address’ box for all three subnets.

![subnetip](/assets/images/subnetip1.png)

![subnetip](/assets/images/subnetip2.png)

***Step-3:*** Create an internet gateway to allow our subnets to connect to the internet.

![igate](/assets/images/igate1.png)

![igate](/assets/images/igate2.png)

***Step-4:*** Attach VPC to internet gateway.

![attachig](/assets/images/attachig1.png)

![attachig](/assets/images/attachig2.png)

***Step-5:*** Now configure the route table to properly direct traffic from the subnets to the internet gateway.

> [!NOTE]
> When we created our VPC, a route table was automatically created. In the ‘Route Tables’ dashboard, you should see the new entry, which is attached to our VPC.

![rtable](/assets/images/rtable1.png)

![rtable](/assets/images/rtable2.png)

![rtable](/assets/images/rtable3.png)

> [!NOTE]
> We need to route all traffic (0.0.0.0/0) from the subnets to the internet gateway.

![route](/assets/images/route1.png)

![route](/assets/images/route2.png)

***Step-6:*** Go to EC2 dashboard and go to Security group section and create a security group to allow SSH and incoming HTTP traffic.

![sg](/assets/images/sg1.png)

![sg](/assets/images/sg2.png)

![sg](/assets/images/sg3.png)

![sg](/assets/images/sg4.png)

***Step-7:*** Go to EC2 dashboard and go to Launch Templates section and create a EC2 launch template that will inform the type of EC2 instances the Auto scaling group should create.

![launch](/assets/images/launch1.png)

![launch](/assets/images/launch2.png)

![launch](/assets/images/launch3.png)

![launch](/assets/images/launch4.png)

![launch](/assets/images/launch5.png)

> [!IMPORTANT]
> Add this script that will install and run an Apache web server every time an instance is launched. Paste the script in the ‘User data’ field under the ‘Advanced details’ section.

```bash
#!/bin/bash

#update all
sudo yum update -y

#install apache
sudo yum install -y httpd

#enable and start apache
sudo systemctl enable httpd
sudo systemctl start httpd

sudo echo '<!DOCTYPE html>

<html lang="en">
 <head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />

  <title>Web Server Started</title> />

  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link
   href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700;800&display=swap"
   rel="stylesheet"
  />

  <link rel="stylesheet" href="css/styles.css?v=1.0" />
 </head>

 <body>
  <div class="wrapper">
   <div class="container">
    <h1>Welcome! An Apache web server has been started successfully.</h1>
    <p>Replace this with your own index.html file in /var/www/html.</p>
   </div>
  </div>
 </body>
</html>

<style>
 body {
  background-color: #34333d;
  padding-top: 128px;
 }

 .container {
  box-sizing: border-box;
  width: 740px;
  height: 440px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  padding: 48px;
  box-shadow: 0px 1px 32px 11px rgba(38, 37, 44, 0.49);
  background-color: #5d5b6b;
  overflow: hidden;
  flex-wrap: nowrap;
  gap: 24;
  border-radius: 24px;
 }

 .container h1 {
  flex-shrink: 0;
  width: 100%;
  position: relative;
  color: #ffffff;
  line-height: 1.2;
  font-size: 40px;
 }
 .container p {
  position: relative;
  color: #ffffff;
  line-height: 1.2;
  font-size: 18px;
 }
</style>
' > index.html
```

![launch](/assets/images/launch6.png) 

![launch](/assets/images/launch7.png)

***Step-8:*** Go to EC2 console and go to Auto Scaling Groups section and create an Auto scaling group (ASG).

![ag](/assets/images/ag1.png)

![ag](/assets/images/ag2.png)

![ag](/assets/images/ag3.png)

> [!NOTE]
> We will create a load balancer in the next step, so we can skip this step for now.

![ag](/assets/images/ag4.png)

> [!NOTE]
> It might take one or two minutes to fully initialize the Auto Scaling Group, but if we go to our EC2 Instances dashboard, we should see that two instances are up and running!

![running](/assets/images/running.png)

> [!NOTE]
> We should be able to go to their public IP addresses to see if the Apache web servers have been installed.

![working](/assets/images/working.png)

***Step-9:*** Go to EC2 dashboard and go to Target Groups section and create a target group for our ALB to direct traffic to.

![tg](/assets/images/tg1.png)

![tg](/assets/images/tg2.png)

![tg](/assets/images/tg3.png)

![tg](/assets/images/tg4.png)

![tg](/assets/images/tg5.png)

***Step-10:*** Go to EC2 dashboard and go to Load Balancers section and create our Application Load Balancer (ALB) to properly route internet traffic to the auto scaling group across multiple subnets.

![alb](/assets/images/alb1.png)

![alb](/assets/images/alb2.png)

![alb](/assets/images/alb3.png)

![alb](/assets/images/alb4.png)

![alb](/assets/images/alb5.png)

![alb](/assets/images/alb6.png)

> [!IMPORTANT]
> Now that we've created our load balancer, we need to connect it to our Auto Scaling Group. So let’s go back to our Auto Scaling Group and scroll down to ‘Load balancing’. Select the Application load balancer and the target group we created.


***Step-11:*** Go to EC2 console and go to Auto Scaling Groups section and click on the Auto scaling group we created and add the Application load balancer we created.

![asgalb](/assets/images/asgalb1.png)

![asgalb](/assets/images/asgalb2.png)

***Step-12:*** Let’s double-check our work by going to the load balancer DNS and see if our site still loads properly.

![lbcheck](/assets/images/lbcheck.png)


> [!WARNING]
> Once you’re done using your Auto Scaling Group and Application Load Balancer, be sure to delete them or you’ll get charged for the running EC2 instances. The instances will automatically terminate.

<h3>We’ve successfully created an auto scaling web app environment that dynamically scales our EC2 instances and properly routes traffic to the right target groups!</h3>