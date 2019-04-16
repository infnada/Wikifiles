---
title: Puppet
description: Puppet
published: true
date: 2019-04-16T07:28:40.567Z
tags: 
---

# Bolt

Bolt provides a command-line interface for running commands, scripts, tasks and plans on the local machine or remote nodes.

```
$ rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
$ yum install puppet-bolt
```

## Run commands with bolt

```
$ bolt command run 'free -th' --nodes localhost
$ bolt --format json command run 'cat /etc/hosts' --nodes localhost
```

## Install Puppet agent with bolt

```
$ bolt command run "sh -c 'curl -k https://learning.puppetlabs.vm:8140/packages/current/install.bash | sudo bash'" --nodes localhost
```

# Puppet Resources

Once the agent is installed on a new system, you will use the puppet resource tool to explore the state of that system.

The term Puppet is often used as a shorthand for a collection of tools and services that work in concert to help define your infrastructure configuration and then automate the process of bringing the systems into their desired state and keeping them there.

Using Puppet, you define a desired state for all systems in your infrastructure. Once this state is defined, it automates the process of bringing your systems into that state and keeping them there.

## Check if resource (file) exists
```
$ puppet resource file /tmp/test
$ touch /tmp/test
$ puppet resource file /tmp/test
```

## Put contents into a file (development)
```
$ sudo puppet resource file /tmp/test content='Hello Puppet!'
```

## Check if resource (package) exists
```
$ puppet resource package httpd
$ puppet resource package httpd ensure=present
```

# Sign agent certificate

```
$1 puppet agent -t
$2 puppetserver ca list
$2 puppetserver ca sign --certname some_cert_name
$1 puppet agent -t
```

# Manifests

At its simplest, a Puppet manifest is Puppet code saved to a file with the .pp extension. This code is written in the Puppet domain specific language (DSL).

## The site.pp manifest

When a Puppet agent contacts the Puppet master, the master checks for any node definitions in the `site.pp` manifest that match the agent system's name. In the Puppet world, the term "node" is used to mean any system or device in your infrastructure, so a node definition defines how Puppet should manage a given system.

### Example basic node declaration

Normally you would include one or more class declarations in this node block. A class defines a group of related resources, allowing them to be declared as a single unit. Using classes in your node definitions keeps them simple and well organized and encourages the reuse of code.

In this case, use a resource type called `notify` that will display a message in the output of your Puppet run without making any changes to the system.

```
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  notify { 'Hello Puppet!': }
}
```

```
$ puppet agent -t

Notice: Hello Puppet!
Notice: /Stage[main]/Main/Node[node_name]/Notify[Hello Puppet!]/message: defined 'message' as 'Hello Puppet!'
Notice: Applied catalog in 0.45 seconds
```

### Example simple manifest
```
$ vi /tmp/hello.pp
---
notify { 'Hello Puppet!': }

$ puppet apply /tmp/hello.pp 
```

## Clases and modules

A class is named block of Puppet code. Defining a class combines a group of resources into single reusable and configurable unit. Once defined, a class can then be declared to tell Puppet to apply the resources it contains.

A class should bring together a set of resources that manage one logical component of a system. For example, a class written to manage a MS SQL Server might include resources to manage the package, configuration files, and service for the MS SQL Server instance.

A module is a directory structure that lets Puppet keep track of where to find the manifests that contain your classes. A module also contains other data a class might rely on, such as the templates for configuration files. When you apply a class to a node, the Puppet master checks in a list of directories called a *modulepath* for a module directory matching the class name. The master then looks in that module's `manifests` subdirectory to find the manifest containing the class definition.

To see what your configured modulepath is, run the following command:

```
$ puppet config print modulepath

/etc/puppetlabs/code/environments/production/modules:/etc/puppetlabs/code/modules:/opt/puppetlabs/puppet/modules
```

This directory includes modules specific to the production environment. (The second directory contains modules used across all environments, and the third is modules that PE uses to configure itself).

In a real production environment, however, you would likely want to keep your Puppet code in a version control repository such as Git and use Puppet's code manager tool to deploy it to your master

### Example module

You might be wondering why we're calling it init.pp instead of cowsay.pp. Most modules contain a main class like this whose name corresponds with the name of the module itself. This main class is always kept in a manifest with the special name init.pp.

```
$ cd /etc/puppetlabs/code/environments/production/modules
$ mkdir -p cowsay/manifests
$ vi cowsay/manifests/init.pp
---
class cowsay {
  package { 'cowsay':
    ensure   => present,
    provider => 'gem',
  }
}

$ puppet parser validate cowsay/manifests/init.pp
```

Apply the class to a node:

```
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  include cowsay
}
```

Before applying any changes to your system, it's always a good idea to use the --noop flag to do a practice run of the Puppet agent. This will compile the catalog and notify you of the changes that Puppet would have made without actually applying any of those changes to your system.

`$ puppet agent -t --noop`
