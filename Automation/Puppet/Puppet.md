---
title: Puppet
description: Puppet
published: true
date: 2019-04-17T08:01:32.583Z
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

$ puppet parser validate pasture/manifests/init.pp
$ puppet agent -t

$ curl 'node_name/api/v1/cowsay?message=Hello!'
```

The `<%= ... %>` tags we use to insert our variables into the file are called expression-printing tags. These tags insert the content of a Puppet expression, in this case the string values assigned to our variables.

The `epp()` function takes two arguments: First, a file reference in the format `'<MODULE>/<TEMPLATE_NAME>'` that specifies the template file to use. Second, a hash of variable names and values to pass to the template.

To avoid cramming all our variables into the `epp()` function, we'll put them in a variable called `$pasture_config_hash` just before the file resource.

# Writing a parameterized class

A class's parameters are defined as a comma-separated list of parameter name and default value pairs (`$parameter_name = default_value,`). These parameter value pairs are enclosed in parentheses (`(...)`) between the class name and the opening curly bracket (`{`) that begins the body of the class. For readability, multiple parameters should be listed one per line, for example:

```bash
class class_name (
  $parameter_one = default_value_one,
  $parameter_two = default_value_two,
){
 ...
}
```

Your parameter list will replace the variables assignments you used in the previous quest. By setting the parameter defaults to the same values you had assigned to those variables, you can maintain the same default behavior for the class.

```bash
$ vi pasture/manifests/init.pp
---
class pasture (
  $port                = '80',
  $default_character   = 'sheep',
  $default_message     = '',
  $pasture_config_file = '/etc/pasture_config.yaml',
){

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

$ puppet parser validate pasture/manifests/init.pp
```

To declare a class with specific parameters, use the resource-like class declaration. As the name suggests, the syntax for a resource-like class declaration is very similar to a resource declaration. It consists of the keyword class followed by a set of curly braces (`{...}`) containing the class name with a colon (`:`) and a list of parameters and values. Any values left out in this declaration are set to the defaults defined within the class, or `undef` if no default is set.

```bash
class { 'class_name':
  parameter_one => value_one,
  parameter_two => value_two,
}
```

Unlike the `include` function, which can be used for the same class in multiple places, resource-like class declarations can only be used once per class. Because a class declared with the `include` uses defaults, it will always be parsed into the same set of resources in your catalog. This means that Puppet can safely handle multiple `include` calls for the same class. Because multiple resource-like class declarations are not guaranteed to lead to the same set of resources, Puppet has no unambiguous way to handle multiple resource-like declarations of the same class. Attempting to make multiple resource-like declarations of the same class will cause the Puppet parser to throw an error.

```bash
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  class { 'pasture':
    default_character => 'cow',
  }
}

$ puppet agent -t

$ curl 'node_name/api/v1/cowsay?message=Hello!'
```

# Facts

```bash
$ facter -p | less
$ facter -p os
$ facter -p os.family
```

All facts are automatically made available within your manifests. You can access the value of any fact via the `$facts` hash, following the `$facts['fact_name']` syntax. To access structured facts, you can chain more names using the same bracket indexing syntax. For example, the `os.family` fact you accessed above is available within a manifest as `$facts['os']['family']`.

From your `modules` directory, create the directory structure for a module called `motd`. You'll need two subdirectories called `manifests` and `templates`.

`$ mkdir -p motd/{manifests,templates}`

Begin by creating an init.pp manifest to contain the main motd class.

```bash
$ vi motd/manifests/init.pp
---
class motd {

  $motd_hash = {
    'fqdn'       => $facts['networking']['fqdn'],
    'os_family'  => $facts['os']['family'],
    'os_name'    => $facts['os']['name'],
    'os_release' => $facts['os']['release']['full'],
  }

  file { '/etc/motd':
    content => epp('motd/motd.epp', $motd_hash),
  }

}
```

The `$facts` hash and top-level (unstructured) facts are automatically loaded as variables into any template. To improve readability and reliability, we strongly suggest using the method shown here. Be aware, however, that you will likely encounter templates that refer directly to facts using the general variable syntax rather than the `$facts` hash syntax we suggest here.

```bash
$ vi motd/templates/motd.epp
---
<%- | $fqdn,
      $os_family,
      $os_name,
      $os_release,
| -%>
Welcome to <%= $fqdn %>

This is a <%= $os_family %> system running <%= $os_name %> <%= $os_release %>

$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_name' {
  include motd
  class { 'pasture':
    default_character => 'cow',
  }
}
```

`$ puppet agent -t`

# Conditional statements

```bash
if $::osfamily == 'RedHat' {
  ...
  $apache_name = 'httpd'
  ...
} elsif $::osfamily == 'Debian' {
  ...
  $apache_name = 'apache2'
  ...
}

package { 'httpd':
  ensure => $ensure,
  name   => $::apache::apache_name,
  notify => Class['Apache::Service'],
}
```

Puppet supports a few different ways of implementing conditional logic:

- if statements,
- unless statements,
- case statements, and
- selectors.

First, add a `$sinatra_server` parameter with a default value of webrick.

Next add the `$sinatra_server` variable to the $pasture_config_hash so that it can be passed through to the configuration file template.

The beginning of your class should look like the following example:

```bash
pasture/manifests/init.pp
---
class pasture (
  $port                = '80',
  $default_character   = 'sheep',
  $default_message     = '',
  $pasture_config_file = '/etc/pasture_config.yaml',
  $sinatra_server      = 'webrick',
){

$pasture_config_hash = {
  'port'              => $port,
  'default_character' => $default_character,
  'default_message'   => $default_message,
  'sinatra_server'    => $sinatra_server,
}

$ vi pasture/templates/pasture_config.yaml.epp
---
<%- | $port,
      $default_character,
      $default_message,
      $sinatra_server,
| -%>
# This file is managed by Puppet. Please do not make manual changes.
---
:default_character: <%= $default_character %>
:default_message:   <%= $default_message %>
:sinatra_settings:
  :port:   <%= $port %>
  :server: <%= $sinatra_server %>
```

Now that your module is able to manage this setting, add a conditional statement to manage the required packages for the Thin and Mongrel webservers.

```bash
$ vi pasture/manifests/init.pp
---
class pasture (
  $port                = '80',
  $default_character   = 'sheep',
  $default_message     = '',
  $pasture_config_file = '/etc/pasture_config.yaml',
  $sinatra_server      = 'webrick',
){

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File[$pasture_config_file],
  }

  $pasture_config_hash = {
    'port'              => $port,
    'default_character' => $default_character,
    'default_message'   => $default_message,
    'sinatra_server'    => $sinatra_server,
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

  if ($sinatra_server == 'thin') or ($sinatra_server == 'mongrel')  {
    package { $sinatra_server:
      provider => 'gem',
      notify   => Service['pasture'],
    }
  }

}
```

With these changes to your class, you can easily accommodate different servers for different agent nodes in your infrastructure. For example, you may want to use the default WEBrick server on a development system and the Thin server on for production.

```bash
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_dev' {
  include pasture
}
node 'node_prod' {
  class { 'pasture':
    sinatra_server => 'thin',
  }
}
```

# Puppet Job

```bash
$ puppet access login --lifetime 1d
$ puppet job run --nodes node_dev,node_prod
$ curl 'node_dev/api/v1/cowsay?message=Hello!'
$ curl 'node_prod/api/v1/cowsay?message=Hello!'
```

# The Forge

The Puppet Forge is a public repository for Puppet modules. The Forge gives you access to community maintained modules. Using existing modules from the Forge allows you to manage a wide variety of applications and systems without spending extensive time on custom module development. Furthermore, because many Forge modules are actively used and maintained by the Puppet community, you'll be working with code that's already well reviewed and tested.

## Install from Forge

```bash
$ puppet module install puppetlabs-postgresql
$ ls /etc/puppetlabs/code/environments/production/modules

$ puppet module list
```

# Writing a wrapper class

Using the existing `postgresql` module, you can add a database component to the Pasture module without having to reinvent the Puppet code needed to manage a PostgreSQL server and database instance. Instead, we'll create what's called a *wrapper* class to declare classes from the `postgresql` module.

We'll call this wrapper class `pasture::db` and define it in a `db.pp` manifest in the `pasture` module's `manifests` directory.

```bash
$ vi pasture/manifests/db.pp
---
class pasture::db {

  class { 'postgresql::server':
    listen_addresses => '*',
  }

  postgresql::server::db { 'pasture':
    user     => 'pasture',
    password => postgresql_password('pasture', 'm00m00'),
  }

  postgresql::server::pg_hba_rule { 'allow pasture app access':
    type        => 'host',
    database    => 'pasture',
    user        => 'pasture',
    address     => '172.18.0.2/24',
    auth_method => 'password',
  }

}

$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'db_node' {
  include pasture::db
}
```

`$ puppet job run --nodes db_node`

Now that this database server is set up, let's add a parameter to our main pasture class to specify a database URI and pass this through to the configuration file.

Add a `$db` parameter with a default value of `'none'`. You'll see why we use `'none'` a little later. Add this `$db` variable to the `$pasture_config_hash` so it will be passed through to the template that defines the application's configuration file. When you've made these two additions, your class should look like the example below.

```bash
$ vi pasture/manifests/init.pp
---
class pasture (
  $port                = '80',
  $default_character   = 'sheep',
  $default_message     = '',
  $pasture_config_file = '/etc/pasture_config.yaml',
  $sinatra_server      = 'webrick',
  $db                  = 'none',
){

  package { 'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File[$pasture_config_file],
  }

  $pasture_config_hash = {
    'port'              => $port,
    'default_character' => $default_character,
    'default_message'   => $default_message,
    'sinatra_server'    => $sinatra_server,
    'db'                => $db,
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

  if ($sinatra_server == 'thin') or ($sinatra_server == 'mongrel')  {
    package { $sinatra_server:
      provider => 'gem',
      notify   => Service['pasture'],
    }
  }

}
```

Next, edit the `pasture_config.yaml.epp` template. We'll use a conditional statement to only include the `:db:` setting if there is a value other than `none` set for the `$db` variable.

```bash
$ vi pasture/templates/pasture_config.yaml.epp
---
<%- | $port,
      $default_character,
      $default_message,
      $sinatra_server,
      $db,
| -%>
# This file is managed by Puppet. Please do not make manual changes.
---
:default_character: <%= $default_character %>
:default_message: <%= $default_message %>
<%- if $db != 'none' { -%>
:db: <%= $db %>
<%- } -%>
:sinatra_settings:
  :port:   <%= $port %>
  :server: <%= $sinatra_server %>
```

Now that you've set up this `db` parameter, edit your `node_prod` node definition.

```bash
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node 'node_prod' {
  class { 'pasture':
    sinatra_server => 'thin',
    db             => 'postgres://pasture:m00m00@db_node/pasture',
  }
}

$ puppet job run --nodes node_prod
$ curl -X POST 'node_prod/api/v1/cowsay/sayings?message=Hello!'
$ curl 'node_prod/api/v1/cowsay/sayings'
$ curl 'node_prod/api/v1/cowsay/sayings/1'
```

# Roles and Profiles

As your Puppetized infrastructure grows in scale and complexity, you'll need to manage more and more kinds of systems. Defining all the classes and parameters for these systems directly in your `site.pp` manifest doesn't scale well. The `roles and profiles` pattern gives you a consistent and modular way to define how the components provided in your Puppet modules come together to define each different kind of system you need to manage.

A *profile* is a class that declares one or more related component modules and sets their parameters as needed. The set of profiles on a system defines and configures the technology stack it needs to fulfill its business role.

A *role* is a class that combines one or more profiles to define the desired state for a whole system. A role should correspond to the business purpose of a server. A role itself should **only** compose profiles and set their parameters—it should not have any parameters itself.

## Writing profiles

Using roles and profiles is a design pattern, not something written into the Puppet source code. As far as the Puppet parser is concerned, the classes that define your roles and profiles are no different than any other class.

`$ mkdir -p profile/manifests/pasture`

```bash
$ vi profile/manifests/pasture/app.pp
---
class profile::pasture::app {
  if $facts['fqdn'] =~ 'large' {
    $default_character = 'elephant'
    $db                = 'postgres://pasture:m00m00@pasture-db.puppet.vm/pasture'
  } elsif $facts['fqdn'] =~ 'small' {
    $default_character = 'cow'
    $db                = 'none'
  } else {
    fail("The ${facts['fqdn']} node name must match 'large' or 'small'.")
  }
  class { 'pasture':
    default_message   => 'Hello Puppet!',
    sinatra_server    => 'thin',
    default_character => $default_character,
    db                => $db,
  }
}

$ vi profile/manifests/pasture/db.pp
---
class profile::pasture::db {
  include pasture::db
}
```

While these profiles define the configuration of components directly related to the Pasture application, you'll typically need to manage aspects of these systems that aren't directly related to their business role. A profile module may include profile classes to manage things like user accounts, DHCP configuration, firewall rules, and NTP. Because these classes are applied across many or all of the systems in your infrastructure, the convention is to keep them in a `base` subdirectory. To give an example of a base profile, we'll create a `profile::base::motd` profile class to wrap the `motd` component class you created earlier.

```bash
$ mkdir profile/manifests/base
$ vi profile/manifests/base/motd.pp
---
class profile::base::motd {
  include motd
}
```

## Writing roles

A role combines profiles to define the full set of components you want Puppet to manage on a system. A role should consist of only `include` statements to pull in the list of profile classes that make up the role. A role should not directly declare non-profile classes or individual resources.

A role's name should be a simple description of the business purpose of the system it describes. Specific implementation details related to the technology stack are left to the profiles. For example, `role::myapp_webserver` and `role::myapp_database` are appropriate names for role classes, while `role::postgres_db` or `role::apache_server` are not.

`$ mkdir -p role/manifests`

```bash
$ vi role/manifests/pasture_app.pp
---
class role::pasture_app {
  include profile::pasture::app
  include profile::base::motd
}

$ vi role/manifests/pasture_db.pp
---
class role::pasture_db {
  include profile::pasture::db
  include profile::base::motd
}
```

## Classification

```bash
$ vi /etc/puppetlabs/code/environments/production/manifests/site.pp
---
node default {
  # This is where you can declare classes for all nodes.
  # Example:
  #   class { 'my_class': }
}

node /^pasture-app/ {
  include role::pasture_app
}

node /^pasture-db/ {
  include role::pasture_db
}

$ puppet job run --nodes pasture-db.puppet.vm
$ puppet job run --nodes pasture-app-small.puppet.vm,pasture-app-large.puppet.vm

$ curl 'pasture-app-small.puppet.vm/api/v1/cowsay?message="hello"'
$ curl 'pasture-app-large.puppet.vm/api/v1/cowsay?message="HELLO!"'

$ curl 'pasture-app-small.puppet.vm/api/v1/cowsay/sayings' (error because small app has no db)
```

# Hiera

Hiera is Puppet's data lookup system. Implementing Hiera in your Puppet infrastructure allows you to separate site-specific data from your Puppet code base. Hiera lookups in your Puppet code can then be used to intelligently set variables according to the specific details of each node where that code is applied.

Hiera is Puppet's built-in data lookup system. It lets you complete this separation by moving data out of your Puppet manifests and into a separate data source.

The first step in implementing Hiera is to add a hiera.yaml configuration file to your environment's code directory. This configuration file defines the levels in your hierarchy and tells Hiera where to find the data source that corresponds to each level.

`$ cd /etc/puppetlabs/code/environments/production`

```bash
$ vi hiera.yaml
---
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "Per-node data"
    path: "nodes/%{trusted.certname}.yaml"

  - name: "Per-domain data"
    path: "domain/%{facts.networking.domain}.yaml" 

  - name: "Common data"
    path: "common.yaml"
```

When Puppet uses Hiera to look for a value, it searches according to the order of levels listed under this configuration file's `hierarchy:` section. If a value is found in a data source defined for the "Per-node data" level, that value is used. If no matching value is found there, Hiera tries the next level: in this case, "Per-domain defaults". Finally, if no value is found in the previous data sources, Hiera looks in the "Common data" level's `common.yaml` file.

Before setting up your data sources for these levels, let's add our Hiera lookups to the `profile::pasture::app` class. By doing this first, we'll know which values the data sources need to define.

Here, use the built-in Hiera lookup() function to tell Puppet to fetch data for each of the pasture component class parameters you want to manage.

```bash
$ vi modules/profile/manifests/pasture/app.pp
---
class profile::pasture::app {
  class { 'pasture':
    default_message   => lookup('profile::pasture::app::default_message'),
    sinatra_server    => lookup('profile::pasture::app::sinatra_server'),
    default_character => lookup('profile::pasture::app::default_character'),
    db                => lookup('profile::pasture::app::db'),
  }
}
```

Note that these lookup keys include a fully qualified scope. Though Hiera itself doesn't require this naming pattern for these keys, this pattern allows anyone looking at a key in a data source to know exactly how and where it is used in Puppet code.

(Hiera also has an implicit data binding feature which makes direct use of fully qualified keys to set class parameters without an explicit lookup. Because this feature can make the relationship between Puppet code and Hiera data less clear, however, the lookup() function is preferred, especially when members of your team are less familiar with Hiera.)

Now that you've added lookup functions to your profile, it's time to define these data in your Hiera data sources.

`$ mkdir -p data/{domain,nodes}`

```bash
$ vi data/common.yaml
---
---
profile::pasture::app::default_message: "Baa"
profile::pasture::app::sinatra_server: "thin"
profile::pasture::app::default_character: "sheep"
profile::pasture::app::db: "none"

$ vi data/domain/beauvine.vm.yaml
---
---
profile::pasture::app::default_message: "Welcome to Beauvine!"

$ vi data/domain/auroch.vm.yaml
---
---
profile::pasture::app::default_message: "Welcome to Auroch!"
profile::pasture::app::db: "postgres://pasture:m00m00@pasture-db.auroch.vm/pasture"

$ vi data/nodes/pasture-app-dragon.auroch.vm.yaml
---
---
profile::pasture::app::default_character: 'dragon'
```

Your data directory should now look like the following:

```bash
data
├── common.yaml
├── domain
│   ├── auroch.vm.yaml
│   └── beauvine.vm.yaml
└── nodes
    └── pasture-app-dragon.auroch.vm.yaml
```

Next you will use the `puppet job` tool to trigger a puppet agent run on each of these nodes.

`$ puppet job run --nodes pasture-db.auroch.vm,pasture-app-dragon.auroch.vm,pasture-app.auroch.vm,pasture-app.beauvine.vm --concurrency 2`

```bash
$ curl pasture-app-dragon.auroch.vm/api/v1/cowsay/sayings
$ curl -X POST 'pasture-app-dragon.auroch.vm/api/v1/cowsay/sayings?message=Hello!'
$ curl pasture-app-dragon.auroch.vm/api/v1/cowsay/sayings/1
$ curl pasture-app.auroch.vm/api/v1/cowsay/sayings/1
$ curl pasture-app.beauvine.vm/api/v1/cowsay
```

# Defined resource types

As we noted in discussions of classes earlier in this guide, Puppet's classes are singleton. In other words, classes in the Puppet language can only exist once within a given catalog, and writing Puppet code that declares a class twice will result in a compilation error. For many system components, there is a natural fit between this idea of a singleton class and a single instance or installation of the class's managed component on the node. For example, an Apache server will generally run only a single instance of an `httpd` service and have one set of configuration files. In this case, Puppet's singleton classes ensure that you don't have multiple conflicting specifications for how a corresponding component should be configured.

In some cases, however, this one-to-one correspondence doesn't hold. A single Apache `httpd` process may serve multiple virtual hosts, for example.

> A defined resource type is a block of Puppet code that can be evaluated multiple times within a single node's catalog.

The syntax to create a defined resource type is very similar to the syntax you would use to define a class. Rather than using the `class` keyword, however, you use `define` to begin the code block.

```bash
define defined_type_name (
  parameter_one = default_value,
  parameter_two = default_value,
){
  ...
}
```

Once it's defined, you can use a defined resource type by declaring it like you would any other resource type.

```bash
defined_type_name { 'title':
  parameter_one => 'foo',
  parameter_two => 'bar',
}
```

A resource's *title* is the unique identifier Puppet user internally to keep track of that resource.
A resource's *namevar* is the parameter that specifies the unique aspect on the target system that the resource manages.

## Managing user accounts

```bash
$ cd /etc/puppetlabs/code/environments/production/modules
$ mkdir -p user_accounts/manifests
```

```bash
$ vi user_accounts/manifests/ssh_user.pp
---
define user_accounts::ssh_user (
  $pub_key,
  $group   = undef,
  $shell   = undef,
  $comment = undef,
){
  ssh_authorized_key { "${title}@puppet.vm":
    ensure => present,
    user   => $title,
    type   => 'ssh-rsa',
    key    => $pub_key,
  }
  file { "/home/${title}/.ssh":
    ensure => directory,
    owner  => $title,
    group  => $title,
    mode   => '0700',
    before => Ssh_authorized_key["${title}@puppet.vm"],
  }
  user { $title:
    ensure  => present,
    groups  => $group,
    shell   => $shell,
    home    => "/home/${title}",
    comment => $comment,
  }
  file { "/home/${title}":
    ensure => directory,
    owner  => $title,
    group  => $title,
    mode   => '0755',
  }
}
```

Note that the `$pub_key` parameter has no default value, while the default values for the other parameters are set to 'undef'. There is an important difference here.

A parameter with no supplied default value is required.

You may have noticed another new element of syntax into this code. Some of the double-quoted strings (`"..."`) in this manifest include variables in the `${var_name}` format. This is Puppet's string interpolation syntax. String interpolation allows you to insert a variable into any double-quoted string.

Rather than place this directly in your `site.pp` manifest, as you might have before we introduced the roles and profiles pattern, we'll create a `pasture_dev_users` profile class, and use a Hiera lookup to programmatically create users as needed on each system in your infrastructure.

`$ cd /etc/puppetlabs/code/environments/production/`

```bash
$ vi data/domain/beauvine.vm.yaml
---
---
profile::pasture::app::default_message: "Welcome to Beauvine!"
profile::base::dev_users::users:
  - title: 'inavarro'
    comment: 'Isart Navarro'
    pub_key: 'AAAAB3NzaC1yc2EAAAADAQABAAABAQDUXVg3uuFsliSLNlKi6K8nH7py4qu3VOch4o3zfGZN
YOSIdiUCVRUgV5CVYdDGps/UrmHVFa/qK4elLGvDmNtuAKnSsHcICUJbmGkaA36b0LipHazbLmrUEpNt7y
srtK0oqNayHGld9hKdSHerynP0IHNKEGyxqg+gB3W+KyeADIF4TgUczdgo0ZEA912F5wmP44TZsQCCj3YR
jspoJ8L734TVX+aSL3u2xxNDtli7PHs45LiGMIUlCy+/qKFbVUQrtnhy9GH5Tvb0D+g7TtTyRYJSr1C3Gu
T/mwbvCWZdCLWLwKJTfsDa7tNP6bZmpSELxdop4TKIzem5PzZ5dLaV
/'
  - title: 'user_two'
    comment: 'User Two'
   pub_key: 'AAAAB3NzaC1yc2EAAAADAQABAAABAQDUXVg3uuFsliSLNlKi6K8nH7py4qu3VOch4o3zfGZN
YOSIdiUCVRUgV5CVYdDGps/UrmHVFa/qK4elLGvDmNtuAKnSsHcICUJbmGkaA36b0LipHazbLmrUEpNt7y
srtK0oqNayHGld9hKdSHerynP0IHNKEGyxqg+gB3W+KyeADIF4TgUczdgo0ZEA912F5wmP44TZsQCCj3YR
jspoJ8L734TVX+aSL3u2xxNDtli7PHs45LiGMIUlCy+/qKFbVUQrtnhy9GH5Tvb0D+g7TtTyRYJSr1C3Gu
T/mwbvCWZdCLWLwKJTfsDa7tNP6bZmpSELxdop4TKIzem5PzZ5dLaV
/'
```

We'll also change the `common.yaml` data source to set a default for this key. In this case, we'll set the value to an empty list. This way, when Hiera tries to look up this value on a node outside of the `beauvine.vm` domain, it will still get a valid result.

```bash
$ vi data/common.yaml
---
---
profile::pasture::app::default_message: "Baa"
profile::pasture::app::sinatra_server: "thin"
profile::pasture::app::default_character: "sheep"
profile::pasture::app::db: "none"
profile::base::dev_users::users: []
```

Now that this data is available in Hiera, create a `dev_users.pp` manifest in `profile/manifests/base/`.

Here, we'll use a Puppet language feature called an iterator. An iterator allows you to repeat a block of Puppet code multiple times, using data from a hash or array to bind different values to the variables in the block for each iteration. In this case, the iterator goes through a list of users accounts defined in your Hiera data source and declares an instance of the `user_accounts::ssh_user` defined resource type for each.

```bash
$ vi /etc/puppetlabs/code/environments/production/modules/profile/manifests/base/dev_users.pp
---
class profile::base::dev_users {
  lookup(profile::base::dev_users::users).each |$user| {
    user_accounts::ssh_user { $user['title']:
        comment => $user['comment'],
        pub_key => $user['pub_key'],
    }
  }
}
```

With this `profile::base::dev_users` class is set up, add it to the `role::pasture_app` class.

```bash
$ vi /etc/puppetlabs/code/environments/production/modules/role/manifests/pasture_app.pp
---
class role::pasture_app {
  include profile::pasture::app
  include profile::base::dev_users
  include profile::base::motd
}
```

```bash
$ puppet job run --nodes pasture-app.beauvine.vm
$ ssh inavarro@pasture-app.beauvine.vm
```

# Control Repository

For the most part, a control repository should include the same contents you see currently in the `production` directory. However, there is one notable difference: rather than a `modules` subdirectory a control repository generally has distinct subdirectory called `site`. Your site directory should contains only the site-specific modules that you (or your team) have written specifically to manage your infrastructure, as these are the modules whose code should be managed through your source control process.

```bash
$ mkdir -p /root/control-repo/site
$ cd /etc/puppetlabs/code/environments/production/modules
$ cp -r {cowsay,pasture,motd,user_accounts,role,profile} /root/control-repo/site/
$ cd ..
```

Here, you need to make one more change to ensure that Puppet can actually find the modules you just copied into the `site` directory. Remember, Puppet can only find modules included in its `modulepath`. By default, this includes the `modules` directory, but not the `site` directory. For Puppet to find these site modules, this directory must be added to the `modulepath`. To do this, we'll use the `environment.conf` configuration file to override the Puppet master's default `modulepath` setting.

```
$ cp environment.conf /root/control-repo/environment.conf
$ vi /root/control-repo/environment.conf
---
modulepath = site:modules:$basemodulepath

$ mkdir /root/control-repo/manifests
$ cp manifests/site.pp /root/control-repo/manifests/site.pp
$ cp hiera.yaml /root/control-repo/hiera.yaml
$ cp -r data /root/control-repo/data
```

## Init repo

```bash
$ cd /root/control-repo
$ git init
$ git add *
$ [git status]
$ git commit -m Init
```

When you initialize a new repository, Git creates a default main branch called `master`. The convention for Puppet, however, is to call its main branch `production`. This way, the branch name matches up with the `production` code environment on your Puppet master.

```bash
$ git branch -m master production
$ git remote add upstream http://SERVER/learning/control-repo.git
$ [git remote -v]
$ git push upstream production
```

## Code Manager deploy key

The Code Manager tool lets you automate the deployment of Puppet code to your master from a control repository.

```bash
$ mkdir /etc/puppetlabs/puppetserver/ssh
$ ssh-keygen -t rsa -b 4096 -C "learning@puppet.vm"
```
When prompted, save the key to the following file:

`/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa`

When prompted for a passphrase, hit enter twice to create a key without a passphrase.

`$ chown -R pe-puppet:pe-puppet /etc/puppetlabs/puppetserver/ssh`

Add the ssh key to your repository.