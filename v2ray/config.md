# V2ray配置文件
> A记录是 域名 对应 ipv4 的记录.多条A记录代表dns会逐一分配给请求,达到简单的负载均衡
> AAAA记录是 域名 对应 ipv6 的记录. 
```
{
  "log": {},                  日志
  "api": {},                  远程控制api
  "dns": {},                  dns服务
  "stats": {},                统计信息
  "routing": {},              路由规则
  "policy": {},               本地策略
  "reverse": {},              反向代理
  "inbounds": [],             入站
  "outbounds": [],            出站
  "transport": {}             传输配置
}
```
## log日志
```json
{
  "loglevel": "info"
}
```
## api远程控制api
## dns服务配置
```
{
  "hosts": {                                静态IP列表,值是一系列 "域名":"地址" 地址可以是域名或IP, 如果地址是域名, 后续将会使用该域名继续匹配
    "baidu.com": "127.0.0.1"                域名: 纯字符串;正则表达式"regexp:";子域名"domain:";子串(包含)"keyword:";预定义域名列表
  },
  "servers": [                              dns服务器列表 
    {                                       太复杂 别用了!
      "address": "1.2.3.4",
      "port": 5353,
      "domains": [
        "domain:v2ray.com"
      ],
      "expectIPs": [
        "geoip:cn"
      ],
    },
    "8.8.8.8",                              dns服务器IP 使用53端口 进行DNS查询
    "8.8.4.4",
    "localhost"
  ],
  "clientIp": "1.2.3.4",                    当前系统的IP地址,用于DNS查询时,服务器可知客户端的位置.不能是私有地址   不明所以
  "tag": "dns_inbound"                      此DNS发出的查询流量标签,可以由路由inboundTag进行匹配
}
```
## stats统计信息
## routing路由配置
```
{
  "domainStrategy": "AsIs",
AsIs只用域名进行路由选择,默认   (不使用IP进行匹配)
IPIfNonMatch当域名匹配失败,解析成IP再进行匹配,匹配成功后还以原域名转发   (先使用域名,失败再IP)
IpOnDeamand当碰到基于IP的规则,直接转换成IP进行匹配.     (根据规则顺序,域名规则匹配域名,IP规则匹配IP)
  "rules": [],       规则列表 规则对应出站协议或负载均衡器 详见后方
  "balancers": []    负载均衡器 详见后方
}
RuleObject
{
  "type": "field",      照抄项,单选
  "domain": [           域名匹配
    "baidu.com",        字符串(包含就行);正则表达式"regexp:";子域名"domain:";完整匹配"full:";预定义域名列表"geosite:"(详见下方);文件中加载"ext:file文件名:tag标签"(文件放在v2ray所在目录,文件中包含tag,格式与geosite.dat相同)
    "regexp:",
    "geosite:cn" 
  ],
  "ip": [               IP匹配
    "0.0.0.0/8",        IP;子网(IP/mask);GeoIP"geoip:双字符国家代码或private包含所有私有地址";从文件中加载"ext:file:tag"(格式与geoip.dat相同)
    "10.0.0.0/8",       
    "fc00::/7",
    "fe80::/10",
    "geoip:cn"
  ],
  "port": "53,443,1000-2000",  端口匹配
  "network": "tcp",            类型匹配 tcp;udp;tcp,udp
  "source": [                  来源于 IP;子网(IP/mask)
    "10.0.0.1"
  ],
  "user": [                    用户邮箱匹配 只有shadowsocks 和 vmess 支持
    "love@v2ray.com"
  ],
  "inboundTag": [              匹配入站协议标识
    "tag-vmess"
  ],
  "protocol":["http", "tls", "bittorrent"],   协议匹配 http tls bittorrent 需开启sniffing
  "attrs": "attrs[':method'] == 'GET'",       一段脚本  Starlark语言
  "outboundTag": "direct",                    对应一个额外出站连接配置的标识 不知道啥意思
  "balancerTag": "balancer"                   对应一个负载均衡器的标识 与出站标识二选一
}
BalancerObject
{
  "tag": "balancer",                       标识
  "selector": []                           出站标识前缀匹配,比如a匹配 标识为a开头的
}
```
### 预定义域名列表 由[domain-list-community](https://github.com/v2ray/domain-list-community)维护

## policy本地策略   用户相关的权限策略 没用
## reverse反向代理配置
```
{
  "bridges": [{                         桥,内网v2ray
    "tag": "bridge",                    由bridge发出的流量都带有这个tag, 可以由inboundTag识别
    "domain": "test.v2ray.com"          使用该域名与外网v2ray连接 不必真实存在
  }],
  "portals": [{                         门户,公网v2ray
    "tag": "portal",                    门户标识,路由中使用outboundTag将流量转发到这个tag
    "domain": "test.v2ray.com"          流量目标是该域名,则认为是桥发来建立连接的,否则就是需要转发给桥的流量.
  }]
}
```
## transport底层传输配置
## inbounds入站连接配置
```json
{
  "port": 1080,
  "listen": "127.0.0.1",
  "protocol": "协议名称",
  "settings": {},
  "streamSettings": {},
  "tag": "标识",
  "sniffing": {                            流量探测
    "enabled": false,
    "destOverride": ["http", "tls"]
  },
  "allocate": {
    "strategy": "always",                  端口分配策略 保持always就行
    "refresh": 5,                          保持always时 该项无效
    "concurrency": 3                       保持always时 该项无效
  }
    "enabled": false,

}
```
## outbounds出站连接配置
```json
{
  "sendThrough": "0.0.0.0",                主机发送数据的ip 主机多ip从哪个ip发出
  "protocol": "协议名称",                  
  "settings": {},
  "tag": "标识",
  "streamSettings": {},
  "proxySettings": {
    "tag": "another-outbound-tag"          没懂
  },
  "mux": {                                 一条TCP上分发多个TCP连接的数据,看视频或下载都有反效果.
    "enabled": false,                      是否启用
    "concurrency": 8                       不用也罢
  }
}
```

## 协议列表
* 入站"settings":{"inboundconfigurationobject":{}}
* 出站:OutboundConfigurationObject
### blackhole 黑洞 出站
```
{
  "response": {       配置黑洞相应数据
    "type": "none"    none直接关闭, http返回http 403数据包
  }
}
```
### dns 出站 只接受 dnf 查询    将IP查询(A和AAAA)转发给内置DNS服务器。
```
{				都不用配置,涉及到dns查询包的细节
    "network": "tcp",
    "address": "1.1.1.1",
    "port": 53
}
```
### dokodemo-door 任意门 入站
> 把所有进入此端口的数据发送至指定服务器的一个端口, 达到端口映射效果
```
{
  "address": "8.8.8.8",		流量转发的目的IP或域名
  "port": 53,			流量转发的目的端口,必填
  "network": "tcp",             可接受的类型 tcp; udp; tcp,udp
  "timeout": 0,			入站数据的时间限制 默认300 connIdle策略 不懂
  "followRedirect": false,      当为true 识别有IPtables 转发而来的数据 并转发到目标地址. 详见传输配置
  "userLevel": 0                用户等级  不知道干嘛用的
}
```
### freedom 向任意网络发送数据 出站
```

{
  "domainStrategy": "AsIs",           AsIs不经过内建dns;UseIP使用内建dns建立连接后转发 
  "redirect": "127.0.0.1:3366",       重定向,改变流量本身的目的地. 用不上
  "userLevel": 0                      用户等级 不知道干嘛用的
}
```
### http 入站/出站 暂时没用
### mtproto 入/出 为Telegram使用暂时没用
### shadowsocks 入/出 不计划使用该协议
### socks 入/出 socks4,5 暂时没用 可能有用于socks代理等
### vmess 入/出
* 出站
```
{
  "vnext": [
    {
      "address": "127.0.0.1",                                   目的IP或域名
      "port": 37192,                                            目的端口
      "users": [
        {
          "id": "27848739-7e62-4138-9fd3-098a63964b6b",         uuid
          "alterId": 4,                                         在主ID基础上再生出多个ID防止探测 应该在运行中自动与服务器沟通并创建
          "security": "auto",                                   auto就行
          "level": 0                                            用户等级
        }
      ]
    }
  ]
}
```
* 入站
{
  "clients": [
    {
      "id": "27848739-7e62-4138-9fd3-098a63964b6b",   uuid
      "level": 0,                                     用户等级  详见本地策略
      "alterId": 4,                                   如上
      "email": "love@v2ray.com"                       区分不同用户流量
    }
  ],
  "default": {
    "level": 0,                                       用户等级
    "alterId": 4                                      如上 推荐4
  },
  "detour": {
    "to": "tag_to_detour"                             入站协议的标签,入站协议必须是vmess
  },
  "disableInsecureEncryption": false                  断开来自不安全的加密方式 选true
}
