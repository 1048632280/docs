# Ubuntu搭建TeamSpeak网易云机器人教程

---

> [!NOTE]
> 运行环境：Ubuntu22.04

---

## 配置Teamspeak

### 准备阶段

创建一个teamspeak用户（不建议使用root用户直接操作）

```
# 切换root用户
su - 
# 新建一个用户
useradd -s /bin/bash -m teamspeak
# 加到sudo组
usermod -aG sudo teamspeak
# 设置密码
passwd teamspeak
# 切换teamspeak用户
su - teamspeak
```

### 安装服务端

前往下载官方linux服务端，最新版本为`SERVER 64-BIT 3.13.7`。

[TeamSpeak 下载 | TeamSpeak](https://www.teamspeak.com/zh-CN/downloads/#server)

> [!ATTENTION]
> 注意！ts3中文站是盗版网站，会改动你的hosts文件，导致你无法进入官方服务器或者缺失部分账号功能，请认准teamspeak官网。

或者使用命令直接下载

```
wget https://files.teamspeak-services.com/releases/server/3.13.7/teamspeak3-server_linux_amd64-3.13.7.tar.bz2
```

解压

```
tar -xvf teamspeak3-server_linux_amd64-3.13.7.tar.bz2
```

重命名解压后的目录

```
mv  teamspeak3-server_linux_amd64 teamspeak
```

进入teamspeak目录

```
cd teamspeak
```

同意许可协议

```
touch .ts3server_license_accepted
```

启动服务端

```
./ts3server_startscript.sh start
```

随后显示如下

```
------------------------------------------------------------------
------------------------------------------------------------------
                      I M P O R T A N T                           
------------------------------------------------------------------
               Server Query Admin Account created                 
         loginname= "serveradmin", password= "xxxxxxxx"
         apikey= "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
------------------------------------------------------------------

------------------------------------------------------------------
                  I M P O R T A N T                           
------------------------------------------------------------------
  ServerAdmin privilege key created, please use it to gain 
  serveradmin rights for your virtualserver. please
  also check the doc/privilegekey_guide.txt for details.

   token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
------------------------------------------------------------------

```

记录token值，现在就可以从客户端连接TS服务器了，**记得开放端口和防火墙**

从TS客户端连接你的域名/IP，第一次进入会要求你输入管理员密钥，就是上面的token代码，输入之后你就是频道的所有者了。

> [!TIP]
> 最好在`ts客户端 -> 权限 -> 权限密钥`里面备份一个权限密钥，**token只能使用一次**。意外失去管理员身份需要进入后台获取一个新的token值

ctrl+c退出

### 配置规则

开放端口和防火墙

| 端口  | 协议 | 说明                      |
| ----- | ---- | ------------------------- |
| 9987  | UDP  | TeamSpeak 默认语音服务    |
| 10011 | TCP  | TeamSpeak ServerQuery raw |
| 10022 | TCP  | TeamSpeak ServerQuery SSH |
| 30033 | TCP  | TeamSpeak 文件传输        |
| 41144 | TCP  | TSDND                     |

### 设置开机自启



新建teamspeak.service

```
sudo vim /etc/systemd/system/teamspeak.service
```

注意，配置文本中`/home/teamspeak/teamspeak`为服务端文件目录路径。若自定义文件目录请自行改动。

```
[Unit]
Description=Teamspeak Service
Wants=network.target

[Service]
WorkingDirectory=/home/teamspeak/teamspeak
ExecStart=/home/teamspeak/teamspeak/ts3server_minimal_runscript.sh
ExecStop=/home/teamspeak/teamspeak/ts3server_startscript.sh stop
ExecReload=/home/teamspeak/teamspeak/ts3server_startscript.sh restart
Restart=always
RestartSec=15
 
[Install]
WantedBy=multi-user.target
```

系统服务操作

```
# 更新配置
sudo systemctl daemon-reload

# 设置开机启动
sudo systemctl enable teamspeak.service

# 启动服务
sudo systemctl start teamspeak.service

# 停止服务
sudo systemctl stop teamspeak.service

# 重启服务
sudo systemctl restart teamspeak.service

# 查看状态
# Active: active (running) 即为运行成功，若不成功可以kill掉之前开启的服务端进程
sudo systemctl status teamspeak.service

```

至此TS服务端配置完毕。

---

## 配置机器人

### 安装ffmpeg

```
sudo apt install libopus-dev ffmpeg
```

### 安装本体

> [!NOTE]
> 这里使用了[ZHANGTIANYAO1](https://github.com/ZHANGTIANYAO1)大佬构建的linux版TS3AudioBot（**官方版本存在编译问题，无法正常加载插件**），需要下载最初release版本，版本号为[1.1.0](https://github.com/ZHANGTIANYAO1/TS3AudioBot-NetEaseCloudmusic-plugin/releases/tag/1.1.0)的[with.TS3Bot.linux-x64.zip](https://github.com/ZHANGTIANYAO1/TS3AudioBot-NetEaseCloudmusic-plugin/releases/download/1.1.0/with.TS3Bot.linux-x64.zip)。附上github项目[TS3AudioBot-NetEaseCloudmusic-plugin](https://github.com/ZHANGTIANYAO1/TS3AudioBot-NetEaseCloudmusic-plugin)
>
> 插件使用[FiveHair](https://github.com/FiveHair/TS3AudioBot-NetEaseCloudmusic-plugin-UNM/commits?author=FiveHair)大佬修改后的版本[TS3AudioBot-NetEaseCloudmusic-plugin-UNM](https://github.com/FiveHair/TS3AudioBot-NetEaseCloudmusic-plugin-UNM)，版本号为v1.0

建议自行下载，远程传输文件给服务器，不然下载很慢

```
# 本体with.TS3Bot.linux-x64.zip
wget https://github.com/ZHANGTIANYAO1/TS3AudioBot-NetEaseCloudmusic-plugin/releases/download/1.1.0/with.TS3Bot.linux-x64.zip
# 插件YunPlugin-UNM-v1.0.zip
wget https://github.com/FiveHair/TS3AudioBot-NetEaseCloudmusic-plugin-UNM/releases/download/v1.0/YunPlugin-UNM-v1.0.zip
```

解压with.TS3Bot.linux-x64.zip和YunPlugin-UNM-v1.0.zip

```
unzip with.TS3Bot.linux-x64.zip
# 将解压后的linux-64目录改名
mv linux-x64 TS3AudioBot
# 解压后会得到两个文件YunPlugin-UNM.dll和YunSettings.ini
unzip YunPlugin-UNM-v1.0.zip
```

删除原来的插件，并拷贝最新版的YunPlugin-UNM.dll和YunSettings.ini进plugins目录

```
rm TS3AudioBot/plugins/*
mv YunPlugin-UNM.dll TS3AudioBot/plugins/ && mv YunSettings.ini TS3AudioBot/plugins/
```

进入TS3AudioBot启动服务

```
cd TS3AudioBot
./TS3AudioBot
```

这里可能会提示权限不够，这个文件默认是不可执行的，所以要给它权限

```
chmod a+x TS3AudioBot
./TS3AudioBot
```
> [!WARNING]
> 如下报错请跳转`常见问题 -> 无法启动bot`
>
> No usable version of libssl was found
> Aborted (core dumped)

接下来会有几步配置需要输入

```
1. Do you want to set up an admin in the default permission file template? [Y/n]
# 设置管理员id，在teamspeak -> 工具 -> 身份 -> UID

2. Please enter the ip, domain or nickname (with port; default: 9987) where to connect to:
# 设置服务器ip或别名

3. Please enter the server password (or leave empty for none):
# 设置服务器密码，没有直接回车
```

最后ctrl+c结束

### 设置开机自启

新建ts3audiobot.service

```
sudo vim /etc/systemd/system/ts3audiobot.service
```

输入以下配置。同样的，`/home/teamspeak/TS3AudioBot`取决于你安装TS3AudioBot的位置

```
[Unit]
Description=TS3AudioBot
After=teamspeak.service

[Service]
WorkingDirectory=/home/teamspeak/TS3AudioBot
ExecStart=/home/teamspeak/TS3AudioBot/TS3AudioBot
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```

系统服务操作

```
# 更新配置
sudo systemctl daemon-reload

# 设置开机启动
sudo systemctl enable ts3audiobot.service

# 启动服务
sudo systemctl start ts3audiobot.service

# 停止服务
sudo systemctl stop ts3audiobot.service

# 重启服务
sudo systemctl restart ts3audiobot.service

# 查看状态
# Active: active (running) 即为启动成功
sudo systemctl status ts3audiobot.service

```

### 网页管理

> [!TIP]
> 作者配置的bot并没有WebInterface目录
>
> 想要实现页面管理，需访问原作者[Splamy/TS3AudioBot: Advanced Musicbot for Teamspeak 3 (github.com)](https://github.com/Splamy/TS3AudioBot)项目
>
> 下载原版TS3AudioBot，将其中的WebInterface文件夹放入TS3AudioBot目录中即可
>
> 之前运行bot服务时看到的ERROR报错也和这个有关，但不影响最终效果
>
> 不过也建议配置一下，方便拓展更多功能

开放58913端口

| 端口  | 协议 | 说明            |
| ----- | ---- | --------------- |
| 58913 | TCP  | ts3audiobot web |

在teamspeak服务器内双击私聊机器人，发送

```
!api token
```

访问ip:58913，输入token即可登录网页管理

主要功能有：上传本地音乐，开启多个机器人

### 本章结语

到这里为止，音乐机器人以及可以正常使用，plugins目录下ini配置文件是别人的网易云API，不安全。想要登录听vip歌曲还要在本地部署网易云API

---

## 本地部署网易云API

开放3000端口

| 端口 | 协议 | 说明              |
| ---- | ---- | ----------------- |
| 3000 | TCP  | 网易云API后台网页 |

### 安装git

> [!NOTE]
> 首先，确认你的系统是否已安装git，可以通过`git`指令进行查看

安装git

```
sudo apt-get install git
```

安装完成后进行git配置

```
# 设置用户名
git config --global user.name "你的名字"
# 设置邮箱
git config --global user.email "你的邮箱"
# 查看配置信息
git config --list
```

配置完成后，需要创建验证用的公钥，因为`git`是通过`ssh`的方式访问资源库的，所以需要在本地创建验证用的文件

```
ssh-keygen -t rsa -C "你的邮箱" 
```

一路回车直到出现随机字符图为止

```
+---[RSA 3072]----+
|xxxxx            |
|xxxxx            |
|xxxxxxx          |
|xxxxxxxx         |
|xxxxxxxxx        |
|xxxxxxxx         |
|xxxxxxxxx        |
|xxxxx            |
|xxxx             |
+----[SHA256]-----+
```

查看生成的密钥

```
cat  ~/.ssh/id_rsa.pub
```

将ssh-rsa开头，邮箱结尾的整个字符串复制下来。进入github，依次点击`Settings -> SSH Keys -> Add SSH Key`，新增一个密钥

回到终端，检测是否可以连接到github

```
ssh -T git@git.oschina.net
```

### 安装node.js

安装Node.js

```
sudo apt install nodejs
```

安装npm

```
sudo apt install npm 
```

利用n来管理版本

```
sudo npm install -g n
# 如果卡在这里，请更换国内镜像源，再重新安装
npm config set registry https://registry.npmmirror.com
```

需要nodejs14以上版本

```
# 升级长期支持版本
sudo n lts
# 有些系统版本老旧，缺失一些文件，可以直接指定安装版本
sudo n xx.x.x
```

切换最新版本

```
# 方向键上下选择即可
sudo n
# 查看nodejs版本，若是没有显示切换后版本，新建一个终端
node -v
```

### 搭建API

git [NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)

```
git clone git@github.com:Binaryify/NeteaseCloudMusicApi.git
```

用户家目录会多出一个NeteaseCloudMusicApi目录，进入

```
cd NeteaseCloudMusicApi
```

执行安装(有可能版本不够高，按照提示更新即可)

```
sudo npm install
```

启动服务

```
node app.js
```

此时会显示

```
server running @ http://localhost:3000
```

访问ip:3000能访问到网易云API页面即为配置成功

ctrl+c退出

### 设置开机自启

新建 netease.service 

```
sudo vim /etc/systemd/system/netease.service
```

复制下列内容到 netease.service 文件中

注意，这里是你的home用户目录下git到的网易云API地址，根据自身情况修改

```
[Unit]
Description=Netease Cloud Music API Service
After=network.target

[Service]
WorkingDirectory=/home/teamspeak/NeteaseCloudMusicApi/
ExecStart=node /home/teamspeak/NeteaseCloudMusicApi/app.js
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```

系统服务操作

```
# 更新配置
sudo systemctl daemon-reload

# 设置开机启动
sudo systemctl enable netease.service

# 启动服务
sudo systemctl start netease.service

# 停止服务
sudo systemctl stop netease.service

# 重启服务
sudo systemctl restart netease.service

# 查看状态
# Active: active (running) && 
# server running @ http://localhost:3000 即为启动成功
sudo systemctl status netease.service
```

ctrl+c退出

---

## 最后阶段

### 修改插件配置

编辑本地网易云API地址

```
vim TS3AudioBot/plugins/YunSettings.ini
```

修改WangYiYunAPI_Address地址为网易云API的链接

```
[YunBot]
playMode=3
WangYiYunAPI_Address=http://localhost:3000
cookies1=
UNM_Address=
```

重启服务器，确保三个服务都运行成功，机器人应该就已经在你的TS频道了

### 启动机器人

接下来都是ts客户端操作，不需要在服务器

双击机器人，输入

```
!plugin list
```

可以看到当前插件状态（RDY就绪）

```
"TS3AudioBot": All available plugins:
#0|RDY|YunPlgun (BotPlugin)
```

需要手动把插件启用

```
!plugin load 0
```

查看当前插件状态（+ON运行）

```
!plugin list
# bot消息
"TS3AudioBot": All available plugins:
#0|+ON|YunPlgun (BotPlugin)
```

### 运行成功

现在进行登录操作

首先给机器人最高权限，以便于加载二维码头像

```
!yun login
```

点击机器人，右侧显示出二维码头像，扫码登录即可

成功会提示

```
"Aster": !yun login
"TS3AudioBot": 登陆成功
```

### 常用指令

建议放入频道简介

```
正在播放的歌单的图片和名称可以点机器人看它的头像和描述
vip音乐想要先登陆才能播放完整版本:（输入指令后扫描机器人头像二维码登陆)
!yun login

双击机器人，目前有以下指令（把[xxx]替换成对应信息，包括中括号）
1.立即播放网易云音乐
!yun play [音乐名称]

2.添加音乐到下一首
!yun add [音乐名称]

3.播放网易云音乐歌单(如果提示Error: Nothing to play...重新输入指令解决)
!yun gedan [歌单名称]

4.播放网易云音乐歌单id
!yun gedanid [歌单名称]

5.立即播放网易云音乐id
!yun playid [歌单id]

6.添加指定音乐id到下一首
!yun add [音乐id]

7.播放列表中的下一首
!yun next

8.修改播放模式
!yun mode [模式选择数字0-3]
0 = 顺序播放 1 = 顺序循环 2 = 随机播放 3 = 随机循环

需要注意的是如果歌单歌曲过多需要时间加载，期间一定一定不要输入其他指令
```

---

## 解锁版权（未实现）

### 环境要求

> [!NOTE]
> node.js 18+
>
> npm 10.2.4+
>



### 搭建API

git [Unblock Netease Music 维护小组 (github.com)](https://github.com/UnblockNeteaseMusic)

```
git clone https://github.com/UnblockNeteaseMusic/server.git UnblockNeteaseMusic
```

用户家目录会多出一个UnblockNeteaseMusic目录，进入

```
cd UnblockNeteaseMusic
```

安装服务

```
sudo npm install
```

启动服务

```
# 默认端口为8080，我在这里改为3001
node app.js -p 3001
```

显示以下信息即为成功

```
HTTP Server running @ http://0.0.0.0:3001
```

### 设置开机自启

新建UnblockNeteaseMusic.service

```
sudo vim /etc/systemd/system/UnblockNeteaseMusic.service
```

复制下列内容到 UnblockNeteaseMusic.service 文件中

```
[Unit]
Description=netease proxy service
After=network-online.target

[Service]
WorkingDirectory=/home/teamspeak/UnblockNeteaseMusic
ExecStart=node /home/teamspeak/UnblockNeteaseMusic/app.js -p 3001 -e https://music.163.com -s
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```

系统服务操作

```
# 更新配置
sudo systemctl daemon-reload

# 设置开机启动
sudo systemctl enable unm.service

# 启动服务
sudo systemctl start unm.service

# 停止服务
sudo systemctl stop unm.service

# 重启服务
sudo systemctl restart unm.service

# 查看状态
# Active: active (running) && 
# HTTP Server running @ http://0.0.0.0:3001 即为启动成功
sudo systemctl status unm.service
```

### 修改插件配置

编辑配置文件

```
vim TS3AudioBot/plugins/YunSettings.ini
```

修改UNM_Address地址为解锁网易云API的链接

```
[YunBot]
playMode=3
WangYiYunAPI_Address=http://localhost:3000
cookies1=
UNM_Address=http://0.0.0.0:3001
```

修改NeteaseCloudMusicApi运行环境

```
vim NeteaseCloudMusicApi/app.js
```

添加环境变量

```
process.env['NODE_TLS_REJECT_UNAUTHORIZED'] = 0
```

重新启动netease.service

```
# 重启服务
sudo systemctl restart netease.service
# 查看状态
sudo systemctl status netease.service

# 显示如下
(node:852) Warning: Setting the NODE_TLS_REJECT_UNAUTHORIZED environment>
(Use `node --trace-warnings ...` to show where the warning was created)
server running @ http://localhost:3000
```

重新启动服务器

---

## 常见问题

### 插件加载错误

>"TS3AudioBot": Error: Plugin error: UnknownError

说明你使用了原版TS3AudioBot，加载不了插件

### 无法启动bot

> No usable version of libssl was found 
>
> Aborted (core dumped)

libssl版本不可用，去[Index of /ubuntu/pool/main/o/openssl1.0](http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/)安装一个1.0版本的libssl

例如，http://security.ubuntu.com/ubuntu/pool/main/o/openssl1.0/libssl1.0.0_1.0.2n-1ubuntu5_amd64.deb

### 拒绝连接

> "TS3AudioBot": An unexpected error occurred: Connection refused Connection refused

说明你的YunSettings.ini中的WangYiYunAPI_Address路径没配置好

### 缺失权限

> "TS3AudioBot": Error: You cannot execute 'yun play'. You are missing the 'cmd.yun.play' right!

普通用户的权限没有配置，编写rights.toml，具体查看[Wiki](https://github.com/Splamy/TS3AudioBot/wiki)中的[Rights](https://github.com/Splamy/TS3AudioBot/wiki/Rights)

### 登录报错

> "TS3AudioBot": An unexpected error occurred:  The remote server returned an error: (502) Bad Gateway.

扫描二维码后登陆失败，没有本地部署网易云API

### 无法连接服务

查看你的端口是否开放，防火墙是否关闭

### 报错含ssl

查看YunSettings.ini中的Address地址是否为http而不是https

### 报错含404,405

查看YunSettings.ini中的Address地址结尾是否多加了一个/

或者为登录失败导致，需清除cookies，重新进行登录操作