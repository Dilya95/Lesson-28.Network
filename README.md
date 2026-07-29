# Домашнее задание 28: 

## Задания
Дано<br>
Скачать и развернуть Vagrant-стенд 
https://github.com/erlong15/otus-linux/tree/network
(ветка network)<br>

Построить следующую сетевую архитектуру:<br>

Сеть office1<br>

192.168.2.0/26 - dev<br>
192.168.2.64/26 - test servers<br>
192.168.2.128/26 - managers<br>
192.168.2.192/26 - office hardware<br>


Сеть office2<br>

192.168.1.0/25 - dev<br>
192.168.1.128/26 - test servers<br>
192.168.1.192/26 - office hardware<br>


Сеть central<br>

192.168.0.0/28 - directors<br>
192.168.0.32/28 - office hardware<br>
192.168.0.64/26 - wifi<br>

```
Office1 ---\
                   -----> Central --IRouter --> internet                
Office2----/
```
Итого должны получится следующие сервера<br>

inetRouter<br>
centralRouter<br>
office1Router<br>
office2Router<br>
centralServer<br>
office1Server<br>
office2Server<br>

Теоретическая часть<br>

Найти свободные подсети<br>
Посчитать сколько узлов в каждой подсети, включая свободные<br>
Указать broadcast адрес для каждой подсети<br>
проверить нет ли ошибок при разбиении<br>

Практическая часть<br>
Соединить офисы в сеть согласно схеме и настроить роутинг<br>
Все сервера и роутеры должны ходить в инет черз inetRouter<br>
Все сервера должны видеть друг друга<br>
у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи<br>
при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс<br>


Формат сдачи ДЗ - vagrant + ansible


## Структура
├── README.md<br>
└── Vagrantfile.

## Теоретическая часть<br>

### Таблица подсетей

| Name                  | Network            | Netmask         | Hosts (N) | Hostmin       | Hostmax       | Broadcast      |
|-----------------------|--------------------|-----------------|-----------|---------------|---------------|----------------|
| **Central**           |                    |                 |           |               |               |                |
| Directors             | 192.168.0.0/28     | 255.255.255.240 | 14        | 192.168.0.1   | 192.168.0.14  | 192.168.0.15   |
| Office hardware       | 192.168.0.32/28    | 255.255.255.240 | 14        | 192.168.0.33  | 192.168.0.46  | 192.168.0.47   |
| WiFi (mgt)            | 192.168.0.64/26    | 255.255.255.192 | 62        | 192.168.0.65  | 192.168.0.126 | 192.168.0.127  |
| **Office1**           |                    |                 |           |               |               |                |
| Dev                   | 192.168.2.0/26     | 255.255.255.192 | 62        | 192.168.2.1   | 192.168.2.62  | 192.168.2.63   |
| Test servers          | 192.168.2.64/26    | 255.255.255.192 | 62        | 192.168.2.65  | 192.168.2.126 | 192.168.2.127  |
| Managers              | 192.168.2.128/26   | 255.255.255.192 | 62        | 192.168.2.129 | 192.168.2.190 | 192.168.2.191  |
| Office hardware       | 192.168.2.192/26   | 255.255.255.192 | 62        | 192.168.2.193 | 192.168.2.254 | 192.168.2.255  |
| **Office2**           |                    |                 |           |               |               |                |
| Dev                   | 192.168.1.0/25     | 255.255.255.128 | 126       | 192.168.1.1   | 192.168.1.126 | 192.168.1.127  |
| Test servers          | 192.168.1.128/26   | 255.255.255.192 | 62        | 192.168.1.129 | 192.168.1.190 | 192.168.1.191  |
| Office hardware       | 192.168.1.192/26   | 255.255.255.192 | 62        | 192.168.1.193 | 192.168.1.254 | 192.168.1.255  |
| **Inet**              |                    |                 |           |               |               |                |
| inet ↔ central        | 192.168.255.0/30   | 255.255.255.252 | 2         | 192.168.255.1 | 192.168.255.2 | 192.168.255.3  |
| central ↔ office1     | 192.168.255.8/30   | 255.255.255.252 | 2         | 192.168.255.9 | 192.168.255.10| 192.168.255.11 |
| central ↔ office2     | 192.168.255.4/30   | 255.255.255.252 | 2         | 192.168.255.5 | 192.168.255.6 | 192.168.255.7  |

После создания таблицы топологии, мы видим, что ошибок в задании нет, также мы сразу видим следующие свободные сети: 

192.168.0.16/28 
192.168.0.48/28
192.168.0.128/25
192.168.255.64/26
192.168.255.32/27
192.168.255.16/28
192.168.255.8/29  
192.168.255.4/30 


## Практическая часть<br>

### Настройка NAT, отключение ufw

```

root@inetRouter:~# systemctl status ufw
● ufw.service - Uncomplicated firewall
     Loaded: loaded (/usr/lib/systemd/system/ufw.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2026-07-28 14:19:21 UTC; 20h ago
 Invocation: e695ccbbc5a74f1c83b5da5439a9d048
       Docs: man:ufw(8)
   Main PID: 924 (code=exited, status=0/SUCCESS)
   Mem peak: 1.9M
        CPU: 63ms

Jul 28 14:19:20 vagrant systemd[1]: Starting ufw.service - Uncomplicated firewall...
Jul 28 14:19:21 vagrant systemd[1]: Finished ufw.service - Uncomplicated firewall.


root@inetRouter:~# systemctl stop ufw


root@inetRouter:~# systemctl disable ufw
Synchronizing state of ufw.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable ufw
Removed '/etc/systemd/system/multi-user.target.wants/ufw.service'.


root@inetRouter:~# systemctl status ufw
○ ufw.service - Uncomplicated firewall
     Loaded: loaded (/usr/lib/systemd/system/ufw.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:ufw(8)

Jul 28 14:19:20 vagrant systemd[1]: Starting ufw.service - Uncomplicated firewall...
Jul 28 14:19:21 vagrant systemd[1]: Finished ufw.service - Uncomplicated firewall.
Jul 29 11:03:19 inetRouter systemd[1]: Stopping ufw.service - Uncomplicated firewall...
Jul 29 11:03:19 inetRouter ufw-init[4923]: Skip stopping firewall: ufw (not enabled)
Jul 29 11:03:19 inetRouter systemd[1]: ufw.service: Deactivated successfully.
Jul 29 11:03:19 inetRouter systemd[1]: Stopped ufw.service - Uncomplicated firewall.


root@inetRouter:~# nano /etc/iptables_rules.ipv4


root@inetRouter:~# cat /etc/iptables_rules.ipv4
# Generated by iptables-save v1.8.7 on Sat Oct 14 16:14:36 2023
*filter
:INPUT ACCEPT [90:8713]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [54:7429]
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
COMMIT
# Completed on Sat Oct 14 16:14:36 2023
# Generated by iptables-save v1.8.7 on Sat Oct 14 16:14:36 2023
*nat
:PREROUTING ACCEPT [1:44]
:INPUT ACCEPT [1:44]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
COMMIT
# Completed on Sat Oct 14 16:14:36 2023


root@inetRouter:~# nano /etc/network/if-pre-up.d/iptables

root@inetRouter:~# cat /etc/network/if-pre-up.d/iptables
#!/bin/sh
/sbin/iptables-restore < /etc/iptables_rules.ipv4

root@inetRouter:~# sudo chmod +x /etc/network/if-pre-up.d/iptables

root@inetRouter:~# iptables-save

root@inetRouter:~# reboot
root@inetRouter:~# Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.


[root@otus-homework otus-linux]# vagrant ssh inetRouter
Welcome to Ubuntu 26.04 LTS (GNU/Linux 7.0.0-22-generic x86_64)

 * Documentation:  https://docs.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Jul 29 10:20:28 AM UTC 2026

  System load:             0.08
  Usage of /:              16.1% of 29.82GB
  Memory usage:            29%
  Swap usage:              0%
  Processes:               105
  Users logged in:         0
  IPv4 address for enp0s3: 10.0.2.15
  IPv6 address for enp0s3: fd17:625c:f037:2:a00:27ff:fe0a:4e27


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
Last login: Wed Jul 29 10:20:35 2026 from 10.0.2.2

vagrant@inetRouter:~$ sudo -i

root@inetRouter:~# iptables-save

root@inetRouter:~# cat /etc/iptables_rules.ipv4
# Generated by iptables-save v1.8.7 on Sat Oct 14 16:14:36 2023
*filter
:INPUT ACCEPT [90:8713]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [54:7429]
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
COMMIT
# Completed on Sat Oct 14 16:14:36 2023
# Generated by iptables-save v1.8.7 on Sat Oct 14 16:14:36 2023
*nat
:PREROUTING ACCEPT [1:44]
:INPUT ACCEPT [1:44]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
COMMIT
# Completed on Sat Oct 14 16:14:36 2023


root@inetRouter:~# iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         


root@inetRouter:/etc/network/if-pre-up.d# nano /etc/iptables_rules.ipv4

root@inetRouter:/etc/network/if-pre-up.d# iptables-restore < /etc/iptables_rules.ipv4

root@inetRouter:/etc/network/if-pre-up.d# iptables-save
# Generated by iptables-save v1.8.11 (nf_tables) on Wed Jul 29 14:30:13 2026
*filter
:INPUT ACCEPT [17:896]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [11:884]
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
COMMIT
# Completed on Wed Jul 29 14:30:13 2026
# Generated by iptables-save v1.8.11 (nf_tables) on Wed Jul 29 14:30:13 2026
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o enp0s3 -j MASQUERADE
COMMIT
# Completed on Wed Jul 29 14:30:13 2026
```


### Маршрутизация транзитных пакетов (IP forward)

```

root@inetRouter:# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0

root@inetRouter:# echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
net.ipv4.conf.all.forwarding = 1

root@inetRouter:# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
 



root@centralRouter:~# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0

root@centralRouter:~# echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
net.ipv4.conf.all.forwarding = 1

root@centralRouter:~# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1



root@office1Router:~# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0

root@office1Router:~# echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
net.ipv4.conf.all.forwarding = 1

root@office1Router:~# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1



root@office2Router:~# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0

root@office2Router:~# echo "net.ipv4.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
net.ipv4.conf.all.forwarding = 1

root@office2Router:~# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

### Отключение маршрута по умолчанию на интерфейсе enp0s3
```

root@centralRouter:~# nano /etc/netplan/00-installer-config.yaml

root@centralRouter:~# cat /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
      dhcp4-overrides:
          use-routes: false
      dhcp-identifier: mac
      dhcp6: true
      dhcp6-overrides:
          use-routes: false
      match:
        macaddress: 08:00:27:0a:4e:27
      set-name: enp0s3
  version: 2

root@centralRouter:~# sudo chmod 600 /etc/netplan/01-netcfg.yaml /etc/netplan/50-vagrant.yaml

root@centralRouter:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 113 seconds
Configuration accepted.

root@centralRouter:~# ip r
2.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
45.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
45.144.48.253 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
192.168.0.0/28 dev eth2 proto kernel scope link src 192.168.0.1 
192.168.0.32/28 dev eth3 proto kernel scope link src 192.168.0.33 
192.168.0.64/26 dev eth4 proto kernel scope link src 192.168.0.65 
192.168.50.0/24 dev eth7 proto kernel scope link src 192.168.50.11 
192.168.255.0/30 dev eth1 proto kernel scope link src 192.168.255.2 
192.168.255.4/30 dev eth6 proto kernel scope link src 192.168.255.5 
192.168.255.8/30 dev eth5 proto kernel scope link src 192.168.255.9 




root@centralServer:~# nano /etc/netplan/00-installer-config.yaml

root@centralServer:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 120 seconds
Configuration accepted.





root@office1Router:~# nano /etc/netplan/00-installer-config.yaml

root@office1Router:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 120 seconds
Configuration accepted.


root@office1Server:~# nano /etc/netplan/00-installer-config.yaml

root@office1Server:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 118 seconds
Configuration accepted.



root@office2Router:~# nano /etc/netplan/00-installer-config.yaml

root@office2Router:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 120 seconds
Configuration accepted.


root@office2Server:~# nano /etc/netplan/00-installer-config.yaml

root@office2Server:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 117 seconds
Configuration accepted.
```


### Настройка статических маршрутов и проверка с traceroute
```
root@inetRouter:# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:
      addresses:
        - 192.168.255.1/30
      routes:
        - to: 192.168.0.0/24
          via: 192.168.255.2
        - to: 192.168.1.0/24
          via: 192.168.255.2
        - to: 192.168.2.0/24
          via: 192.168.255.2
    eth2:
      addresses:
        - 192.168.50.10/24


root@inetRouter:/# ip r
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
2.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
45.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
45.144.48.253 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
192.168.0.0/24 via 192.168.255.2 dev eth1 proto static 
192.168.1.0/24 via 192.168.255.2 dev eth1 proto static 
192.168.2.0/24 via 192.168.255.2 dev eth1 proto static 
192.168.50.0/24 dev eth2 proto kernel scope link src 192.168.50.10 
192.168.255.0/30 dev eth1 proto kernel scope link src 192.168.255.1 






root@office1Server:~# ip r
2.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
45.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
45.144.48.253 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
192.168.2.128/26 dev eth1 proto kernel scope link src 192.168.2.130 
192.168.50.0/24 dev eth2 proto kernel scope link src 192.168.50.21 

root@office1Server:~# ip route add 0.0.0.0/0 via 192.168.2.129

root@office1Server:~# ip r
default via 192.168.2.129 dev eth1 
2.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
45.144.48.252 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
45.144.48.253 via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
192.168.2.128/26 dev eth1 proto kernel scope link src 192.168.2.130 
192.168.50.0/24 dev eth2 proto kernel scope link src 192.168.50.21 


root@office1Server:~# nano /etc/netplan/50-vagrant.yaml

root@office1Server:~# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:
      addresses:
      - 192.168.2.130/26
      routes:
      - to: 0.0.0.0/0
        via: 192.168.2.129
    eth2:
      addresses:
      - 192.168.50.21/24

root@office1Server:~# netplan try
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 120 seconds
Configuration accepted.


root@office1Server:~# traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (192.168.2.129)  5.031 ms  4.849 ms  10.638 ms
 2  192.168.255.9 (192.168.255.9)  25.907 ms  25.825 ms  25.751 ms
 3  192.168.255.1 (192.168.255.1)  26.323 ms  26.242 ms  27.497 ms
 4  10.0.2.2 (10.0.2.2)  30.008 ms  29.939 ms  32.361 ms
 5  45.144.51.1 (45.144.51.1)  37.952 ms  37.885 ms  36.203 ms
 6  45.144.48.245 (45.144.48.245)  35.070 ms 45.144.48.244 (45.144.48.244)  32.473 ms  28.706 ms
 7  waw-lim-b1-ae1-vlan3660.fiord.net (80.77.167.200)  19.993 ms  19.815 ms 45.144.48.245 (45.144.48.245)  28.102 ms
 8  * waw-lim-b1-ae1-vlan3660.fiord.net (80.77.167.200)  28.639 ms  28.581 ms
 9  * * *
10  142.250.236.117 (142.250.236.117)  46.354 ms 142.251.254.215 (142.251.254.215)  52.389 ms  52.337 ms
11  108.170.233.41 (108.170.233.41)  52.287 ms 192.178.44.87 (192.178.44.87)  52.235 ms 142.250.239.185 (142.250.239.185)  43.702 ms
12  172.253.72.123 (172.253.72.123)  43.491 ms dns.google (8.8.8.8)  38.928 ms 142.251.65.83 (142.251.65.83)  38.797 ms




root@office1Router:~# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:   # 192.168.255.10/30 → centralRouter
      addresses:
        - 192.168.255.10/30
      routes:
        - to: 0.0.0.0/0
          via: 192.168.255.9
        - to: 192.168.0.0/24
          via: 192.168.255.9
        - to: 192.168.1.0/24
          via: 192.168.255.9
    eth2:   # 192.168.2.1/26 dev
      addresses:
        - 192.168.2.1/26
    eth3:   # 192.168.2.65/26 test
      addresses:
        - 192.168.2.65/26
    eth4:   # 192.168.2.129/26 managers
      addresses:
        - 192.168.2.129/26
    eth5:   # 192.168.2.193/26 office hardware
      addresses:
        - 192.168.2.193/26
    eth6:   # 192.168.50.20
      addresses:
        - 192.168.50.20/24





root@centralRouter:~# nano /etc/netplan/50-vagrant.yaml


root@centralRouter:~# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:   # 192.168.255.2/30 → inetRouter
      addresses:
        - 192.168.255.2/30
      routes:
        - to: 0.0.0.0/0
          via: 192.168.255.1
    eth2:   # 192.168.0.1/28 directors
      addresses:
        - 192.168.0.1/28
    eth3:   # 192.168.0.33/28 office hardware
      addresses:
        - 192.168.0.33/28
    eth4:   # 192.168.0.65/26 wifi
      addresses:
        - 192.168.0.65/26
    eth5:   # 192.168.255.9/30 → office1Router
      addresses:
        - 192.168.255.9/30
      routes:
        - to: 192.168.2.0/24
          via: 192.168.255.10
    eth6:   # 192.168.255.5/30 → office2Router
      addresses:
        - 192.168.255.5/30
      routes:
        - to: 192.168.1.0/24
          via: 192.168.255.6
    eth7:   # 192.168.50.11
      addresses:
        - 192.168.50.11/24





root@centralServer:~# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:   # 192.168.0.2/28
      addresses:
        - 192.168.0.2/28
      routes:
        - to: 0.0.0.0/0
          via: 192.168.0.1
    eth2:   # 192.168.50.12
      addresses:
        - 192.168.50.12/24




root@office2Router:~# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:   # 192.168.255.6/30 → centralRouter
      addresses:
        - 192.168.255.6/30
      routes:
        - to: 0.0.0.0/0
          via: 192.168.255.5
        - to: 192.168.0.0/24
          via: 192.168.255.5
        - to: 192.168.2.0/24
          via: 192.168.255.5
    eth2:   # 192.168.1.1/25 dev
      addresses:
        - 192.168.1.1/25
    eth3:   # 192.168.1.129/26 test
      addresses:
        - 192.168.1.129/26
    eth4:   # 192.168.1.193/26 office hardware
      addresses:
        - 192.168.1.193/26
    eth5:   # 192.168.50.30
      addresses:
        - 192.168.50.30/24



root@office2Server:~# cat /etc/netplan/50-vagrant.yaml
---
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:   # 192.168.1.2/25
      addresses:
        - 192.168.1.2/25
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
    eth2:   # 192.168.50.31
      addresses:
        - 192.168.50.31/24        
```



