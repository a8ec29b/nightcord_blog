---
title: "个人自留/Debian系统新开机后初始设置"
date: 2023-11-30
draft: false
---
<!--more-->
### 1.apt update & apt upgrade

//TODO：如何更改软件源

### 2.安装sudo、vim

```bash
apt install sudo
apt install vim
```

### 3.新建用户

```bash
adduser
```

根据提示输入信息即可。（仅Debian）

创建用户之后为其赋予sudo权限：使用visudo。默认直接打开visudo可能会拉起nano，使用以下命令来用vim编辑sudoers文件：

```bash
EDITOR=vim visudo
```

定位到以下内容：

```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
```

在root那一行之后添加新的一行：

```bash
$USER$  （此处不是空格是一个tab，上下两行的第一个ALL需要对齐）  ALL=(ALL:ALL) （这里是一个空格不是tab）ALL
```

如果要设置为使用sudo不需要密码验证，则在最后一个ALL之前加上”NOPASSWD:”：

```bash
$USER$    ALL=(ALL:ALL) NOPASSWD:ALL
```

接着保存退出。如果不是通过visudo命令进入的sudoers，则可能需要使用:wq!来退出。

切换到新建的用户：

```bash
su $USER$
```

### 4.使用ssh密钥对登录

4.1新生成密钥（需要安装ssh-keygen）

```bash
ssh-keygen -t ed25519
```

生成之后需要将新生成的公钥（.pub）放入authorised_keys中：

```bash
cat id_ed25519.pub >> authorized_keys
```

将私钥（另一个文件）保存到本地在以后的登录时候用，使用SFTP或者：

```bash
cat id_ed25519
```

4.2上传自己的公钥

首先确保自己有ed25519的一对密钥，将公钥使用SFTP传输到服务器，或者直接复制粘贴来上传到服务器的 ~/.ssh/id_ed25519.pub 内。

同样地，将公钥（.pub）放入authorised_keys中：

```bash
cat id_ed25519.pub >> authorized_keys
```

由于不是ssh-keygen自动生成的，所以我们需要手动配置权限。将id_ed25519.pub设置为600：

```bash
chmod 600 id_ed25519.pub
```

接着回到家目录，将~/.ssh/设置为700：

```bash
chmod 700 ~/.ssh
```

同时请注意，.ssh/这个文件夹的所有者必须是当前用户自己，否则同样的会在登录的时候出错（ssh找不到可用的密钥对），更改文件/文件夹所有者：

```bash
sudo chown -R $USER$ $FILE$ 
```

配置完密钥对之后，前往下面的文件里面打开密钥对登录功能：

```bash
sudo vim /etc/ssh/sshd_config
```

下面是一些常用的可能会需要更改的地方：

1.登录端口，默认是22，取消注释并更改数字来更改端口：

```bash
#Port 22（默认状态）
Port $PORT$
```

2.使用密钥对登录，no改为yes：

```bash
#PubkeyAuthentication no
PubkeyAuthentication yes
```

3.是否允许root用户登录，建议关闭：

```bash
#PermitRootLogin yes
PermitRootLogin no
```

保存更改后，重启ssh服务：

```bash
sudo /etc/init.d/ssh restart
```

重启后尝试使用密钥对登录。若登录成功，则可以再关闭密码登录：

4.关闭密码登录，yes改为no：

```bash
#PasswordAuthentication yes
PasswordAuthentication no
```

*关闭密码登录≠密码用不上了，例如sudo如果不设置免密码使用则还是需要密码验证。别忘了关闭密码登录后再一次重启ssh服务。

### 5.其他杂项

1.安装Docker：（使用官方源而非apt中的docker）

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

安装完之后需要把自己加入Docker用户组（或者使用sudo来使用docker）：
