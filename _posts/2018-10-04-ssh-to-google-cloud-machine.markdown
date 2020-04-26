---
layout: post
title: "Setting up SSH access from local machine to Google Cloud Compute engine"
description: "I am very much used to aws and I love the easiness of working with pem files and accessing the ec2 instances over ssh. Today I was trying to play around with Google Cloud Platform. It is very easy to bring up a compute engine (similar to aws ec2)."
date: 2018-10-05 00:00:00
comments: true
image: https://cdn-images-1.medium.com/max/1600/1*eWYm3Dm7s0t8lLWYRW4pow.jpeg
keywords: "gcloud, Google Cloud PlatformSsh,Google Compute Engine"
category: tech
tags:
- tech
---

I am very much used to aws and I love the easiness of working with pem files and accessing the ec2 instances over ssh. Today I was trying to play around with Google Cloud Platform. It is very easy to bring up a compute engine (similar to aws ec2). Also the dashboard was similar to aws (but if you are used to aws terminology you might stumble across all the options google has).

After setting up the instance I was going to connect from my local machine to the compute engine and then only I realised the absence of pem files, I didn’t create any!, because they didn’t prompt for one, because they don’t have the concept of one.

They do have a button in the machine’s row on the compute engine dashboard called SSH. When you click on that, a new browser window will popup with a terminal on it. This is a fully functional terminal which you can use to setup, debug and monitor your compute engine. But I was not satisfied with it and I needed a way to use my local machine to connect to the compute engine (not through a browser window). Another reason I wanted it so badly was because it is easy to setup.

## Let’s set it up!

I would like to split the entire process in to two sections, one to be performed on the local machine and the other on the Google Cloud Platform console.

<b>On your local machine:</b>

1. Open Terminal.
2. `$ ssh-keygen -t rsa -C [username] `
    * replace [username] with the value you need.
    * It will prompt you for a directory to place the files that are going to get generated (you can just hit ENTER and keep it to default).
    * Then it will prompt you for a passphrase (you can also hit ENTER and opt not to set a passphrase, but I would recommend it. Also I would recommend to remember it 😉).
    * If it is done there will be to new files generated under the directory you opted, by default it will be under ‘.ssh/’ and will be named id_rsa and id_rsa.pub, this is a private-public key pair.

Now copy the id_rsa.pub (public key) and head over to the next section.

## On Google cloud console:

1. Login to your account
2. Navigate to Compute Engine section
3. Select Metadata from the left side pane
4. Click on Edit
5. Click Add item and then paste the public key in to text box and hit Save.
6. Again select the option VM Instances from left pane
7. From the dashboard choose your machine
8. Click Edit
9. Under the section SSH Keys select show and edit and paste the public key again in the text box and hit Save

If everything is done right, you will be able to ssh in to the compute engine from your local machine 😎by `$ ssh -i id_rsa [username]@[public ip]`

