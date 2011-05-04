---
layout: post
title: "Deploying Redmine on Amazon EC2"
permalink: /2011/05/deploying-redmine-on-amazon-ec2
---

It's insanely easy to get [Redmine](http://www.redmine.org) up and running on [Amazon EC2](http://aws.amazon.com/ec2/).  I went through the process
today and thought it would make sense to document it on here for others to read.

##Why EC2

You may ask yourself, "Why EC2? [Heroku](http://www.heroku.com) has a free tier of rails hosting". You're right, they do.  The difference is, EC2 gives you a full linux install with 10gb of persistent storage, whereas heroku gives you a 5mb database.  Couple storage differences with the ability hook redmine up to your local git repo on your EC2 instance, upload files and documents, and do full backup snapshots to S3 out of the box, and EC2 seemed like an easy choice to me.

##Start to Finish in 5 Minutes  

If you don't already have an EC2 account, go sign up for one now.  You won't be able to do any of this without an account.  Once you are signed up, and in the EC2 management console, click on "Launch Instance".  A window will pop up showing 6 sections. Here is a quick overview of what to select in each section:

### Choose an AMI 
* Click the Community AMIs tab
* Set viewing to All Images
* Type "bitnami redmine" in the box next to "Viewing"
* As of writing this, the most recent version is 1.1.2 on Ubuntu with ebs as the root device.  Find the most recent version in the list and click the appropriate "Select" button

### Instance Details
* Set instance type to Micro
* Click Continue
* Set any User Data you'd like, and click Continue again
* Set any tags you'd like, and click Continue again

### Create Key Pair

* Create a new key pair (or use an existing set)
* Download a copy of your keypair
* Click Continue

### Configure Firewall

* Click Continue

### Review

* Click Launch

### Adjust Security Group

* Click on Security Groups in the left hand bar
* Select your default group
* Click the Inbound tab in the details pane
* Add preset rules for SSH, HTTP and HTTPS
* Click Apply Rules

Now, you can get your public dns name from your instances dashboard, and you should be up and running with a copy of redmine.  To boot, this installation falls under the "free" tier of AWS, which means you won't have to pay the hefty $0.02/hr cost of running this instance, if you've got no other instances running.

Happy Bug Tracking!

