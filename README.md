# server-config
Typical linux server config (CentOS 6.x)

* Max out the file I/O by adding the following to `/etc/security/limit.conf`

```
*    - nofile 65536
```

* Change SELinux to from `enforcing` to `permissive` in `/etc/selinux/config` (Note: this lower the security but much eaiser to sysadmin)
```
SELINUX=permissive
```

* Max out the network I/O by adding the following to `/etc/sysctl.conf` the run `sysctl -p /etc/sysctl.conf` to update the settings
```
# Increase system IP port limits to allow for more connections
net.ipv4.ip_local_port_range = 2000 65000
net.ipv4.tcp_window_scaling = 1

# Increase File I/O
fs.file-max=65536

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
```

* Set server localtime to HKT (or other location as wish)
```
sudo cp /usr/share/zoneinfo/Hongkong /etc/localtime
```

* Flush PageCache once awhile to free up server memory
```
sync; echo 1 > /proc/sys/vm/drop_caches
```
Measure flush PageCache by Cronjob

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

