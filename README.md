## 使用Cloudflare中转VTowRay流量

担心 IP 被强？或者不想 IP 被强？是的！使用 Cloudflare 来中转 VTowRay 的 WebSocket 流量就行！由于使用了 Cloudflare 中转，所以强根本不知道背后的 IP 是多少，你可以愉快的玩耍了~

### 提醒
如果你不是使用 移动宽带 的用户，那么使用 Cloudflare 中转的速度相对来说是比较慢的，这个是因为线路的问题，无解。  
**警告警告警告**  
该教程目前写得比较简陋，以后应该会增加详细图文教程  
VTowRay 的 WS + TLS 不是神话，如果你没学会走路就不要急着跑  
大佬。。。你如果是从来没接触过 VTowRay 的人一上来就开玩 WS + TLS  
你真的不怕摔跤吗  
你有解析过域名吗，知道什么是 A 记录吗，会修改 NS 吗。。  
如果不懂，那就先补上这些知识再往下看  
如果实在想玩 WS + TLS，请认认真真看教程  
教程真的写得比较简陋，如果实在折腾不成功，那也很正常的，改天再来  
或者直接放弃

### 这是一个提示
真是无聊，折腾啥啊。[买个搬瓦工 Just My Socks 先凑合用着就可以了](https://justmysocks.xyz/buy-justmysocks/)， **被强自动换 IP，无须担心 IP 被强！** Just My Socks 是搬瓦工出品的代理服务，质量可靠，优质 CN2 GIA 线路，并且支持退款，放心无忧。  
套什么 CF，速度慢到怀疑人生。

### 准备
一个域名，建议使用免费域名  
确保域名已经可以在 Cloudflare 正常使用。  
**在 Cloudflare 的 Overview 选项卡可以查看域名状态，请确保为激活状态，即是： Status: Active**  
怎么 SSH 连接上被强的 IP ? Xshell 在属性那里可以设置代理，或者你可以在一台没有被强的境外 VPS 使用 iptables 转发数据到被强的机器上，此处不细说了。
![image](https://github.com/Bryceyao/bryceFile/blob/master/work/image/VTworay/cloudflareActive.png)

### 添加域名解析
在 DNS 选项卡那边添加一个 A 记录的域名解析，假设你的域名是 233blog.com，并且想要使用 www.233blog.com 作为翻~强的域名  
那么在 DNS 那里配置，Name 写 www，IPv4 address 写你的 VPS IP，务必把云朵点灰，然后选择 Add Record 来添加解析记录即可  
(如果你已经添加域名解析， **请务必把云朵点灰** ，即是 DNS only)

OK，确保操作没有问题的话，继续
![image](https://github.com/Bryceyao/bryceFile/blob/master/work/image/VTworay/godaddyNameServers.png)
![image](https://github.com/Bryceyao/bryceFile/blob/master/work/image/VTworay/cloudflareDNS1.png)

### 安装 VTowRay
>如果你已经使用本人提供的 VTowRay 一键安装脚本并安装了 VTowRay，那就直接输入 VTowRay config 修改传输协议为 WebSocket + TLS

如果你并没有使用本站提供的 VTowRay 一键安装脚本来安装 VTowRay  
那么现在开始使用吧，最好用的 VTowRay 安装脚本，保证你满意  
使用 root 用户输入下面命令安装或卸载

`bash <(curl -s -L https://git.io/vZray.sh)`  
>如果提示 curl: command not found ，那是因为你的小鸡没装 Curl  
>ubuntu/debian 系统安装 Curl 方法: apt-get update -y && apt-get install curl -y  
>centos 系统安装 Curl 方法: yum update -y && yum install curl -y  
>安装好 curl 之后就能安装脚本了  

之后选择安装，传输协议选择 WebSocket + TLS (即是选择 4 )，VTowRay 端口随便，不要是 80 和 443 即可， **然后输入你的域名，域名解析 Y ，自动配置 TLS 也是 Y** ，其他就默认吧，一路回车。等待安装完成  
如果你的域名没有正确解析，安装会失败，解析相关看上面的 **添加域名解析**

安装完成后会展示 VTowRay 的配置信息，并且会询问是否生成二维码等，不用管它，直接回车

**然后输入 vZray status 查看一下运行状态，请确保 VTowRay 和 Caddy 都在运行**

如果没有问题的话，继续

### 设置 Crypto 和 开启中转
确保 Cloudflare 的 Crypto 选项卡的 SSL 为 Full
**并且请确保 SSL 选项卡有显示 Universal SSL Status Active Certificate 这样的字眼，如果你的 SSL 选项卡没有显示这个，不要急，只是在申请证书，24 小时内可以搞定。**

**然后在 DNS 选项卡那里，把刚才点灰的那个云朵图标，点亮它，一定要点亮一定要点亮一定要点亮**

云朵图标务必为橙色状态，即是 DNS and HTTP proxy(CDN)

![image](https://github.com/Bryceyao/bryceFile/blob/master/work/image/VTworay/cloudflareSSL.png)
![image](https://github.com/Bryceyao/bryceFile/blob/master/work/image/VTworay/cloudflareDNS.png)

### VTowRay 配置信息
很好，现在接下来配置客户端使用
输入 `vZray info` 即可查看 VTowRay 的配置，如果你有使用某些 VTowRay 客户端，可以根据给出的配置的信息来配置使用了。赶紧测试吧

>VTowRay 客户端使用教程：  
>Windows  
>[VTowRayN使用教程](https://github.com/233boy/v2ray/wiki/V2RayN%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)

什么鬼？对啊，就是如此简单啊，要不然你以为啊。

### 快速管理
vZray info 查看 VTowRay 配置信息  
vZray config 修改 VTowRay 配置  
vZray link 生成 VTowRay 配置文件链接  
vZray infolink 生成 VTowRay 配置信息链接  
vZray qr 生成 VTowRay 配置二维码链接  
vZray ss 修改 ss 配置  
vZray ssinfo 查看 ss 配置信息  
vZray ssqr 生成 ss 配置二维码链接  
vZray status 查看 VTowRay 运行状态  
vZray start 启动 VTowRay  
vZray stop 停止 VTowRay  
vZray restart 重启 VTowRay  
vZray log 查看 VTowRay 运行日志  
vZray update 更新 VTowRay  
vZray update.sh 更新 VTowRay 管理脚本  
vZray uninstall 卸载 VTowRay  

### 配置文件路径
VTowRay 配置文件路径：/etc/vZray/config.json  
Caddy 配置文件路径：/etc/caddy/Caddyfile  
脚本配置文件路径: /etc/vZray/233blog_vZray_backup.conf  
>警告，请不要修改脚本配置文件，免得出错。。  
>如果你不是有特别的需求，也不要修改 VTowRay 配置文件  
>不过也没事，若你实在想要瞎折腾，出错了的话，你就卸载，然后重装，再出错 ，再卸载，再重装，重复到自己不再想折腾为止。。

### 备注
如果你的 VPS 位置是在美国西海岸的话，速度应该还算可以吧，如果不是在美国西海岸，那么也许速度会很慢，不过好在不用担心 IP 被强或者能让被强的 IP 重生也挺好的。难道不是么？
如果你使用移动网络的话，那么 Cloudflare 的中转节点可能会在香港，速度也许会不错 (不完全保证)。

### 无限域名备用
懒得写了，自己悟吧…  
反正绝大多数人只要知道怎么把强的 IP 救活就行…  
算啦，我还是提示一下吧，WebSocket 协议，80 端口，Cloudflare 的 Crypto 选项卡 SSL 为 Flexible  
如果没有太多必要，不需要折腾这

### 结束
哇，没有图文教程你就看不懂的话，我能怎么办，我也很绝望，我更加迷茫

### 原文链接
[使用Cloudflare中转VTowRay流量](https://github.com/233boy/v2ray/wiki/%E4%BD%BF%E7%94%A8Cloudflare%E4%B8%AD%E8%BD%ACV2Ray%E6%B5%81%E9%87%8F)  
[VTowRay一键安装脚本](https://github.com/233boy/v2ray/wiki/V2Ray%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC)

`git clone https://github.com/Bryceyao/vTwoRay -b master
cd v2ray
chmod +x install.sh
./install.sh local`
