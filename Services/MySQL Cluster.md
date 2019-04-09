---
title: MySQL Cluster
description: MySQL Cluster
published: true
date: 2019-04-09T13:50:13.817Z
tags: 
---

Configure MySQL Cluster:

```
var dbPass = "mysqlpass"
var clusterName = "devCluster"

try {
  print('Setting up InnoDB cluster...\n');
  shell.connect('root@mysql-server-1:3306', dbPass)
  var cluster = dba.createCluster(clusterName);
  print('Adding instances to the cluster.');
  cluster.addInstance({user: "root", host: "mysql-server-2", password: dbPass})
  print('.');
  cluster.addInstance({user: "root", host: "mysql-server-3", password: dbPass})
  print('.\nInstances successfully added to the cluster.');
  print('\nInnoDB cluster deployed successfully.\n');
} catch(e) {
  print('\nThe InnoDB cluster could not be created.\n\nError: ' + e.message + '\n');
}
```

Manage MySQL Cluster

https://dev.mysql.com/doc/refman/5.7/en/mysql-innodb-cluster-working-with-cluster.html#retrieving-an-innodb-cluster
http://insidemysql.com/mysqlvp/idc/