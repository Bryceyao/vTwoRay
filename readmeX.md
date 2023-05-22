# 准备工作
> 获取一台 VPS  安装系统    Ubuntu 20.04 x86_64 
 
> 注册域名 https://www.godaddy.com/zh-sg  <br/>

> 第一次更新 Linux 的软件 <br/>
`apt update  ` <br/>
`apt upgrade`

> 修改远程登录端口 <br/>
`vi /etc/ssh/sshd_config` <br/>
`systemctl restart ssh` <br/>

> 可选 安装sudo功能 <br/>
`apt update && apt install sudo` <br/>


# 网站建设篇

## 登录 VPS、安装运行 Nginx <br/>
> `sudo apt update && sudo apt install nginx` <br/>

> 登录ip http://100.200.300.400:80  能看到Welcome to nginx! <br/>


## 创建简单网页
> 配置域名a类型指向VPS地址 <br/>
如 macwork.dbhappy.cn 100.200.300.400 ，通过ping命令，测试是否正确连接 <br/>
`ping macwork.dbhappy.cn`

> 创建一个网站专用的文件夹和index首页 
`mkdir -p /happy/webmacwork/ && vi /happy/webmacwork/index.html`

```
<html lang="">
  <!-- Text between angle brackets is an HTML tag and is not displayed.
        Most tags, such as the HTML and /HTML tags that surround the contents of
        a page, come in pairs; some tags, like HR, for a horizontal rule, stand
        alone. Comments, such as the text you're reading, are not displayed when
        the Web page is shown. The information between the HEAD and /HEAD tags is
        not displayed. The information between the BODY and /BODY tags is displayed.-->
  <head>
    <title>Enter a title, displayed at the top of the window.</title>
  </head>
  <!-- The information between the BODY and /BODY tags is displayed.-->
  <body>
    <h1>Enter the main heading, usually the same as the title.</h1>
    <p>Be <b>bold</b> in stating your key points. Put them in a list:</p>
    <ul>
      <li>The first item in your list</li>
      <li>The second item; <i>italicize</i> key words</li>
    </ul>
    <p>Improve your image by including an image.</p>
    <p>
      <img src="https://i.imgur.com/SEBww.jpg" alt="A Great HTML Resource" />
    </p>
    <p>
      Add a link to your favorite
      <a href="https://www.dummies.com/">Web site</a>. Break up your page
      with a horizontal rule or two.
    </p>
    <hr />
    <p>
      Finally, link to <a href="page2.html">another page</a> in your own Web
      site.
    </p>
    <!-- And add a copyright notice.-->
    <p>&#169; Wiley Publishing, 2011</p>
  </body>
</html>
```
> 编辑Nginx配置文件,server节点可配置多个 <br/>
`vi /etc/nginx/nginx.conf`

```
server {
                listen 80;
                server_name macwork.dbhappy.cn;
                root /happy/webmacwork;
                index index.html;
        }
```

> 让 nginx 重新载入配置使其生效 <br/>

`sudo systemctl reload nginx`

> 访问 http://macwork.dbhappy.cn, 显示index网页

# 证书管理
申请TLS 证书
## 安装 acme.sh
运行安装脚本 <br/>
`wget -O -  https://get.acme.sh | sh`
让 acme.sh 命令生效 <br/>
`. .bashrc`
开启 acme.sh 的自动升级 <br/>
`acme.sh --upgrade --auto-upgrade`

## 测试证书申请
`acme.sh --issue --server letsencrypt --test -d macwork.dbhappy.cn -w /happy/webmacwork --keylength ec-256` <br/>

会提示Cert success，并输出一大段 --BEGIN CERTIFICAT--  --END CERTIFICAT-- <br/>
如果出错，运行调试命令 <br/>
`acme.sh --issue --server letsencrypt --test -d macwork.dbhappy.cn -w /happy/webmacwork --keylength ec-256 --debug` <br/>

## 正式证书申请

`acme.sh --set-default-ca --server letsencrypt` <br/>
`acme.sh --issue --server letsencrypt -d macwork.dbhappy.cn -w /happy/webmacwork --keylength ec-256 --force` <br/>

## 证书安装
`mkdir /happy/cert` <br/>
`acme.sh --installcert -d macwork.dbhappy.cn --cert-file /happy/cert/maccert.crt --key-file /happy/cert/maccert.key --fullchain-file /happy/cert/macfullchain.crt --ecc` <br/>

提示Installing cert to: /happy/cert/maccert.crt 共三条记录代表安装成功 <br/>

xray.key文件默认对其他用户不可读，所以需要赋予其可读性权限 <br/>
`chmod +r /happy/cert/maccert.key`

# Xray 服务器

##  安装 Xray
`wget https://github.com/XTLS/Xray-install/raw/main/install-release.sh` <br/>
`sudo bash install-release.sh` <br/>
`rm ~/install-release.sh` <br/>

## 创建日志文件
建立日志文件  <br/>
`mkdir /happy/xray_log`  <br/>
`touch /happy/xray_log/access.log && touch /happy/xray_log/error.log`  <br/>

因为 Xray 默认是 nobody 用户使用，所以我们需要让其他用户也有“写”的权限  <br/>
`chmod a+w /happy/xray_log/*.log`

## 配置Xray 
生成uuid `xray uuid` <br/>
`vi /usr/local/etc/xray/config.json` <br/>

```
// REFERENCE:
// https://github.com/XTLS/Xray-examples
// https://xtls.github.io/config/
// 常用的 config 文件，不论服务器端还是客户端，都有 5 个部分。外加小小白解读：
// ┌─ 1*log 日志设置 - 日志写什么，写哪里（出错时有据可查）
// ├─ 2_dns DNS-设置 - DNS 怎么查（防 DNS 污染、防偷窥、避免国内外站匹配到国外服务器等）
// ├─ 3_routing 分流设置 - 流量怎么分类处理（是否过滤广告、是否国内外分流）
// ├─ 4_inbounds 入站设置 - 什么流量可以流入 Xray
// └─ 5_outbounds 出站设置 - 流出 Xray 的流量往哪里去
{
  // 1\_日志设置
  "log": {
    "loglevel": "warning", // 内容从少到多: "none", "error", "warning", "info", "debug"
    "access": "/happy/xray_log/access.log", // 访问记录
    "error": "/happy/xray_log/error.log" // 错误记录
  },
  // 2_DNS 设置
  "dns": {
    "servers": [
      "https+local://1.1.1.1/dns-query", // 首选 1.1.1.1 的 DoH 查询，牺牲速度但可防止 ISP 偷窥
      "localhost"
    ]
  },
  // 3*分流设置
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      // 3.1 防止服务器本地流转问题：如内网被攻击或滥用、错误的本地回环等
      {
        "type": "field",
        "ip": [
          "geoip:private" // 分流条件：geoip 文件内，名为"private"的规则（本地）
        ],
        "outboundTag": "block" // 分流策略：交给出站"block"处理（黑洞屏蔽）
      },
      {
        // 3.2 防止服务器直连国内
        "type": "field",
        "ip": ["geoip:cn"],
        "outboundTag": "block"
      },
      // 3.3 屏蔽广告
      {
        "type": "field",
        "domain": [
          "geosite:category-ads-all" // 分流条件：geosite 文件内，名为"category-ads-all"的规则（各种广告域名）
        ],
        "outboundTag": "block" // 分流策略：交给出站"block"处理（黑洞屏蔽）
      }
    ]
  },
  // 4*入站设置
  // 4.1 这里只写了一个最简单的 vless+xtls 的入站，因为这是 Xray 最强大的模式。如有其他需要，请根据模版自行添加。
  "inbounds": [
    //macbook，mac版本只支持xtls-rprx-direct+ xtls模式
    {
      "port": 411,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "xray uuid 生成内容", // 填写你的 UUID
            "flow": "xtls-rprx-direct",
            "level": 1,
            "email": "vpsin@yourdomain.com"
          }
        ],
        "decryption": "none",
        "fallbacks": [
          {
            "dest": 80 // 默认回落到防探测的代理
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "xtls",
        "xtlsSettings": {
          "alpn": "http/1.1",
          "certificates": [
            {
              "certificateFile": "/happy/cert/macfullchain.crt",
              "keyFile": "/happy/cert/maccert.key"
            }
          ]
        }
      }
    },

    //手机，安卓版本只支持xtls-rprx-vision + tls模式
    {
      "port": 412,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "xray uuid 生成内容", // 填写你的 UUID
            "flow": "xtls-rprx-vision",
            "level": 1,
            "email": "vpsin@yourdomain.com"
          }
        ],
        "decryption": "none",
        "fallbacks": [
          {
            "dest": 80 // 默认回落到防探测的代理
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "alpn": "http/1.1",
          "certificates": [
            {
              "certificateFile": "/happy/cert/matefullchain.crt",
              "keyFile": "/happy/cert/matecert.key"
            }
          ]
        }
      }
    }
  ],
  // 5*出站设置
  "outbounds": [
    // 5.1 第一个出站是默认规则，freedom 就是对外直连（vps 已经是外网，所以直连）
    {
      "tag": "direct",
      "protocol": "freedom"
    },
    // 5.2 屏蔽规则，blackhole 协议就是把流量导入到黑洞里（屏蔽）
    {
      "tag": "block",
      "protocol": "blackhole"
    }
  ]
}
```

## 启动 Xray 服务！！（并查看服务状态）
`sudo systemctl start xray` <br/>

`sudo systemctl status xray` <br/>
显示 active (running) ，说明 Xray 已经在正确的运行了

`sudo systemctl restart xray`

## 另参考vmess配置文件
```
{
  "inbounds": [{
    "port": 2222,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "cat /proc/sys/kernel/random/uuid",
          "level": 1,
          "alterId": 122
        }
      ]
    }
  },
  {
    "port": 81,
    "protocol": "vmess",
    "settings": {
      "clients": [
       //pending
       {"id": "cat /proc/sys/kernel/random/uuid", "level": 1,"alterId": 111},
       //work
       {"id": "cat /proc/sys/kernel/random/uuid", "level": 1,"alterId": 222},
       //personal
       {"id": "cat /proc/sys/kernel/random/uuid", "level": 1,"alterId": 333},
       //kaili
       {"id": "cat /proc/sys/kernel/random/uuid", "level": 1,"alterId": 444},
       //pending
       {"id": "cat /proc/sys/kernel/random/uuid", "level": 1,"alterId": 555},       
       //zhuhuiming
       {"id": "cat /proc/sys/kernel/random/uuid", "level": 1,"alterId": 666} 
      ]
    },
    "streamSettings":{
     "wsSettings":{
       "path":"/happy",
       "headers":{}
     },
     "network":"ws"
   }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

# 相关链接

参考教程 https://xtls.github.io/document/level-0/ch01-preface.html <br/>
参考教程 https://v2xtls.org/xray%E6%95%99%E7%A8%8B/ <br/>
客户端 https://itlanyan.com/v2ray-clients-download/ <br/>
客户端 https://www.xiaoglt.top/v2ray%e5%ae%a2%e6%88%b7%e7%ab%af%e5%85%a8%e9%9b%86/ <br/>
