?> 作者：拂晓，供稿日期：2019/11/15

## 应用场景

智能家庭相关的设备出现后，我们在家中的局域网可以访问这样的一些设备，比如路由器、NAS、网络摄像头等等。由于网络提供商的限制，我们想在外网访问这些设备提供的服务是有一定难度的。

## 什么是 NAS

NAS，也就是Network Attached Storage的缩写，标识网络附属存储。

<img src="https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1264488230,3209445528&fm=26&gp=0.jpg" width="400" style="vertical-align: top;">
<img src="https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=7e26c81fbd096b63951456026d5aec21/b03533fa828ba61edcb3c71f4234970a304e59a4.jpg" height="300">

家庭中使用的私有云存储。现在比较出名的几个品牌：群晖（Synology）、威联通（QNAP）。

也可以自己去买相关的硬件（攒机）然后刷这样的一些系统进去，比如蜗牛星际、树莓派、电视盒子、软路由。刷黑群晖系统，win10 系统

NAS 可以提供一些文件存储，系统备份(Time Machine)，照片备份浏览编辑(手机照片自动备份)，音频、视频媒体的播放（电视盒子 kodi），web 服务。

NAS 的功耗低，100w 不到，可以实现 7\*24 小时开机。网络资源利用率较高，可以长期挂 BT，PT 下载

<div style="margin-bottom:20px;">
  <img src="./network/house-networking/images/1.jpg" width="280" style="margin-right: 10px;">
  <img src="./network/house-networking/images/2.jpg" width="280" style="margin-right: 10px;">
  <img src="./network/house-networking/images/6.jpg" width="280">
</div>

<div style="margin-bottom:20px;">
  <img src="./network/house-networking/images/3.jpg" width="280">
  <img src="./network/house-networking/images/4.jpg" width="280">
</div>

## 如何可以在外网也能进行访问？

-   网络运营商提供公网 IP 进行绑定（IPv4）（中国电信可以通过向客服申请），将内网 IP 和端口通过 ddns（动态域名解析）映射到公网 IP 上

    -   NAS: 192.168.0.44:5000 -> 39.0.0.1:5000 -> www.abc.com:5000
    -   Router: 192.168.0.1:80 -> 39.0.0.1:5001 -> www.abc.com:5001

-   通过内网穿透

## 内网穿透

### 什么是内网穿透呢？

通过一台有公网 IP 的服务器，代理到自己家中的网络设备上，通过这种方式来访问就叫内网穿透

提供内网穿透的代理工具：花生壳、Ngrok、FRP

### 什么是 FRP

FRP 全名：Fast Reverse Proxy

FRP 是一个使用 Go 语言开发的高性能的反向代理应用，可以轻松地进行内网穿透，对外网提供服务。FRP 支持 TCP、UDP、HTTP、HTTPS 等协议类型，并且支持 Web 服务根据域名进行路由转发。

FRP 的优势：

-   利用处于内网或防火墙后的机器，对外网环境提供 HTTP 或 HTTPS 服务
-   利用处于内网或防火墙后的机器，对外网环境提供 TCP 和 UDP 服务，比如 SSH 访问处于内网的设备

### 内网穿透的前期准备工作

-   需要一台有公网 IP 的服务器（阿里云、亚马逊 VPS、搬瓦工 Bandwagon）
-   家里的一台 7 \* 24 小时开机的设备（NAS、路由器）
-   Frp：https://github.com/fatedier/frp 下载 release 版本，选好家里设备对应的版本

### 配置和运行（最简易形式）

-   将服务端 frps 及 frps.ini 放到具有公网 IP 的机器上
-   将客户端 frpc 及 frpc.ini 放到处于内网环境的机器上
-   注意：
    -   vhost_http_port 和 vhost_https_port 对应配置 vps 上的 http 和 https 服务
    -   服务端和客户端的 common 配置中的 token 参数一致则身份验证通过
    -   subdomain_host 是配置自定义二级域名

1. 配置 frps.ini

```ini
[common]
bind_port = 7000
vhost_http_port = 8081
vhost_https_port = 8082
max_pool_count = 5
token = xxxxx
subdomain_host = abc.com

[router]
listen_port = 5001

[nas]
listen_port = 5000
```

2. 启动 frps

```bash
./frps -c ./frps.ini
```

3. 配置 frpc.ini

```ini
[common]
server_addr = abc.com
server_port = 7000
token = xxxxx

[nas]
type = tcp
local_ip = 192.168.0.44
local_port = 5000
remote_port = 5000

[nas-web]
type = tcp
local_ip = 192.168.0.44
local_port = 80
remote_port = 5001

[nas-ssh]
type = tcp
local_ip = 192.168.0.44
local_port = 22
remote_port = 5022
```

4. 启动 frpc

```bash
./frpc -c ./frpc.ini
```

5. 将域名解析到公网 IP 上
6. 通过 浏览器访问 域名 : 端口 即可访问内网机器上的相应服务

### 内网穿透的一些问题

-   访问速度慢
-   需要外部的云服务器支持，需要额外花钱

## IPv6

### 什么是 IPv6

-   ipv6 是互联网协议的第 6 个版本，地址长度是 128 位，也就是说它的数量可以达到 2^128，大概是 3.4\*10^38 个
-   ipv6 的测试地址：https://www.test-ipv6.com/

    <img src="./network/house-networking/images/5.jpg" width="200">

-   ipv6 地址样例：2409:8962:1:a29d:1d55:7b44:ec6:31xx
-   移动的一般 2409 开头，联通 2408 开头，电信 240e 开头
-   运营商对于 ipv4 只能提供内网 ip，原因是 ipv4 的地址已经枯竭。但是对于 ipv6 的话可以做到每一台设备都能分配一个全球唯一的 ipv6 地址。在去年 3 大运营商就开始响应国务院的号召开始部署 ipv6 的宽带网络，在今年基本上部署完成。

### 如何在家中通过 ipv6 进行外网访问

如果你想通过 ipv6 在外网来访问你家里的设备，必须两端都要支持 ipv6 才可以

-   移动光猫的配置
    -   通过超级管理员登录光猫（账号：CMCCAdmin 密码：aDm8H%MdA）
    -   将我们的千兆网口设置为桥接模式，ip 协议更改为 ipv4 / ipv6 双栈模式
-   路由器配置
    -   家里的路由器里配置 PPPoE 的登录模式，通过运营商给我们的账号和密码（可以在光猫的路由模式下看见的）进行拨号上网。进入到路由器的 ipv6 的设置，看一下是否已经获取到 ipv6 地址，然后设置 ipv6 专用的防火墙（主要是设置端口的开放，如果不太清楚怎么设置可以先选择完全关闭）
-   需要进行外网访问的设备（nas）的配置
    -   这时候路由器已经可以给路由器下的设备分配 ipv6 地址了，进入这些设备看一下 ipv6 地址的分配情况
    -   由于运营商的原因，ipv6 的地址和 ipv4 的地址一样，也会定期进行更新的。所以我们需要通过 DDNS 动态域名解析，定时更新 ip 地址到我们购买的域名上

### 相关链接

https://github.com/fatedier/frp

https://github.com/fatedier/frp/blob/master/README_zh.md

https://www.jianshu.com/p/00c79df1aaf0

https://blog.csdn.net/zhangguo5/article/details/77848658

http://www.gov.cn/zhengce/2017-11/26/content_5242389.htm
