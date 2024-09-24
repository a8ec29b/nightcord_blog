---
title: "410随身Wifi制作浓缩猫池"
date: 2024-09-24
draft: false
---
<!--more-->

### 0.背景故事  
选择410作为浓缩猫池，主要还是因为它比较便宜。在各大购物平台上往往是5-10元一抽。运气好的话可以抽中特等奖：UFI003+自带卡槽+8GB存储。  
不过8GB存储在浓缩猫池方面倒是没什么大用，但是要是运气不好抽中没卡槽的asr强控，那就只能垫桌脚吃灰了。不过好在发现一个稳定商家卖410之后，就只需要再从这个商家那边抽卡就好。~~更何况这10元一抽的抽卡价也比某些二游便宜呢。~~  
如果有折腾过410的话，可能会觉得410作为猫池是不太靠谱的——它没有Volte，也没有cdma2000（也许有的硬件支持，但是实际上到了软件这块都不支持了），电信短信是无法转发的，只能空有一个LTE信号开开流量用。~~更不巧的是410拿来开热点的表现实在是太令人堪忧了，真的不推荐这么做。~~  
但是情况碰到漫游卡的时候就不一样了：漫游到中国电信only的境外运营商真的不多，我见过的只有CTMO、CTHK，还有可能会注册到中国电信的3HK。（当然还有它们旗下的MVNO。）这种时候拿410做猫池就相当合理了——大部分运营商都可以收到短信。  

### 1.准备工作/备份  
收到410之后，首先需要确认版号。如果没有小螺丝刀，则请购买一个小螺丝刀并将随身WiFi的外壳彻底拆开，找到看起来像是板号的东西——可能是sp970，可能是UFI003，可能是UFI001C。这里主要介绍UFI003，~~因为它折腾起来最简单。~~  
然后观察一下有没有卡槽。没有的话需要自己焊接一个上去，此处不提供相关教程请自行搜索。（当然如果你和我一样没这个手艺的话，也可以选择找个手机店，或者在上面部署一个哪吒探针控制所有棒子废物利用。）    
接着按住Reset键（一般就是卡槽附近的小按键）的同时插入Windows电脑（建议选择不低于Windows 10的中文版，然后把UTF-8支持关掉），使用miko工具将整个9008备份出来。

![](https://alist.nightcord.org/d/imgprivate/post06-fig01.png)

**9008全盘备份是一切410棒子开始折腾的基础。只要备份在就很有希望在出错的时候救回来。不要不备份就刷机。**  
备份出来后会获得一个3-4GiB或者6-8GiB的文件。别急着传云盘然后删掉，后面还有用。**建议开机后登录后台查看棒子的IMEI，不要相信外壳上印的或者外壳的贴纸，然后把后台显示的IMEI跟在备份出来的文件名之后作为标识符。这在刷多根棒子或者救砖的时候很有用。**  

### 2.获取root后进行全网通流程
一般情况下老旧410棒子可能会有adb。如果没有的话请八仙过海各显神通。每个棒子方式可能会不一样这里不提供教程。  
获得adb之后，使用adb进入fastboot并刷写具有root的boot进boot分区。可以使用别人修改好的镜像，或者把刚才的9008镜像使用压缩软件打开，把boot.img提取出来，建议使用Magisk 22.1修补，然后fastboot刷入。刷机基础不再赘述，不会的话也可以考虑各种随身Wifi工具箱，它们也可能直接提供修改好的boot。
重启之后，关闭无信号自动重启，并获取切卡密码:  

```
adb shell
su
setptop persist.ufi.signal.check no
setprop persist.ufi.nosignal.reboot no
cat /system/build.prop
```

cat build.prop之后寻找"simswitchpword"或类似语句，不同棒子可能不一样。后面的值就是切卡密码。获得密码之后前往后台切换到外置卡槽，然后重启，使关闭无信号自动重启的策略生效。  
重启后使用星海工具，强开端口并备份qcn。 **备份时请勿插卡！** 备份出来的qcn理应在553-558KiB之间。目前我仅碰到过一次qcn为538KiB且最后能正常使用的。  

![](https://alist.nightcord.org/d/imgprivate/post06-fig02.png)

多次备份时，备份出来的大小理应相同，但是md5会不同。  
备份完成之后重启到fastboot，并选择高通擦基带。（见上图，不再标注）擦基带完成之后重启到安卓，然后开端口恢复QCN。

![](https://alist.nightcord.org/d/imgprivate/post06-fig03.png)

可能有的人会疑惑，为什么要备份qcn又写入呢？多此一举？这其实是全网通流程。不执行此流程可能会导致刷入Debian后无法收到移动网络下短信。  

### 3.刷入Debian并配置环境做好转发
恢复好QCN后就可以刷入Debian了，这里使用酷安-jsbsbxjxh66制作的Debian12。或许你也可以看看Openstick项目并自编译一个，但是我未尝试过，也不知道最终能不能用。  
刷好之后打开设备管理器，找到未知设备，或者是Android Device，升级它的驱动，将它的驱动更改为如图所示相同：（如果此时你的电脑已经可以连上adb shell了，则不需要更改）  

![](https://alist.nightcord.org/d/imgprivate/post06-fig05.png)
![](https://alist.nightcord.org/d/imgprivate/post06-fig04.png)

接着进入adb shell。

```
adb shell
export TERM=linux //这一步是为了打开nmtui
nmtui
```

打开nmtui后，首先在“编辑连接”中，选中Wifi，在右侧删除；接着返回选择“启用连接”，此时它就会搜索附近的Wifi。输入密钥连接到Wifi。  
可以使用ifconfig或者从路由器查看棒子的ip，以便以后脱离电脑后ssh使用。  
接下来往 /etc/rc.local 中添加如下切卡脚本：

```
sleep 15
echo 1 > /sys/class/leds/sim:sel/brightness
echo 0 > /sys/class/leds/sim:en/brightness
echo 0 > /sys/class/leds/sim:sel2/brightness
echo 0 > /sys/class/leds/sim:en2/brightness
modprobe -r qcom-q6v5-mss
sleep 1
modprobe qcom-q6v5-mss
sleep 1
systemctl restart rmtfs
sleep 1
systemctl restart dbus-org.freedesktop.ModemManager1.service
```

完成后需要编译安装libqmi最新版。请自行Google搜索。  

还记得一开始留下来的备份文件吗？使用压缩软件打开，找到0.modem.fat，将其解压，可以找到这些文件：

![](https://alist.nightcord.org/d/imgprivate/post06-fig06.png)

将其中的 mba.mbn、modem.bxx(数字)、modem.mdt这些文件传入到棒子的 /etc/firmware 中。可能会替换文件，选择覆盖。  
接着就可以插入Removable euicc了。经过我的测试，樱花1、樱花2(5ber)、手制st33都可以读写。请注意非东信和平/ESTK可能不会在空卡下模拟Profile，请先写入一个profile并enable后插入。  
插入卡重启之后，等待一会儿使用 mmcli -m 0 查看信号。如果有信号，那么之前的步骤就算全部成功了。  
至此为止已经完成了所有前期（其实是90%）准备工作了，最后的猫池部署请参考 https://github.com/damonto/telegram-sms 的Readme。  

如果你有自建的反代tgapi的话，可以在编写telegram-sms.service的时候参考下面的配置：  

```
[Unit]
Description=Telegram SMS Forwarder
After=network.target

[Service]
Type=simple
User=root
Restart=on-failure
ExecStart=/root/telegram-sms-linux-arm64 --bot-token=YourTelegramToken --admin-id=YourTelegramChatID --endpoint=你的自建反代地址，地址最后不要有斜杠
RestartSec=10s
TimeoutStopSec=30s

[Install]
WantedBy=multi-user.target
```
