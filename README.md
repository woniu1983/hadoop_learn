# hadoop 集群配置

## 为当前用户增加管理员权限，hadoop为当前用户
### su root  ###切换root用户
### visudo
### 追加行：hadoop ALL=(ALL) ALL ###（当中的间隔为tab， hadoop为当前用户）

## 查看是否使用静态IP
ls /etc/sysconfig/network-scripts/ifcfg-en*
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33  ###（默认情况下是 网络连接名称为 ens33）

TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="fbbde99e-45a3-4ac2-8911-de15a2cf980f"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.37.11"
PREFIX="24"
GATEWAY="192.168.37.2"
DNS1="192.168.37.2"
IPV6_PRIVACY="no"

## 修改本机主机名称  ha11.woniu.com, woniu.com是域名
sudo vi /etc/hostname

## 配置Host
###sudo vi /etc/hosts
192.168.37.11 ha11.woniu.com
192.168.37.22 ha22.woniu.com
192.168.37.33 ha33.woniu.com



