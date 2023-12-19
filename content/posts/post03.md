---
title: "通过修改配置文件让Sing-box for iOS隐藏VPN图标"
date: 2023-12-19
draft: false
---
<!--more-->

首先，Sing-box自己没有提供隐藏VPN图标的功能，所以需要在配置文件里面实现。  
在inbounds下的tun入站里，首先要开启auto_route；  
然后排除路由需要在 inet4_route_exclude_address 中添加 0.0.0.0/8 或者 /31 即可，下面是一个配置示例：

```  
{  
  "type": "tun",  
  "tag": "tun-in",  
  "interface_name": "tun0",  
  "inet4_address": "172.19.0.1/30",  
  "auto_route": true, //autoroute务必开启，否则下面ipv4排除路由将不生效  
  "strict_route": false,  
  "inet4_route_exclude_address": ["0.0.0.0/8"], //关键在于加入这行  
  "inet6_route_exclude_address": ["::ffff:0.0.0.0/31"] //ipv6可加可不加，反正我没加是也能正常隐藏。
}  
```  
效果如图：
![图片](https://alist.nightcord.org/d/Public/%E5%9B%BE%E5%BA%8A/photo_2023-12-19_18-21-52.jpg)

以及我在排除这个之后出现了部分使用aes-256-gcm加密方法的ss节点不可用的情况。  
排查了很久没查出个所以然，最后发现关闭Sing-box服务，然后划掉后台，接着重开服务，那些节点就正常了。
