# Instalación estándard de CoreOS para Docker

## Configurar inition.json

> https://coreos.com/os/docs/latest/provisioning.html

- Especificar recursos de red:
```
"networkd": {
  "units": [
    {
      "contents": "[Match]\nName=ens192\n\n[Network]\nAddress=172.17.2.121/24\nGateway=172.17.2.101\nDNS=172.17.2.39",
      "name": "static.network"
    }
  ]
}
```

- Usuarios:

> Cómo generar un hash de contraseña:
```
# On Debian/Ubuntu (via the package "whois")
$ mkpasswd --method=SHA-512 --rounds=4096

# OpenSSL (note: this will only make md5crypt.  While better than plantext it should not be considered fully secure)
$ openssl passwd -1

# Python
$ python -c "import crypt,random,string; print(crypt.crypt(input('clear-text password: '), '\$6\$' + ''.join([random.choice(string.ascii_letters + string.digits) for _ in range(16)])))"

# Perl (change password and salt values)
$ perl -e 'print crypt("password","\$6\$SALT\$") . "\n"'
```

```
"passwd": {
    "users": [
      {
        "groups": [
          "sudo",
          "docker"
        ],
        "name": "core",
        "passwordHash": "HASH",
        "sshAuthorizedKeys": [
          "ssh-rsa AAAA..."
        ]
      },
      {
        "groups": [
          "sudo",
          "docker"
        ],
        "name": "tuusuario",
        "sshAuthorizedKeys": [
          "ssh-rsa AAAA..."
        ]
      }
    ]
  },
```

- Escritura de archivos
```
"storage": {
  "files": [
  
    // Update strategy
  
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/coreos/update.conf",
      "user": {},
      "contents": {
        "source": "data:,GROUP%3Dstable%0AREBOOT_STRATEGY%3D%22off%22%0ASERVER%3Dhttps%3A%2F%2Fpublic.update.core-os.net%2Fv1%2Fupdate%2F%0A",
        "verification": {}
      },
      "mode": 420
    },
    
    // Set hostname
    
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/hostname",
      "user": {},
      "contents": {
        "source": "data:,dBCNT1worker02%0A",
        "verification": {}
      },
      "mode": 420
    },
    
    // Set DNS
    
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/resolv.conf",
      "user": {},
      "contents": {
        "source": "data:,nameserver%09172.17.2.39%0Anameserver%09172.17.2.40%0A",
        "verification": {}
      },
      "mode": 420
    },
    
    // Set max_map_count
    
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/sysctl.conf",
      "user": {},
      "contents": {
        "source": "data:,vm.max_map_count=262144%0A",
        "verification": {}
      },
      "mode": 420
    },
    
    // Set Docker plugin vfile conf
    
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/vfile.conf",
      "user": {},
      "contents": {
        "source": "data:,%7B%22MaxLogAgeDays%22%3A28%2C%22MaxLogFiles%22%3A10%2C%22MaxLogSizeMb%22%3A10%2C%22LogPath%22%3A%22%2Fvar%2Flog%2Fvfile%2Elog%22%7D%0A",
        "verification": {}
      },
      "mode": 420
    },
    
    // // Set Docker plugin vsphere conf
    
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/vsphere-storage-for-docker.conf",
      "user": {},
      "contents": {
        "source": "data:,%7B%22MaxLogAgeDays%22%3A28%2C%22MaxLogFiles%22%3A10%2C%22MaxLogSizeMb%22%3A10%2C%22LogPath%22%3A%22%2Fvar%2Flog%2Fvsphere%2Dstorage%2Dfor%2Ddocker%2Elog%22%7D%0A",
        "verification": {}
      },
      "mode": 420
    },
    
    // Set ulimits
    
    {
      "filesystem": "root",
      "group": {},
      "path": "/etc/security/limits.conf",
      "user": {},
      "contents": {
        "source": "data:,%2A%09hard%09memlock%09unlimited%0A%2A%09soft%09memlock%09unlimited%0A",
        "verification": {}
      },
      "mode": 420
    }
  ]
}
```

- Units:
```
"systemd": {
  "units": [
  
    // Service limits
  
    {
      "enable": true,
      "dropins": [
        {
          "contents": "[Service]\nLimitMEMLOCK=infinity",
          "name": "10-memlock.conf"
        }
      ],
      "name": "docker.service"
    },
    
    // Set max_map_count
    
    {
      "contents": "[Unit]\nDescription=Set Max Map Count\n\n[Service]\nType=oneshot\nExecStart=/usr/sbin/sysctl -w vm.max_map_count=16777216\n\n[Install]\nWantedBy=multi-user.target",
      "enable": true,
      "name": "runsysctl.service"
    },
    
    // Install Docker plugins
    
    {
      "contents": "[Unit]\nDescription=Install vsphere plugin\nAfter=docker.service\nRequires=docker.service\n\n[Service]\nType=oneshot\nExecStart=/usr/bin/docker plugin ls | grep -q 'vsphere' && echo \"matched\" || /usr/bin/docker plugin install --alias vsphere vmware/vsphere-storage-for-docker:latest --grant-all-permissions \"VDVS_SOCKET_GID=233\"\nExecStart=/usr/bin/docker plugin ls | grep -q 'vfile' && echo \"matched\" || /usr/bin/docker plugin install --alias vfile vmware/vfile:latest VFILE_TIMEOUT_IN_SECOND=90 \"VDVS_SOCKET_GID=233\" --grant-all-permissions\n\n[Install]\nWantedBy=multi-user.target",
      "enable": true,
      "name": "runcmd.service"
    }
  ]
}
```

## Instalar Coreos con inition.json en vSphere

`$ coreos-install -d /dev/sda -i ignition.json -o vmware_raw`
