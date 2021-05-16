# 确认网口

要配置eth1网卡 `ethtool -p eth1`

这时eth1对应的串口会有橘光闪烁，这样就能确定配置了

# 配置临时IP

#### centos：

###### 增加网卡eth1的ip

`ip addr add 192.168.11.190/24 dev eth1`

###### 增加网卡ip经过的路由

`ip route add default via 192.168.11.254 dev eth1`

#### ubuntu：

//adding  later

# 配置静态IP

#### centos:

###### ip a查看网卡名称，网卡名称为enp3s0 

###### vim /etc/sysconfig/network-scripts/ifcfg-enp3s0

*   修改BOOTPROTO=static

* 修改ONBOOT=yes
* 追加以下内容后保存退出
```shell
IPADDR=192.168.3.11         # 要配置的地址
NETMASK=255.255.255.0       # 不是网管之类的，应该都是这个
GATEWAY=192.168.3.254       # 一般是配置地址的最后一段改为254
DNS1=114.114.114.114        # DNS只有连接外网才用到
DNS2=8.8.8.8
```
###### service network restart

ping www.baidu.com测试连通性

###### 执行 yum install net-tools（非必选）

安装后就能使用ifconfig工具

#### ubuntu：

​	//adding later