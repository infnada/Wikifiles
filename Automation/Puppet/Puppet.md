---
title: Puppet
description: Puppet
published: true
date: 2019-04-15T15:14:27.889Z
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

# The site.pp manifest

When a Puppet agent contacts the Puppet master, the master checks for any node definitions in the `site.pp` manifest that match the agent system's name. In the Puppet world, the term "node" is used to mean any system or device in your infrastructure, so a node definition defines how Puppet should manage a given system.

## Example basic node declaration

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