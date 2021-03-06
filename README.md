# server-config
Typical linux server config (CentOS 6.x)

* Max out the file I/O and user process by adding the following to `/etc/security/limits.conf`

```
*                  -       nofile          292144
username           soft    nproc           16384
username           hard    nproc           16384
```

* Change SELinux to from `enforcing` to `permissive` in `/etc/selinux/config` (Note: this lower the security but much eaiser to sysadmin)
```
SELINUX=permissive
```

* Max out the network I/O by adding the following to `/etc/sysctl.conf` the run `sysctl -p /etc/sysctl.conf` to update the settings
```
# Increase system IP port limits to allow for more connections
# net.ipv4.ip_local_port_range = 2000 65000
net.ipv4.tcp_window_scaling = 1

# Increase File I/O
fs.file-max=262144

# number of packets to keep in backlog before the kernel starts dropping them
net.ipv4.tcp_max_syn_backlog = 3240000

# increase socket listen backlog
net.core.somaxconn = 3240000
net.ipv4.tcp_max_tw_buckets = 1440000

# Increase TCP buffer sizes
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_congestion_control = cubic
# net.netfilter.nf_conntrack_max = 262144

# For fluentd
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10240 65535

# Set swappiness to 0 to avoid swapping
vm.swappiness = 0
```

* Set server localtime to HKT (or other location as wish) and enable NTPd
```
sudo vi /etc/sysconfig/clock
```
```
ZONE="Asia/Hong_Kong"
UTC=false
ARC=false
```
```
sudo rm -rf /etc/localtime; 
sudo ln -s /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime; 
sudo hwclock --systohc --localtime;
sudo service ntpd restart;
sudo chkconfig ntpd on;

```

* Flush PageCache once 80% of memory are allocated (added into /etc/cron.d/server)
Create a shellscript file
```
#!/bin/bash
mem=$(free -m | awk 'NR==2{printf "%d", $3*100/$2 }')
if [ $mem -gt 80 ]
then
  sync; echo 1 > /proc/sys/vm/drop_caches
fi
```
To setup cronjob
```
# Check free memory for every minute
*/1 * * * * root /bin/bash _path_to_shellscript_file
```

* Create application-specific users and assign random password for security purpose. These users will then be accessed by `sudo su - xxx`
```
useradd -d /home/xxx -m xxx
```

* Create single user group for the ease of file update
```
groupadd yyy
```

* For each user is assigned to the same user group and allow read and write by that group
```
usermod -g yyy xxx (for each user)
sudo su - xxx (for each user)
vi ~/.bashrc add 'umask 002'
```

* For user who need to login, use SSH with private key (must!) and disable password prompt
```
sudo su - zzz
ssh-keygen -t rsa -b 2048
name of key: key_zzz

mkdir .ssh
touch .ssh/authorized_keys
cat key_zzz.pub >> .ssh/authorized_keys
chmod 700 .ssh
chmod 0640 .ssh/authorized_keys
rm key_zzz.pub
```

