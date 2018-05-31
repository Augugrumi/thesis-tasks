---
description: How to run floodlight with docker
---

# Floodlight and docker

## Prerequisites

First of all, you need an AWS account to be able to follow these steps. You can create a new one, and we'll use the free tier, so we won't waste money.

This page tells you how to set up floodlight with docker.

### Components to install

You have to install these components on your PC \(we assume you use Linux, otherwise you'll be on your own\):

* Vagrant
* AWS plugin for Vagrant \([https://github.com/mitchellh/vagrant-aws](https://github.com/mitchellh/vagrant-aws)\)
* SSH

### Getting AWS credentials

We assume you're proficient with AWS, so as you know when you use AWS API you need an access token and a secret key. Grab one from your IAM console, and be sure it can manage EC2 instances \(in particular, be sure `create` and `destroy` permissions are given\)

## Fun stuff

After you've set up successfully your machine, we're ready to start. Go on your AWS console and create an EC2 micro instance, then delete it. This will also create a key pair \(keep it!\) and automatically set up a default security group, that we're gonna use.

Now clone the `vagrantfiles` repository`:`

```bash
$ git clone https://github.com/Augugrumi/vagrantfiles.git
```

{% hint style="info" %}
If you want to use the same version the author is using at the moment of writing this guide, check out to this SHA: `de3f6a31c31b2b888fa2a9961fcf26eeb566d56d`. Simply open a terminal and type:

```bash
cd vagrantfiles; git checkout de3f6a31c31b2b888fa2a9961fcf26eeb566d56d
```
{% endhint %}

Move inside the repository, then `cd` into ovs-vagrant folder and finally type:

```bash
export ACCESS_KEY_ID = "your aws access key id"
export SECRET_ACCESS_KEY = "your secret key"
export SSH_KEY_PATH = "path/to/the/.pam/file"
vagrant up
# wait for the machine to go up...
# ...if the command successfully exit
vagrant ssh
# you should now ssh'ed in the VM
```

Once you've copy-pasted all this boring stuff, you have a working sandbox.

