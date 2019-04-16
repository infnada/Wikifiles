---
title: Puppet
description: Puppet
published: true
date: 2019-04-16T09:03:30.678Z
tags: 
---

# Bolt

Bolt provides a command-line interface for running commands, scripts, tasks and plans on the local machine or remote nodes.

```bash
$ rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
$ yum install puppet-bolt
```

## Run commands with bolt

```bash
$ bolt command run 'free -th' --nodes localhost
$ bolt --format json command run 'cat /etc/hosts' --nodes localhost
```

## Install Puppet agent with bolt

```bash
$ bolt command run "sh -c 'curl -k https://learning.puppetlabs.vm:8140/packages/current/install.bash | sudo bash'" --nodes localhost
```

# Puppet Resources

Once the agent is installed on a new system, you will use the puppet resource tool to explore the state of that system.

The term Puppet is often used as a shorthand for a collection of tools and services that work in concert to help define your infrastructure configuration and then automate the process of bringing the systems into their desired state and keeping them there.

Using Puppet, you define a desired state for all systems in your infrastructure. Once this state is defined, it automates the process of bringing your systems into that state and keeping them there.

## Check if resource (file) exists
```bash
$ puppet resource file /tmp/test
$ touch /tmp/test
$ puppet resource file /tmp/test
```

## Put contents into a file (development)
```bash
$ sudo puppet resource file /tmp/test content='Hello Puppet!'
```

## Check if resource (package) exists
```bash
$ puppet resource package httpd
$ puppet resource package httpd ensure=present
```

# Sign agent certificate

```bash
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

```bash
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  notify { 'Hello Puppet!': }
}
```

```bash
$ puppet agent -t

Notice: Hello Puppet!
Notice: /Stage[main]/Main/Node[node_name]/Notify[Hello Puppet!]/message: defined 'message' as 'Hello Puppet!'
Notice: Applied catalog in 0.45 seconds
```

### Example simple manifest
```bash
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

```bash
$ puppet config print modulepath

/etc/puppetlabs/code/environments/production/modules:/etc/puppetlabs/code/modules:/opt/puppetlabs/puppet/modules
```

This directory includes modules specific to the production environment. (The second directory contains modules used across all environments, and the third is modules that PE uses to configure itself).

In a real production environment, however, you would likely want to keep your Puppet code in a version control repository such as Git and use Puppet's code manager tool to deploy it to your master

### Example module

You might be wondering why we're calling it init.pp instead of cowsay.pp. Most modules contain a main class like this whose name corresponds with the name of the module itself. This main class is always kept in a manifest with the special name init.pp.

```bash
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

```bash
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  include cowsay
}
```

Before applying any changes to your system, it's always a good idea to use the --noop flag to do a practice run of the Puppet agent. This will compile the catalog and notify you of the changes that Puppet would have made without actually applying any of those changes to your system.

`$ puppet agent -t --noop`

If your dry run looks good, go ahead and run the Puppet agent again without the --noop flag.

`$ puppet agent -t`

Now you can try out your newly installed cowsay command:

`$ cowsay Puppet is awesome!`

### Composed classes and class scope

A module often includes multiple components that work together to serve a single function.

```bash
$ vi cowsay/manifests/fortune.pp
---
class cowsay::fortune {
  package { 'fortune-mod':
    ensure => present,
  }
}

$ puppet parser validate cowsay/manifests/fortune.pp
```

We could use another include statement in the `site.pp` manifest to classify `node_name` with this `cowsay::fortune` class. In general, however, it's best to keep your classification as simple as possible.

In this case, use a class declaration to pull the `cowsay::fortune` class into our main `cowsay` class.

```bash
$ vi cowsay/manifests/init.pp
---
class cowsay {
  package { 'cowsay':
    ensure   => present,
    provider => 'gem',
  }
  include cowsay::fortune
}

$ puppet parser validate cowsay/manifests/init.pp

$ puppet agent -t --noop
$ puppet agent -t
$ fortune | cowsay
```

# The package, file, service pattern

The `package`, `file`, and `service` resource types are used in concert often enough that we talk about them together as the "package, file, service" pattern. A package resource manages the software package itself, a file resource allows you to customize a related configuration file, and a service resource starts the service provided by the software you've just installed and configured.

## Package

```bash
$ cd /etc/puppetlabs/code/environments/production/modules
$ mkdir -p pasture/{manifests,files}
$ vi pasture/manifests/init.pp
---
class pasture {
  package { 'pasture':
    ensure   => present,
    provider => gem,
  }
}

$ puppet parser validate pasture/manifests/init.pp
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  include pasture
}

$ puppet agent -t
$ pasture start &
$ curl 'localhost:4567/api/v1/cowsay?message=Hello!'
$ curl 'localhost:4567/api/v1/cowsay?message=Hello!&character=elephant'
```

## File

Packages you install with Puppet often have configuration files that let you customize their behavior.

```
vi pasture/files/pasture_config.yaml
---
:default_character: elephant
```

The `file` resource takes a `source` parameter, which allows you to specify a source file that will define the content of the managed file. As its value, this parameter takes a URI. While it's possible to point to other locations, you'll typically use this to specify a file in your module's '`files` directory. Puppet uses a shortened URI format that begins with the `puppet:` prefix to refer to these module files kept on your Puppet master. This format follows the pattern `puppet:///modules/<MODULE NAME>/<FILE PATH>`. Notice the triple forward slash just after `puppet:`. This stands in for the implied path to the modules on your Puppet master.

```
$ vi pasture/manifests/init.pp
---
class pasture {

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
  }

  file { '/etc/pasture_config.yaml':
    source => 'puppet:///modules/pasture/pasture_config.yaml',
  }
}

$ puppet parser validate pasture/manifests/init.pp
```

## Service

Because our agent node is running CentOS 7, we'll use the `systemd` service manager to handle our Pasture process. Although some packages set up their own service unit files, Pasture does not. It's easy enough to use a file resource to create your own. This service unit file will tell systemd how and when to start our Pasture application.

```bash
$ vi pasture/files/pasture.service
---
[Unit]
Description=Run the pasture service

[Service]
Environment=RACK_ENV=production
ExecStart=/usr/local/bin/pasture start

[Install]
WantedBy=multi-user.target

$ vi pasture/manifests/init.pp
---
class pasture {

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
  }

  file { '/etc/pasture_config.yaml':
    source => 'puppet:///modules/pasture/pasture_config.yaml',
  }

  file { '/etc/systemd/system/pasture.service':
    source => 'puppet:///modules/pasture/pasture.service',
  }

  service { 'pasture':
    ensure => running,
  }

}

$ puppet parser validate pasture/manifests/init.pp
```

## Resource ordering

We need a way to ensure that the resources defined in this class are managed in the correct order. The "package, file, service" pattern describes the common dependency relationships among these resources: we want to install a package, write a configuration file, and start a service, in that order. Furthermore, if we make changes to the configuration file later, we want Puppet to restart our service so it can pick up those changes.

For our class, we'll use two relationship metaparameters: `before` and `notify`. `before` tells Puppet that the current resource must come before the target resource. The `notify` metaparameter is like `before`, but if the target resource is a service, it has the additional effect of restarting the service whenever Puppet modifies the resource with the metaparameter set.

Relationship metaparameters take a resource reference as a value. This resource reference points to another resource in your Puppet code. The syntax for a resource reference is the capitalized resource type, followed by square brackets containing the resource title: `Type['title']`.

```bash
$ vi pasture/manifests/init.pp
---
class pasture {

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File['/etc/pasture_config.yaml'],
  }

  file { '/etc/pasture_config.yaml':
    source  => 'puppet:///modules/pasture/pasture_config.yaml',
    notify  => Service['pasture'],
  }

  file { '/etc/systemd/system/pasture.service':
    source  => 'puppet:///modules/pasture/pasture.service',
    notify  => Service['pasture'],
  }

  service { 'pasture':
    ensure => running,
  }

}

$ puppet parser validate pasture/manifests/init.pp

$ puppet agent -t

$ curl 'node_name:4567/api/v1/cowsay?message=Hello!'
```

# Variables

A variable name is prefixed with a `$` (dollar sign), and a value is assigned with the `=` operator. Assigning a short string to a variable, for example, looks like this:

`$my_variable = 'look, a string!'`

Once you have defined a variable, you can use it anywhere in your manifest where you want to use the assigned value. Note that variables are parse-order dependent, which means that a variable must be defined before it can be used. Trying to use an undefined variable will result in a special `undef` value. Though this may result in explicit errors, in some cases it will still lead to a valid catalog with unexpected contents.

Technically, Puppet variables are actually *constants* from the perspective of the Puppet parser as it parses your Puppet code to create a catalog. Once a variable is assigned, the value is fixed and cannot be changed. The *variability*, here, refers to the fact that a variable can have a different value set across different Puppet runs or across different systems in your infrastructure.

```bash
$ vi pasture/manifests/init.pp
---
class pasture {

  $port                = '80'
  $default_character   = 'sheep'
  $default_message     = ''
  $pasture_config_file = '/etc/pasture_config.yaml'

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File[$pasture_config_file],
  }
  file { $pasture_config_file:
    source  => 'puppet:///modules/pasture/pasture_config.yaml',
    notify  => Service['pasture'],
  }
  file { '/etc/systemd/system/pasture.service':
    source => 'puppet:///modules/pasture/pasture.service',
    notify  => Service['pasture'],
  }
  service { 'pasture':
    ensure    => running,
  }
}
```

# Templates

Many of the tasks involved in system configuration and administration come down to managing the content of text files. The most direct way to handle this is through a templating language. A template is similar to a text file but offers a syntax for inserting variables as well as some more advanced language features like conditionals and iteration. This flexibility lets you manage a wide variety of file formats with a single tool.

Puppet supports two templating languages, Embedded Puppet (EPP) and Embedded Ruby (ERB).

```bash
$ mkdir pasture/templates
$ vi pasture/templates/pasture_config.yaml.epp
---
<%- | $port,
      $default_character,
      $default_message,
| -%>
# This file is managed by Puppet. Please do not make manual changes.
---
:default_character: <%= $default_character %>
:default_message:   <%= $default_message %>
:sinatra_settings:
  :port: <%= $port %>
```

The bars (`|`) surrounding the list of parameters are a special syntax that define the parameters tag. The `<%` and `%>` are the opening and closing tag delimiters that distinguish EPP tags from the body of the file. Those hyphens (`-`) next to the tag delimiters will remove indentation and whitespace before and after the tag. This allows you to put this parameter tag at the beginning of the file, for example, without the newline character after the tag creating an empty line at the beginning of the output file.

Now that that's set, we can repeat the process for the service unit file.

```bash
$ cp pasture/files/pasture.service pasture/templates/pasture.service.epp
$ vi pasture/templates/pasture.service.epp
---
<%- | $pasture_config_file = '/etc/pasture_config.yaml' | -%>
# This file is managed by Puppet. Please do not make manual changes.
[Unit]
Description=Run the pasture service

[Service]
Environment=RACK_ENV=production
ExecStart=/usr/local/bin/pasture start --config_file <%= $pasture_config_file %>

[Install]
WantedBy=multi-user.target
```
And modify the file resource for your service unit file to use the template you just created.

```bash
$ vi pasture/manifests/init.pp
---
class pasture {

  $port                = '80'
  $default_character   = 'sheep'
  $default_message     = ''
  $pasture_config_file = '/etc/pasture_config.yaml'

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File[$pasture_config_file],
  }
  $pasture_config_hash = {
    'port'              => $port,
    'default_character' => $default_character,
    'default_message'   => $default_message,
  }
  file { $pasture_config_file:
    content => epp('pasture/pasture_config.yaml.epp', $pasture_config_hash),
    notify  => Service['pasture'],
  }
  $pasture_service_hash = {
    'pasture_config_file' => $pasture_config_file,
  }
  file { '/etc/systemd/system/pasture.service':
    content => epp('pasture/pasture.service.epp', $pasture_service_hash),
    notify  => Service['pasture'],
  }
  service { 'pasture':
    ensure    => running,
  }
}
```

The `<%= ... %>` tags we use to insert our variables into the file are called expression-printing tags. These tags insert the content of a Puppet expression, in this case the string values assigned to our variables.

The `epp()` function takes two arguments: First, a file reference in the format `'<MODULE>/<TEMPLATE_NAME>'` that specifies the template file to use. Second, a hash of variable names and values to pass to the template.

To avoid cramming all our variables into the `epp()` function, we'll put them in a variable called `$pasture_config_hash` just before the file resource.