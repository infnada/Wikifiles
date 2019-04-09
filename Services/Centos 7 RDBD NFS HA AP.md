---
title: Centos7 RDBD NFS HA AP
description: Centos7 RDBD NFS HA AP
published: true
date: 2019-03-08T19:25:16.027Z
tags: 
---

# Active/Pasive NFS with file replication


`mkdir /nfsshare`

## Create lvm


```
pvcreate /dev/sdb
vgcreate vg_nfs /dev/sdb
lvcreate -l 100%FREE --thinpool lv_nfs vg_nfs
--lvs
```

## Install required packages

```
yum update
http://elrepo.org/tiki/tiki-index.php
yum install -y drbd90-utils kmod-drbd90
modprobe drbd
echo drbd > /etc/modules-load.d/drbd.conf
lsmod | grep -i drbd
``` 

```
yum install -y pcs
systemctl enable pcsd && systemctl start pcsd
systemctl enable pacemaker
systemctl enable corosync
```

```
yum install dnf
dnf install 'dnf-command(config-manager)'
dnf config-manager --add-repo http://download.opensuse.org/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-7/network:ha-clustering:Stable.repo
yum install crmsh
```

```
vi etc/corosync/corosync.conf
----
totem {
    version: 2
    cluster_name: nfs_cluster
    secauth: off
    transport: udpu
}

nodelist {
    node {
        ring0_addr: dBCNnfs02
        nodeid: 1
    }

    node {
        ring0_addr: dMADnfs02
        nodeid: 2
    }

    node {
        ring0_addr: dQUOnfs02
        nodeid: 3
    }
}
```
```
vi /etc/default/corosync
----
START=yes
```
```
passwd hacluster
pcs cluster auth dBCNnfs02 dMADnfs02 dQUOnfs02
 Username: hacluster
 Password: PASSWORD
```




------ SOLO EN 1 SRV -------

```
pcs cluster setup --name nfs_cluster dBCNnfs02 dMADnfs02 dQUOnfs02
pcs cluster start --all
pcs cluster enable --all
--pcs status cluster
--pcs status
pcs property set stonith-enabled=false
pcs property set no-quorum-policy=ignore
--corosync-cmapctl | grep members
```
----------------------------


```
vi /etc/drbd.d/global_common.conf
---
global {
        usage-count no;
}

common {
        net {
                protocol C;
        }
}

vi /etc/drbd.d/drbd0.res
---
resource r0 {
   net {
      protocol C;
   }
   on dBCNnfs02 {
      device /dev/drbd0;
      disk /dev/vg_nfs/lv_nfs;
      node-id 0;
      address 172.17.4.25:7788;
      meta-disk internal;
   }
   on dMADnfs02 {
      device /dev/drbd0;
      disk /dev/vg_nfs/lv_nfs;
      node-id 1;
      address 172.17.4.24:7788;
      meta-disk internal;
   }
   on dQUOnfs02 {
      device /dev/drbd0;
      disk /dev/vg_nfs/lv_nfs;
      node-id 2;
      address 172.17.4.26:7788;
      meta-disk internal;
   }

   connection-mesh {
      hosts dBCNnfs02 dMADnfs02 dQUOnfs02;
      net {
         use-rle no;
      }
   }
}
```

```
drbdadm create-md r0

systemctl start drbd
systemctl enable drbd

drbdadm up r0
--drbdadm status r0
--cat /proc/drbd
--drbdmon
--drbdadm dstate r0
- drbdsetup status --verbose --statistics
```

----- ON MASTER NODE -----
```
drbdadm primary r0 --force
mkfs.xfs /dev/drbd0
```
--------------------------

```` 
pcs resource create nfs02-vol ocf:linbit:drbd drbd_resource=r0 op monitor interval=30s
pcs resource master nfs02-clone nfs02-vol master-max=1 master-node-max=1 clone-max=2 clone-node-max=1 notify=true

pcs resource create nfs02_fs Filesystem device="/dev/drbd0" directory="/nfsshare" fstype="xfs" options=uquota,gquota
pcs constraint colocation add nfs02_fs with nfs02-clone INFINITY with-rsc-role=Master
pcs constraint order promote nfs02-clone then start nfs02_fs

pcs resource create nfs02_vipT3 ocf:heartbeat:IPaddr2 ip=172.17.4.27 cidr_netmask=24 op monitor interval=30s
pcs constraint colocation add nfs02_vipT3 with nfs02_fs INFINITY
pcs constraint order nfs02_fs then nfs02_vipT3

pcs resource create nfs02_vipT2 ocf:heartbeat:IPaddr2 ip=172.17.3.27 cidr_netmask=24 op monitor interval=30s
pcs constraint colocation add nfs02_vipT2 with nfs02_fs INFINITY
pcs constraint order nfs02_fs then nfs02_vipT2

pcs resource create nfs-service nfsserver nfs_shared_infodir=/nfsshare/nfsinfo op monitor interval=30s --group nfsresourcegroup
pcs constraint colocation add nfs-service with nfs02_vipT3 INFINITY
pcs constraint colocation add nfs-service with nfs02_vipT2 INFINITY
pcs constraint order nfs02_vipT3 then nfs-service


pcs resource create nfs_export_root exportfs clientspec="172.17.4.0/24" options=rw,sync,no_root_squash directory=/nfsshare/exports fsid=0 --group nfsresourcegroup
pcs constraint colocation add nfs_export_root with nfs-service INFINITY

pcs resource create nfs_export_crouchdb_data exportfs clientspec="172.17.4.0/24" options=rw,sync,no_root_squash directory=/nfsshare/exports/crouchdb_data fsid=1 --group nfsresourcegroup
pcs constraint colocation add nfs_export_crouchdb_data with nfs-service INFINITY

pcs resource create nfs_export_portainer_data exportfs clientspec="172.17.4.0/24" options=rw,sync,no_root_squash directory=/nfsshare/exports/portainer_data fsid=2 --group nfsresourcegroup
pcs constraint colocation add nfs_export_portainer_data with nfs-service INFINITY

pcs resource create nfs_export_traefik_acme exportfs clientspec="172.17.4.0/24" options=rw,sync,no_root_squash directory=/nfsshare/exports/traefik_acme fsid=3 --group nfsresourcegroup
pcs constraint colocation add nfs_export_traefik_acme with nfs-service INFINITY
```


-----------------------
```
drbdadm resume-sync r0
drbdadm pause-sync r0
```

-----------------------
