# CWF

For personal reference only

## Part 1

### Requirement 1 - Create UAT Docker Container

1. Configure host's host file
```
sudo vi /etc/hosts

#TODO: modify
172.20.113.194 sddo-vm.localdomain sddo-vm
192.168.120.130 3984565a_uatsvr.localdomain
192.168.120.140 3984565a_prodsvr.localdomain
```

2. Create custom network
```
docker network create --subnet=192.168.120.0/24 customnetwork
```

3. Create custom image
```
docker run -it --name puppetclient ubuntu:18.04 /bin/bash

apt-get update
apt-get install curl wget vim iputils-ping net-tools openssh-server
wget https://apt.puppetlabs.com/puppet7-release-bionic.deb
dpkg -i puppet7-release-bionic.deb
vi /etc/apt/sources.list.d/puppet7-release.list #TODO: change http to https
apt-get update
apt-get install puppet-agent

docker commit puppetclient puppetclient-image
```

4. Create Container
```
#UAT Container
docker run -dit --network customnetwork --privileged -h 3984565a_uatsvr.localdomain \
  --name 3984565a_uatsvr --add-host=sddo-vm:172.20.113.194  --ip=192.168.120.130 \
  -p 32500:80 puppetclient-image /sbin/init

#Prod Container
docker run -dit --network customnetwork --privileged -h 3984565a_prodsvr.localdomain \
  --name 3984565a_prodsvr --add-host=sddo-vm:172.20.113.194  --ip=192.168.120.140 \
  -p 32600:80 puppetclient-image /bin/bash
```

5. Configure container's host file
```
sudo vi /etc/hosts

#TODO: UAT
172.20.113.194 sddo-vm
192.168.120.130 3984565a_uatsvr.localdomain 3984565a_uatsvr

#TODO: Prod
172.20.113.194 sddo-vm
192.168.120.140 3984565a_prodsvr.localdomain 3984565a_prodsvr
```

6. Configure puppet client settings
```
vim /etc/puppetlabs/puppet/puppet.conf
```
```
#TODO: UAT
certname = 3984565a_uatsvr.localdomain
server = sddo-vm.localdomain

#TODO: Prod
certname = 3984565a_prodsvr.localdomain
server = sddo-vm.localdomain
```

7. Register puppet agent with puppet master
```
/opt/puppetlabs/bin/puppet agent -t
```

8. Extend path variable to include the puppet command
```
vi ~/.bashrc
export PATH=$PATH:/opt/puppetlabs/bin
.  ~/.bashrc
```

9. Sign certificates on puppet master
```
sudo /opt/puppetlabs/bin/puppetserver ca list --all
sudo /opt/puppetlabs/bin/puppetserver ca sign --all
```

10. Install puppet bolt on host
```
wget https://apt.puppet.com/puppet-tools-release-bionic.deb
dpkg -i puppet-tools-release-bionic.deb
vi /etc/apt/sources.list.d/puppet-tools-release.list #TODO: change http to https
apt-get update
apt-get install puppet-bolt
```

11. Add new users in containers
```
docker exec -it [3984565a_uatsvr | 3984565a_prodsvr] /bin/bash
apt-get update
apt-get install sudo
useradd clientadm  -m  -d /home/clientadm
passwd clientadm
usermod -G sudo clientadm
```

12. Test puppet bolt command
```
bolt command run "ls /opt" -t [3984565a_uatsvr | 3984565a_prodsvr].localdomain \
  -u clientadm -p ...  --no-host-key-check  --run-as root
```

## Requirement 2 - Jenkins Pipeline

Poll interval: */2 * * * *

### Requirement 3 - Required repo files

1. Jenkins file - 3984565a_jenfile
2. index.html - 3984565a_index.html
3. Script - 3984565a_script
   - Use puppet command to ensure apache2 service is installed and running
   - Use puppet command to remove directory /tmp/3984565a/research and recreate directory /tmp/3984565a/research (to clean up the directory)
   - Cloning 3984565a_opr_repo to container’s /tmp/3984565a/research
   - Copy new 3984565a_index.html to replace /var/www/html/index.html
     (Important: This script is used by bolt to ensure apache2 is installed and to replace the index.html file via puppet resource commands on the target container). 

### Requirement 4 - Results of curl test

UAT: `/tmp/uatsvr-result-file`
Prod: `/tmp/prodsvr-result-file`

### Requirement 5 - Contents of html file

Refer to index.html

### Other Requirements
- Screenshot of console output
- Before and after update of index.html

## Part 2

To be populated

## References
- [Puppet Bolt](https://www.puppet.com/docs/bolt/latest/bolt.html)
- [Puppet Man Pages](https://www.puppet.com/docs/puppet/7/man/overview)
- [Puppet Resource types](https://www.puppet.com/docs/puppet/7/resource_types)
