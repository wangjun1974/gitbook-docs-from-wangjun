# 如何与本地时间源同步

注意：由于overcloud采用ntpd与ntpserver进行时间同步，因此需要配置ntpd兼容的时间服务器，而不能配置chronyd时间服务器。

## 安装ntp软件
```
yum install -y ntp
```

## 在director上修改配置文件/etc/ntp.conf内容如下
```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
restrict 172.16.0.0 mask 255.255.255.0 nomodify notrap
server 127.127.1.0 iburst
fudge  127.127.1.0 stratum 8
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

## 启用ntpd服务
```
systemctl enable ntpd
systemctl start ntpd
```

（可选）如果启用了chronyd服务，请禁用并停止chronyd服务
```
systemctl disable chronyd
systemctl stop chronyd
```

## 确认防火墙包含允许ntp服务的规则
```
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            multiport dports 123 /* 105 ntp */ state NEW
```

## How to sync chrony to the local clock
[How to sync chrony to the local clock](https://www.thegeekdiary.com/centos-rhel-7-chrony-how-to-sync-to-local-clock/)

Question : How to sync chrony to the local clock.

Answer :
When the chrony service starts, there are some settings in the /etc/chrony/chrony.conf file that tells it to actually set the time if specific conditions occur. Below procedure lts you set the local clock as the source for chrony to synchronize the time.

1. Currently the chrony does not sync to local clock and ‘chronyc sources’ command gives the following result 
```
# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^? localhost
```

2. Edit /etc/chrony.conf to add the settings below. The configuration file needs at least 4 of the below entries to have a local clock synchronization and for other server on provision-net
```
# vi /etc/chrony.conf
server 127.127.1.0
allow 127.0.0.0/8
allow <provision-net/mask>
local stratum 3
```

3. Restart chronyd service
```
# systemctl restart chronyd.service
```
4. Verify the status of chrony synchronization
```
# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx     Last         sample
========================================================================
^* 127.127.1.0                  15       6   377    42      -4471ns    [-13us] +/-  204us
```
