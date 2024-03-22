---
title: "你这辈子就是被cloudflare害了——无nginx无443端口搭建matrix/synapse服务器"
date: 2024-03-22
draft: false
---
以下玩梗，请勿当真：

你这辈子就是被cloudflare给害了，没法正经用服务器，下载nginx的时候，总是在想，要不我简化下服务看看能不能用cloudflare tunnel跑吧；制作博客的时候，总是在想，要不我用静态页面挂在cloudflare pages上吧；准备搭复杂服务的时候，服务商刚好来了波机房迁移，你的心怦怦跳，总是在想，机房迁移了之后到cloudflare CDN的延迟变高了怎么办？要是机房数据也丢了我的cloudflare account token还得去重新生成怎么办？然后机房带着新的优质线路和原封不动的数据重新开机的时候，你疯狂测试线路到cloudflare CDN的质量，服务器沉默了一会儿说唉用cloudflare用的

<!--more-->

正经博客记录：

如上面所说，我这辈子就是被cloudflare tunnel害了，到现在还不会用nginx，最基本的都不会用。所以使用cloudflare tunnel来搭建synapse服务器。

因为nightcord本身neta Discord，因此将用户的账号后缀设置为nightcord.org，而synapse的实际网站在matrix.nightcord.org，使用cloudflare worker实现。

首先生成synapse配置文件：

```bash
docker run -it --rm -v ~/synapse/synapse-data/:/data/ -e SYNAPSE_SERVER_NAME=nightcord.org -e SYNAPSE_REPORT_STATS=no matrixdotorg/synapse:latest generate
```

第一次就需要确定好生成的位置，否则生成之后挪动可能会出现权限不足的问题。出现权限不足重新生成即可。

在cloudflare zero trust设置matrix.nightcord.org的tunnel时，将需要暴露的链接设置为：

```
http://synapse（与下方container name相同）:8008
```

接着使用docker-compose：

```yaml
version: '3'

services:
  synapse:
    image: matrixdotorg/synapse:latest
    container_name: synapse
    volumes:
      - /home/username/synapse/synapse-data/:/data/
    restart: unless-stopped

  cloudflare-tunnel:
    image: cloudflare/cloudflared:latest
    container_name: cloudflare-tunnel-synapse
    restart: unless-stopped
    command:
      - tunnel
      - --no-autoupdate
      - run
    environment:
      - TUNNEL_TOKEN=nightcord at 25:00
```

启动之后访问网站，看到中间的matrix标志就是成功启动了。

接着设置cloudflare worker：

```javascript
addEventListener("fetch", (event) => {
  event.respondWith(
    handleRequest(event.request).catch(
      (err) => new Response(err.stack, { status: 500 })
    )
  );
});


export async function handleRequest(request){
  
  const url = new URL(request.url)

  const headers = {
    headers: {
      "content-type": "application/json;charset=UTF-8",
      'Access-Control-Allow-Origin': '*',  //解决可能会遇到跨域问题
    }
  }

  const serverJson = {
    "m.server": "matrix.example.com:443"  //原本不使用cloudflare tunnel而使用nginx的部署需要填8448，下面端口同理
  }

  const clientJson = {
      "m.homeserver": {
          "base_url": "https://matrix.example.com:443"
      },
  }

  var msg

  if (url.pathname.endsWith("server")) {
    msg  = JSON.stringify(serverJson)
  }

  if (url.pathname.endsWith("client")) {
    msg = JSON.stringify(clientJson)
  }

  if (msg) {
    return new Response( msg , headers);
  } else {
    return new Response('Not Found.', { status: 404 })
  }
}
```

route设置：example.com/.well-known/matrix/*

由于已经关闭了用户注册，所以手动注册一个账户：(请根据实际情况在docker容器内跑并选择正确配置文件）
```bash
register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml http://localhost:8008
```

然后就可以登录开始用了。

Worker代码原链接：https://skybirds.sbs/2023/Dendrite/#Delegation
