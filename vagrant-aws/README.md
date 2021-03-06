# powerstrip-flocker demo on AWS

This demo will guide you through the steps needed to run a multi-node powerstrip-flocker demo using AWS.  We use Vagrant to spin up and manage the AWS nodes.

## Requirements

Ensure that you have the following installed on your system:

 * [vagrant](http://www.vagrantup.com/downloads.html) **1.7.2 or later** (e.g. download from the website, don't use Ubuntu package repo)
 * [python](https://www.python.org/downloados/) 2.7

Also you need the `yaml` python module installed on your machine - you may wish to run `sudo pip install pyyaml`

## Setup

#### Get vagrant-aws

We need the vagrant AWS plugin to provision machines with AWS - once vagrant is installed, type:

```bash
$ vagrant plugin install vagrant-aws
$ vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
```

**NB:** On Ubuntu installing the latest vagrant (1.7.2+) from http://www.vagrantup.com/downloads.html was necessary.

#### Clone repository

Make sure you have checked out this repo and are in the vagrant-aws directory:

```bash
$ git clone https://github.com/clusterhq/powerstrip-flocker
$ cd powerstrip-flocker/vagrant-aws
```

#### `settings.yml`

Copy `settings.yml.sample` to `settings.yml` and fill in the details.

**NB:** This demo only works in AWS region us-east-1.

#### `vagrant up`

Now we type `vagrant up --provider=aws` which will spin up the two nodes.

```bash
$ vagrant up --provider=aws
```

#### `./push-config.py`

Wait for your nodes to come up, then use the provisioner script which also introduces the two nodes to each other:

```bash
$ ./push-config.py
```

This will also set up system services on the nodes.

## Demo

Now we have the 2 machines - we can use flocker to migrate our data!

On the first node we write some data to a flocker volume using nothing but the standard docker client.

```bash
$ vagrant ssh node1
node1$ sudo docker run -v /flocker/test:/data ubuntu sh -c "echo powerflock > /data/file.txt"
node1$ exit
```

On the second node we trigger a migration of that data and read it - using nothing but the standard docker client!

```bash
$ vagrant ssh node2
node2$ sudo docker run -v /flocker/test:/data ubuntu cat /data/file.txt
powerflock
node2$ exit
```

## Explanation

What just happened was the following:

 * the `docker run` command on node1 was intercepted by powerstrip
 * powerstrip sent the volume information to powerstrip-flocker
 * powerstrip-flocker created the volume with flocker
 * the container started and wrote some data to the volume
 * the `docker run` command on node2 was sent to powerstrip-flocker
 * it noticed that this volume had already been created on node1
 * flocker then moved the volume over the node2
 * the data that was written to node1 was read from node2

## Caveat

Warning: do not use this for anything!
As well as being as a massive hack, and a prototype, this demo uses ephemeral instance storage for the ZFS pool.
So you *will* lose your data.

[Let us know what you think!](https://github.com/ClusterHQ/powerstrip-flocker/issues/new)
