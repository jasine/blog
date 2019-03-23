---
title: mac添加静态路由
---

路由查询： `netstat -rn`，通过该命令可以查看系统的路由表

永久添加静态路由: `networksetup -setadditionalroutes "Wi-Fi" 192.168.2.14 255.255.255.0 10.100.0.9` 参数分别是网络服务名；目标网络；目标网络子网掩码；网关IP， 网络服务名可以通过`networksetup -listallnetworkservices`查询

但是以上方式貌似没法设置网络接口，比如要将路由设置到`VPN`的网络接口时就没法用了，这种情况下可以使用`sudo route add -host 192.168.2.14 -interface utun3` 命令，而这种方法的缺点是重启丢失


