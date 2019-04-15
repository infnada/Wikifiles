---
title: Puppet
description: Puppet
published: true
date: 2019-04-15T15:04:43.478Z
tags: 
---

# Install the basics

```
$ rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
$ yum install puppet-bolt
```

# Run commands with bolt

```
$ bolt command run 'free -th' --nodes localhost
$ bolt --format json command run 'cat /etc/hosts' --nodes localhost
```

# Install Puppet agent with bolt

```
$ bolt command run "sh -c 'curl -k https://learning.puppetlabs.vm:8140/packages/current/install.bash | sudo bash'" --nodes localhost
```

# Resources

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