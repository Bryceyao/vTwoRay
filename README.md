##使用Cloudflare中转V2Ray流量

担心 IP 被墙？或者不想 IP 被墙？是的！使用 Cloudflare 来中转 V2Ray 的 WebSocket 流量就行！由于使用了 Cloudflare 中转，所以墙根本不知道背后的 IP 是多少，你可以愉快的玩耍了~

###提醒
如果你不是使用 移动宽带 的用户，那么使用 Cloudflare 中转的速度相对来说是比较慢的，这个是因为线路的问题，无解。  
**警告警告警告**  
该教程目前写得比较简陋，以后应该会增加详细图文教程  
V2Ray 的 WS + TLS 不是神话，如果你没学会走路就不要急着跑  
大佬。。。你如果是从来没接触过 V2Ray 的人一上来就开玩 WS + TLS  
你真的不怕摔跤吗  
你有解析过域名吗，知道什么是 A 记录吗，会修改 NS 吗。。  
如果不懂，那就先补上这些知识再往下看  
如果实在想玩 WS + TLS，请认认真真看教程  
教程真的写得比较简陋，如果实在折腾不成功，那也很正常的，改天再来  
或者直接放弃

###这是一个提示
真是无聊，折腾啥啊。[买个搬瓦工 Just My Socks 先凑合用着就可以了](https://justmysocks.xyz/buy-justmysocks/)， **被墙自动换 IP，无须担心 IP 被墙！** Just My Socks 是搬瓦工出品的代理服务，质量可靠，优质 CN2 GIA 线路，并且支持退款，放心无忧。  
套什么 CF，速度慢到怀疑人生。

###准备
一个域名，建议使用免费域名  
确保域名已经可以在 Cloudflare 正常使用。  
**在 Cloudflare 的 Overview 选项卡可以查看域名状态，请确保为激活状态，即是： Status: Active**  
怎么 SSH 连接上被墙的 IP ? Xshell 在属性那里可以设置代理，或者你可以在一台没有被墙的境外 VPS 使用 iptables 转发数据到被墙的机器上，此处不细说了。

###添加域名解析
在 DNS 选项卡那边添加一个 A 记录的域名解析，假设你的域名是 233blog.com，并且想要使用 www.233blog.com 作为翻~墙的域名  
那么在 DNS 那里配置，Name 写 www，IPv4 address 写你的 VPS IP，务必把云朵点灰，然后选择 Add Record 来添加解析记录即可  
(如果你已经添加域名解析， **请务必把云朵点灰** ，即是 DNS only)

OK，确保操作没有问题的话，继续

###安装 V2Ray
>如果你已经使用本人提供的 V2Ray 一键安装脚本并安装了 V2Ray，那就直接输入 v2ray config 修改传输协议为 WebSocket + TLS

如果你并没有使用本站提供的 V2Ray 一键安装脚本来安装 V2Ray  
那么现在开始使用吧，最好用的 V2Ray 安装脚本，保证你满意  
使用 root 用户输入下面命令安装或卸载

`bash <(curl -s -L https://git.io/v2ray.sh)`  
>如果提示 curl: command not found ，那是因为你的小鸡没装 Curl  
>ubuntu/debian 系统安装 Curl 方法: apt-get update -y && apt-get install curl -y  
>centos 系统安装 Curl 方法: yum update -y && yum install curl -y  
>安装好 curl 之后就能安装脚本了  

之后选择安装，传输协议选择 WebSocket + TLS (即是选择 4 )，V2Ray 端口随便，不要是 80 和 443 即可， **然后输入你的域名，域名解析 Y ，自动配置 TLS 也是 Y** ，其他就默认吧，一路回车。等待安装完成  
如果你的域名没有正确解析，安装会失败，解析相关看上面的 **添加域名解析**

安装完成后会展示 V2Ray 的配置信息，并且会询问是否生成二维码等，不用管它，直接回车

**然后输入 v2ray status 查看一下运行状态，请确保 V2Ray 和 Caddy 都在运行**

如果没有问题的话，继续

###设置 Crypto 和 开启中转
确保 Cloudflare 的 Crypto 选项卡的 SSL 为 Full
**并且请确保 SSL 选项卡有显示 Universal SSL Status Active Certificate 这样的字眼，如果你的 SSL 选项卡没有显示这个，不要急，只是在申请证书，24 小时内可以搞定。**

**然后在 DNS 选项卡那里，把刚才点灰的那个云朵图标，点亮它，一定要点亮一定要点亮一定要点亮**

云朵图标务必为橙色状态，即是 DNS and HTTP proxy(CDN)

###V2Ray 配置信息
很好，现在接下来配置客户端使用
输入 `v2ray info` 即可查看 V2Ray 的配置，如果你有使用某些 V2Ray 客户端，可以根据给出的配置的信息来配置使用了。赶紧测试吧

>V2Ray 客户端使用教程：  
>Windows  
>[V2RayN使用教程](https://github.com/233boy/v2ray/wiki/V2RayN%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)

什么鬼？对啊，就是如此简单啊，要不然你以为啊。

###备注
如果你的 VPS 位置是在美国西海岸的话，速度应该还算可以吧，如果不是在美国西海岸，那么也许速度会很慢，不过好在不用担心 IP 被墙或者能让被墙的 IP 重生也挺好的。难道不是么？
如果你使用移动网络的话，那么 Cloudflare 的中转节点可能会在香港，速度也许会不错 (不完全保证)。

###无限域名备用
懒得写了，自己悟吧…  
反正绝大多数人只要知道怎么把墙的 IP 救活就行…  
算啦，我还是提示一下吧，WebSocket 协议，80 端口，Cloudflare 的 Crypto 选项卡 SSL 为 Flexible  
如果没有太多必要，不需要折腾这

###结束
哇，没有图文教程你就看不懂的话，我能怎么办，我也很绝望，我更加迷茫
