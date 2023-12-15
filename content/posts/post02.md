---
title: "使用box for magisk中的sing-box核实现透明代理"
date: 2023-12-02
draft: false
---
<!--more-->

### 0.前置条件与使用场景说明
我的使用场景是有一台Pixel 3作为旁路由（没有正经路由设备是这样的）。已经解锁了bootloader并刷入了magisk 26，并且常年接入电源。  

使用透明代理的好处不多赘述了，从WiFi calling到iCloud Private Relay，或者是单纯地给自己的设备省电都很有用。    

首先应当放一个官方文档在这里：[box_for_magisk中文README][box4magisk中文readme]    

部署时应以官方文档的教程为主，此处重点记录自己踩过的坑。


### 1.刷入模块
从项目的[Release][bof4magiskrelease页面]中下载最新的模块并刷入，Busybox我也一并刷入了。  

重启后由于刚刷入模块，它的默认内核是clash（本文日期：2023.12.2）是没有正确节点配置的，需要先进入Magisk中把这个模块禁用。

![禁用此模块][禁用模块图片]

禁用后进行内核的更新。  
```bash
#更新所有内核
su -c /data/adb/box/scripts/box.tool all
```

### 2.导入配置
singbox的在box for magisk中的配置在/data/adb/box/sing-box/config.json中。如果使用shell(su)的话只有vi用，或者在termux中就可以配合sudo来使用vim打开它。  

我的使用场景是把自己的本地的配置文件修改并导入。
//（TODO：如何添加singbox订阅链接？）  
实际上box for magisk没有提供singbox的订阅更新，当前版本1.4.2只提供了clash的订阅更新。
所以自己写个脚本，想更新的时候跑一下，跑完重启box for magisk即可：
```bash
sudo wget $sing-box订阅链接$ -O /data/adb/box/sing-box-config.json
```


首先你需要确保你有一份sing-box的配置文件，或者订阅链接。其他内核的配置需要进行订阅转换（或者直接使用其他核心。）  

下面是一些要注意的地方：  

1.导入你的配置文件时，singbox在1.8.0版本中，实验性功能下弃用了一些原有的选项，并转而使用cached_files。因此首先需要检查自己的singbox配置在experimental这儿开启了clash_api和cache_file，并且正确配置。  

为什么这是必要的？因为如果一份配置文件中有多个节点或selector的话，通过clash_api或是v2fly_api来切换是比较方便的。下面是一个配置示例：（更多的自定义请自行参考[sing-box说明文档][sing-box说明文档]）
```json
"experimental": {
    "clash_api": 
    {
        "external_controller": "127.0.0.1:9090", 
        "secret": "Nightcord_at_25:00"
    },
    "cache_file":
    {
        "enabled": true,
        "store_fakeip": false
    }
}
```
2.需要确保你的配置文件中的tproxy的端口不和settings.ini中的配置冲突。如果没有tproxy这一项的话就不需要在意了，启动的时候模块会自动补上的。

其余注意事项请参考官方文档。

接着需要到/data/adb/box/settings.ini中修改内核，将bin_name后的值修改为sing-box。

### 3.启动服务
（理论上）在Magisk中启用模块，透明代理就启动了。但是在Shell中使用命令启动可以方便看到log和报错。启动前建议关闭本机代理。  

启动/关闭命令如下：
```bash
# 启动 BFM
  su -c /data/adb/box/scripts/box.service start &&  su -c /data/adb/box/scripts/box.iptables enable

# 停止 BFM
  su -c /data/adb/box/scripts/box.iptables disable && su -c /data/adb/box/scripts/box.service stop
```
也可以添加为start.sh和stop.sh来使用。  

如果没有红字报错，那么就有可能是启动成功了。接下来前往[clash面板][razord面板]，输入方才设置的端口和secret试试能不能连上api。连接成功后如图：  
![clash api连接成功示例][clash api连接成功示例]

### 4.其他杂项
1.box for magisk默认是也为热点提供透明代理的。在我的使用场景中就有从其他设备中控制这里的面板的需求。因此对其他设备也开放监听（使其可以连接到api）需要更改clash_api中的设置，将external_controller中的127.0.0.1（本机）改成0.0.0.0（任意）。
```json
    "clash_api": 
    {
        "external_controller": "0.0.0.0:9090", 
        "secret": "Nightcord_at_25:00"
    }
```


[box4magisk中文readme]: https://github.com/taamarin/box_for_magisk/blob/master/docs/index_cn.md
[bof4magiskrelease页面]: https://github.com/taamarin/box_for_magisk/releases
[禁用模块图片]: https://41vk6f-my.sharepoint.com/:i:/g/personal/a8ec29b_nightcord_org/EStjyEirSN1GudrO2Q35--EBM_tDx0RndgEBBS-0Zg-NUg?download=1
[sing-box说明文档]: https://sing-box.sagernet.org/configuration/experimental/
[razord面板]: https://clash.razord.top/
[clash api连接成功示例]: https://41vk6f-my.sharepoint.com/:i:/g/personal/a8ec29b_nightcord_org/EQdUX9fvpcRCom47B1kcBaABf6nazZnlncqCQ8wxTSipkw?download=1
