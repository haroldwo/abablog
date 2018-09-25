---
title: "Use Cobbler to Deploy Ansible Controller."
date: 2018-09-25T12:00:00+08:00
weight: 20
keywords: ["cobbler","ansible"]
description: "Use Cobbler to Deploy Ansible Controller."
tags: ["cobbler","ansible"]
categories: ["Ops"]
author: "Fred"
banner: ""
---

## 1. Scenario.

How about to use a light application for IaaS. As we all know, the platform like OpenStack is awesome but too heavy. For small business, we always do not create a complex architecture. Since Kubernetes(K8S) appears, we give more scope to it in IaaS. Now, I will use [Cobbler](http://cobbler.github.io/manuals/2.8.0/) and Ansible to manage IaaS. They are much simpler than other platform such as OpenStack, Rancher, Vmware and so on. However, this architecture will be easy, simple and efficient.

Please see below picture for the scope.

[![cobbler03.png](https://i.postimg.cc/brztCGqq/cobbler03.png)](https://postimg.cc/5YRyj2Hr)

The only thing I need to do is to connect the server to the network and to note the NIC MAC address before using Cobbler to automatically execute OS installation task.

## 2. Install Cobbler.

It is **important** that if we have several(n) ISO files to be imported, we should prepare (2 x n x capacity of ISO) free disk space at less for this server.

Here is a simple introduction for Cobbler.

[![cobbler02.png](https://i.postimg.cc/k5NM1qwc/cobbler02.png)](https://postimg.cc/FfRQ1tMd)

[![cobbler01.png](https://i.postimg.cc/mDwDDBHD/cobbler01.png)](https://postimg.cc/TLKGN8QM)

I choose yum to install.
```
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y install cobbler pykickstart fence-agents dhcp xinetd
```
You can search the relevant "fence-agents" for your server and install "debmirrors" if you need to import Debian system.

After that, please manually modify the tftp config on `/etc/xinetd.d/tftp`. Modify "disable" option from "yes" to "no".

Then, further configuration is as following.
```
# my OS: CentOS Linux release 7.4.1708 (Core)
# please customize these two var
pass=$(openssl passwd -1 "redhat")
dhcpRange=172.16.0.230 172.16.0.249

# some var got from network config. please check if it works for you.
localhost=$(/sbin/ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:")
net=$(echo $localhost | awk 'BEGIN{FS=OFS="."}{$NF=0;print}')
gateway=$(cat /etc/sysconfig/network-scripts/ifcfg-e* | grep GATEWAY | awk -F = '{print $2}')
dns=$(cat /etc/sysconfig/network-scripts/ifcfg-e* | grep DNS1 | awk -F = '{print $2}')

sudo sed -i '/default_password_crypted/d' /etc/cobbler/settings
sudo sed -i '/and put the output between the/a\default_password_crypted: "'"$pass"'"' /etc/cobbler/settings
sudo sed -i 's/server: 127.0.0.1/server: '"$localhost"'/g' /etc/cobbler/settings
sudo sed -i 's/func_auto_setup: 0/func_auto_setup: 1/g' /etc/cobbler/settings
sudo sed -i 's/register_new_installs: 0/register_new_installs: 1/g' /etc/cobbler/settings
sudo sed -i 's/anamon_enabled: 0/anamon_enabled: 1/g' /etc/cobbler/settings
sudo sed -i 's/subnet 192.168.1.0/subnet '"$net"'/g' /etc/cobbler/dhcp.template
sudo sed -i 's/option routers             192.168.1.5/option routers             '"$gateway"'/g' /etc/cobbler/dhcp.template
sudo sed -i 's/option domain-name-servers 192.168.1.1/option domain-name-servers '"$dns"'/g' /etc/cobbler/dhcp.template
sudo sed -i 's/range dynamic-bootp        192.168.1.100 192.168.1.254/range dynamic-bootp        '"$dhcpRange"'/g' /etc/cobbler/dhcp.template
sudo setenforce 0
sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
cobbler get-loaders
mkdir -p /usr/share/cobbler/web
sudo setsebool -P httpd_can_network_connect true
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-port=80/tcp
sudo firewall-cmd --add-port=69/udp
sudo firewall-cmd --add-port=51234-51235/tcp
sudo firewall-cmd --runtime-to-permanent
sudo systemctl enable cobblerd &&
sudo systemctl start cobblerd
sudo systemctl enable httpd &&
sudo systemctl start httpd
sudo systemctl enable rsyncd &&
sudo systemctl start rsyncd
sudo systemctl enable dhcpd &&
sudo systemctl start dhcpd
sudo systemctl enable xinetd &&
sudo systemctl start xinetd
cobbler sync
```

## 3. Import ISO.

I downloaded those ISO and create a file formated as `/path/to/ISO  isoIndex`. This is my example.
```
~/CentOS-7-x86_64-DVD-1708.iso   CentOS-7-x86_64
~/CentOS-6-x86_64-DVD.iso    CentOS-6-x86_64
```
Then, import ISO to Cobbler.
```
# please customize the first var with the file we just created.
isoList=/path/to/file
maxLine=$(cat $isoList | wc -l)

for ((line = 1; $line <= $maxLine; line++)); do
  isoName=$(head -n $line $isoList | awk '{print $1}');
  cobblerISO=$(head -n $line $isoList | awk '{print $2}');
  mkdir /mnt/$cobblerISO
  mount -t iso9660 -o loop,ro $isoName /mnt/$cobblerISO;
  cobbler import --name=$cobblerISO --path=/mnt/$cobblerISO;
done
cobbler distro list
# show the ISO list
```

## 4. Deploy Ansible.

After we note the NIC MAC address of the server, we need to configure some parameters for the installation. You can customize these info. Please read the [manual](http://cobbler.github.io/manuals/2.8.0/3/1/3_-_Systems.html) first.
```
cobbler system add --name=ansible --profile=CentOS-7-x86_64
cobbler system edit --name=ansible --interface=ens33 --mac=00:50:56:3D:16:32 \
--ip-address=172.16.0.12 --netmask=255.255.255.0 --static=1 --dns-name=ansible.mydomain.com \
--name-servers=172.16.0.253 --kickstart=/var/lib/cobbler/kickstarts/ansible.ks \
--if-gateway=172.16.0.253 --hostname=ansible
```

Then, please create a file which involving the packages you want to install. I will call this file with some command in kickstart. This is the example.
```
@base
@compat-libraries
@debugging
@development
net-tools
zip
unzip
bzip2
tree
psmisc
wget
gcc
gcc-c++
openssl
openssl-devel
zlib
zlib-devel
sysstat
```

Please create a script file located in any folder of `/var/www/cobbler/` which will be used to deploy Ansible. This is because I used some command in kickstart to call this script from the target server. I put it on `/var/www/cobbler/pub/ansible.sh`. Below script is for your reference.
```
ansible_pass=admin
ansible_account=admin

wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate
python get-pip.py
pip install http://github.com/diyan/pywinrm/archive/master.zip#egg=pywinrm
pip install --upgrade pip
pip install pyvmomi # for VMware

yum -y install epel-release expect # expect is used for further ansible deployment. you can ignore it.
yum -y install ansible
useradd $ansible_account
echo "$ansible_pass" | passwd $ansible_account --stdin
echo "$ansible_account ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
runuser -l $ansible_account -c 'ssh-keygen -f ~/.ssh/id_rsa -N ""'
echo '[uncatalogued]' >> /etc/ansible/hosts
```

The Snippets feature is useful for the [kickstart template](http://cobbler.github.io/manuals/2.8.0/3/5_-_Kickstart_Templating.html). I have configured some feature about it in the former script you may have noticed. Then I will create an ansible kickstart template from a standard template. PLease see the script.
```
# please customize the packages file path, script name and kickstart name.
packages=$(sed ':a;N;s/\n/\\n/g;ba' /path/to/file)
script="ansible.sh"
kickstart=ansible.ks
localhost=$(/sbin/ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:")

sudo sed -i 's%America/New_York%Asia/Shanghai%g' /var/lib/cobbler/kickstarts/sample_end.ks
cp /var/lib/cobbler/kickstarts/sample_end.ks /var/lib/cobbler/kickstarts/$kickstart
sudo sed -i "s/^func/$packages/g" /var/lib/cobbler/snippets/func_install_if_enabled
sudo sed -i "/boot notification/i\systemctl set-default multi-user.target" /var/lib/cobbler/kickstarts/$kickstart
sudo sed -i "/boot notification/i\wget http:\/\/$localhost\/cobbler\/pub\/$script" /var/lib/cobbler/kickstarts/$kickstart
sudo sed -i "/boot notification/i\chmod a+x $script" /var/lib/cobbler/kickstarts/$kickstart
sudo sed -i "/boot notification/i\. $script" /var/lib/cobbler/kickstarts/$kickstart
cobbler validateks /var/lib/cobbler/kickstarts/$kickstart
# validate kickstart

cobbler sync
```

At last, start you server. You will see the installation.

## 5. Reinstall.

We can use PXE to reinstall or install [Koan](http://cobbler.github.io/manuals/2.8.0/6_-_Koan.html) for re-installation.
```
# please customize the info
yum -y install epel-release
yum -y install koan
koan --server=cobbler.mydomain.org --display --system=name
koan --server=cobbler.mydomain.org --replace-self --system=name
reboot
```
[Auto-Registration](http://cobbler.github.io/manuals/2.8.0/4/8_-_Auto-Registration.html) is useful when using koan for Cobbler.
