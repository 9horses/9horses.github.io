---
title: Use WordPress to build a static website and host it on AWS for (practically) free.
date: 2020-03-30
description: Description
tags: [AWS]
---

![aws and wordpress](aws-wordpress.jpg)
<p style="text-align: center;"> AWS and WordPress can go together well </p>

## Introduction
Many people are using WordPress today to host websites that are primarily static. 
These are websites such as blogs that basically contain static content.
Such a website just needs to be updated when, e.g., a new article is written.

Today, static websites can be hosted in very cost-effective solutions like [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html) and [GitHub Pages](https://pages.github.com/).

Using one of these services to host your website (instead of WordPress) will very likely:

* save money,
* be more secure,
* speed up the website, and
* remove the need to manage/maintain WordPress and its host OS.

This article is for those people who like the ease of creating such websites via WordPress but wants these benefit of hosting their websites in AWS S3.


## Background
Recently, I decided to sign up as an 'expert' on [AWS IQ](https://aws.amazon.com/iq/) - it's a way for AWS customers to connect with third party consultants who are AWS experts.

There, I met [Mordecai Machazire](https://www.linkedin.com/in/mordecai-machazire-8279b4b0/), an impressive medical student, who is striving to learn data science. He had used AWS to provision an EC2 instance loaded with WordPress. And he in turn used WordPress to build his website and host it there.

The question he posed to me: "why is my simple website costing me $9 a month?"

Well, there are several ways to reduce the cost of hosting WordPress:

* Try to use a smaller instance type.
* Pre-buy EC2 instance hours ("reserved instance").
* Migrate to LightSail.
* Migrate to another WordPress hosting service.

But the core of the problem is you're still paying for idle compute.

And this is the precise scenario where just saving the site as a "static website" and hosting it on S3 will practically reduce the cost to the less than $1/month range. Additionally you'd get the benefits I've outlined above.

After a quick Google search, I came across a nice WordPress plugin called WP2Static that can save a WordPress site as a static wite. And so with Mordecai's permission, I've decided to put together this article.

## Solution Outline
Here's the high-level workflow for "having your cake and eat it too."  
I.e., use WordPress, but host the website for low price.

* Use AWS EC2 instance for running WordPress, use this to build and preview your static website.
* Use AWS S3 for hosting the static website.
* Use the [WP2Static](https://wp2static.com/) plugin to push the site to S3.

And be sure to power-off the EC2 instance when not actively using WordPress for building/updating the website!

For more details, I've also put together a walk-through below.

## Caveats
Note that this solution is not for those people using WordPress to host sites that needs dynamism. 

For example, if you're hosting a forum where users are uploading their own content, then you do _not_ have a static website. 

While most modern websites have a separation of static content and dynamic content (which are hosted separately), architecting a website this way is well beyond the scope of this article.


## Walk-through
### Get an AWS account.
If you don't already have one - AWS makes this [easy](https://portal.aws.amazon.com/billing/signup#/start).

### Register your Domain and create Route53 zone for DNS.
You'll want to host your website on a domain you like - e.g., http://www.my-fantastic-static-website.com.
First you'll need to purchase/register that domain either on [AWS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html) or elsewhere. 
And then you'll need to [configure Route53 as your DNS service](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring.html).

### Create S3 based static site, and an IAM user.
I've created a CloudFormation template (open-sourced) to simplify this step.
You can find it [here](https://github.com/joylogics/cloudformation#s3_website_for_wp2staticyaml)
Note there is a launcher you can use to launch this template in your AWS console.

This will create a bucket with a name which matches your website (e.g., "www.my-fantastic-static-website.com"). It will also create an IAM user specifically for uploading the content from WordPress to S3. You'll need the credentials in the output section of the CloudFormation to provide wp2static later.

At this point you can point your web browser to the specified URL to verify that the URL is reachable.
You should get a 404 error however, since there is no content in that website as yet.

### Create WordPress EC2 instance
There are several walk-throughs for doing this. One easy way is to (again) leverage CloudFormation.
AWS has a sample template they've open-sourced [here](https://github.com/awslabs/aws-cloudformation-templates/blob/master/aws/solutions/WordPress_Single_Instance.yaml).
I've copied it into my repository and placed a launcher for it [here](https://github.com/joylogics/cloudformation#wordpress_single_instanceyaml).

Note you'll need to [create a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) in AWS first.

### Install wp2static in WordPress
Here are the steps:

* Navigate to WordPress admin site, then login 
* Go to 'Plugins' tab in left navigation bar.
* "Add New"
* Search for "wp2static"
* "Install Now"
* "Activate"

### Build your website!
I'm hardly the expert on actually using WordPress to build your website. But I know there are many plugins that make this simple. In particular [elementor](https://elementor.com/) seems quite popular, and this is in fact the one that Mordecai had used.

### Use wp2static to export the site to S3
Now, deploy to the S3 website using w2static. Go to the WP2Static tab in the WordPress admin site, then select "Deploy static website" tab in the top. Fill in the information in the UI using the information you've captured from the steps above. Here's an example screen shot:
![](capture1.png)
Now, when you click on "Start static site export", the plugin will do all the heavy lifting of saving the site in S3.
When it's completed, click on "Go to my deployed site" to verify that your site is indeed live.

### Stop the WordPress EC2 instance!
Once you are happy with the website, make sure to stop the EC2 instance.
Just navigate to the [EC2 dashboard](https://console.aws.amazon.com/ec2/v2/home) on the AWS console, 
find the EC2 instance you've used, and stop (not terminate) the instance.
This effectively powers down that server, while the data is still on the EBS disk. 
When it's time to update the website again, return to the dashboard and start the instance.
(Note the public IP address will likely change.)

## After thoughts
### Backup
### SSL
### Staging
### Forms
### Login
### Even Lower cost options
#### Lightsail
#### Fargate
#### Laptop/Desktop


