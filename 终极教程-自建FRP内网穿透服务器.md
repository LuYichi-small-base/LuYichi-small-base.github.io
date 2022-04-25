# 终极教程-自建FRP内网穿透服务器

## 前言

> 作为一个运维佬家里没有公网 IP 想远程家里的电脑拿些文件很麻烦
>
> 所以自己建了个 FRP服务器作为中转好去访问家里的设备
>
> 我这种外放访问方式的速度取决与家里的上传速度和服务器的带宽上限
>
> 我曾经试过 zerotier 延迟太高，所以用了自建 FRP 的方案。

### 什么是 frp

- frp 是一个高性能的反向代理应用，可以帮助您轻松地进行内网穿透，对外网提供服务，支持 tcp, udp,http, https 等协议类型，并且 web 服务支持根据域名进行路由转发。
- frp 内网穿透主要用于没有公网 IP 的用户，实现远程桌面、远程控制路由器、搭建的 WEB、FTP、SMB 服务器被外网访问、远程查看摄像头、调试一些远程的 API（比如微信公众号，企业号的开发）等。

## 接下来就来就是详细的配置教程

### 一、购买云服务器并初步设置

想要用 FRP 就必须要有一个具备公网 IP 的服务器才能来进行内网穿透的功能

#### 准备工作第一步-购买服务器

**购买地址**：

> [腾讯云](https://cloud.tencent.com/act/pro/lighthouse2021?fromSource=gwzcw.4398562.4398562.4398562&utm_medium=cpc&utm_id=gwzcw.4398562.4398562.4398562&cps_key=f1a1612cbec41c719b11ae528ef54e79)
>
> [阿里云](https://www.aliyun.com/product/swas?userCode=rz7dct2n&pid=mm_25282911_3455987_122436732)  

选择链接点进去注册之后就可以开始购买啦

购买服务器方式（这里以腾讯云举例，阿里云大同小异）

我们进入主页把鼠标移动到产品那里就可以看到轻量应用服务器了

[![LaWOD1.jpg](https://s1.ax1x.com/2022/04/18/LaWOD1.jpg)](https://imgtu.com/i/LaWOD1)

点进去之后就可以看到选择地域，我们选中国香港镜像选择系统镜像，选择 centos7.6 在下面的套餐配置选择 24 块的 1 核 CPU，1G 内存，每月流量 1024GB，峰值带宽 30Mbps 的套餐就可以啦，数据盘就保持不变购买时长一般都是一个月然后选立即购买就可以啦

[![LaWLuR.jpg](https://s1.ax1x.com/2022/04/18/LaWLuR.jpg)](https://imgtu.com/i/LaWLuR)

最后提交订单就可以啦

#### 准备工作第二步-配置服务器

​      鼠标移动到云产品里面有个轻量应用服务器

[![LaWbv9.jpg](https://s1.ax1x.com/2022/04/18/LaWbv9.jpg)](https://imgtu.com/i/LaWbv9)

   点更多下拉有个管理点进去

[![LaWHgJ.jpg](https://s1.ax1x.com/2022/04/18/LaWHgJ.jpg)](https://imgtu.com/i/LaWHgJ)

首先先在实例信息里面可以重置密码

[![LaW734.jpg](https://s1.ax1x.com/2022/04/18/LaW734.jpg)](https://imgtu.com/i/LaW734)

然后再防火墙里面添加规则开放防火墙

[![LaWTCF.jpg](https://s1.ax1x.com/2022/04/18/LaWTCF.jpg)](https://imgtu.com/i/LaWTCF)

到这里为止服务器购买的基本准备工作就完成了

### 二、配置 frp 服务器

我们先准备好 frp 的包

目前可以在 Github 的  [Release](http://iphone.myzaker.com/zaker/link.php?pk=60ec675d8e9f094f4554b4b8&b=aHR0cHM6Ly9naXRodWIuY29tL2ZhdGVkaWVyL2ZycC9yZWxlYXNlcw==&bcode=b83e53da&target=_new) 页面中下载到最新版本的客户端和服务端二进制文件，所有文件被打包在一个压缩包中。

接下来我们先下载我们服务器端的包选择画红线的那个包`frp_0.37.0_linux_386.tar.gz`

[![LaWI4U.jpg](https://s1.ax1x.com/2022/04/18/LaWI4U.jpg)](https://imgtu.com/i/LaWI4U)

下载下来后就要远程进入服务器了

我们要通过 xshall 或 winscp 对我们的服务器进行连接如图所示

[![LaW5NT.jpg](https://s1.ax1x.com/2022/04/18/LaW5NT.jpg)](https://imgtu.com/i/LaW5NT)

按照提示输入用户名密码之后就可以链接上啦

进入服务器后先输入 cd / 进入根目录下面

```
cd /
```

然后选择 xshall 上面如图示画红线的小工具

把我们刚刚从网上下载的压缩包从左边拖到右边或者点右键上传

[![LaW4EV.jpg](https://s1.ax1x.com/2022/04/18/LaW4EV.jpg)](https://imgtu.com/i/LaW4EV)

直接把 `frp_0.37.0_linux_386.tar.gz` 拖动到右边那个栏目中

[![LaWfH0.jpg](https://s1.ax1x.com/2022/04/18/LaWfH0.jpg)](https://imgtu.com/i/LaWfH0)

或者右键 `frp_0.37.0_linux_386.tar.gz` 选择传输

这样我们就把刚刚的那个包上传上去了

现在我们把刚刚上传上的包进行解压输入 

```
tar -zxvf frp_0.37.0_linux_386.tar.gz（看清楚你上传的包的名字）
```

[![LaWWBq.jpg](https://s1.ax1x.com/2022/04/18/LaWWBq.jpg)](https://imgtu.com/i/LaWWBq)

然后 ls 看看你解压出来的东西，就是 frp_0.37.0_linux_386 这个文件夹

ls

给他加权限：输入 chmod -R 777frp_0.37.0_linux_386

```
chmod -R 777frp_0.37.0_linux_386
```

然后 cd frp_0.37.0_linux_386 进入 frp_0.37.0_linux_386 文件夹 ls 看看里面的文件

```
cd frp_0.37.0_linux_386
```

然后输入 ifconfig 查看一下本机的 ip 地址，可以看到我们的内网 ip 是 172.21.28.176 一定要记住

```
ifconfig
```

来编辑一下 frps.ini 这个文件

```
vim frps.ini
```

按字母 i 启用编辑，然后删掉全部内容后将里面的配置文件编辑的跟我一样

```
[ common ]

\#bind_addr 是云主机的内网 ip

bind_addr= 172.21.28.176

\# 与客户端绑定端口

bind_port= 9654

\#dashboard 用户名

dashboard_user= admin

\#dashboard 密码

dashboard_pwd= admin

\#dashboard 端口，启动成功后可通过浏览器访问如 http://ip:9527

dashboard_port= 9527

\# 设置客户端 token，对应客户端有页需要配置一定要记住，如果客户端不填写你连不上服务端

token =8n262f2b-6daa-4g8d-85ee-8ad3d1x439a2
```

[![LaWgjs.jpg](https://s1.ax1x.com/2022/04/18/LaWgjs.jpg)](https://imgtu.com/i/LaWgjs)

按字母 `i`，`bind_addr = 你自己服务器的内网 IP`

编辑完后按键盘上的 esc 键，输入一个英文冒号下面会如图显示

![LaW63Q.jpg](https://s1.ax1x.com/2022/04/18/LaW63Q.jpg)](https://imgtu.com/i/LaW63Q)

实在不会就百度一下 vim 编辑器的使用方法

最后输入 `wq` 回车完成保存，刚刚编辑过的这些东西都要记住抄下来过一会配置客户端要用到

![LaW63Q.jpg](https://s1.ax1x.com/2022/04/18/LaW63Q.jpg)](https://imgtu.com/i/LaW63Q)

最后 `catfrps.ini` 这个文件看看保存的内容是否跟刚刚编辑的一致

[![LaWy9g.jpg](https://s1.ax1x.com/2022/04/18/LaWy9g.jpg)](https://imgtu.com/i/LaWy9g)

```
cat frps.ini
```

关闭防火墙和 selinux，图片里面的三条命令都要输入

[![LaWr4S.jpg](https://s1.ax1x.com/2022/04/18/LaWr4S.jpg)](https://imgtu.com/i/LaWr4S)

```
setenforce 0 
systemctl stop firewalld
systemctl disable firewalld
```

安装 screen

```
yum -y install screen
```

安装完成后输入 screen

```
screen
```

启动 frps

```
./frps -c ./frps.ini
```

可通过浏览器访问如 http:// 公网 ip:9527

输入用户名密码后能看到这个界面就说明搭建成功了

[![LaWDN8.jpg](https://s1.ax1x.com/2022/04/18/LaWDN8.jpg)](https://imgtu.com/i/LaWDN8)

### 三、配置客户端

这里以 windows 举例 , 如果是其他平台 frpc.ini 文件中的配置内容是一样的

进入这个[页面](https://github.com/fatedier/frp/releases)

下载画红线的这个安装包

[![LaWBAf.jpg](https://s1.ax1x.com/2022/04/18/LaWBAf.jpg)](https://imgtu.com/i/LaWBAf)

把这个包解压到 c 盘内会有如下几个文件

[![LaWdBt.jpg](https://s1.ax1x.com/2022/04/18/LaWdBt.jpg)](https://imgtu.com/i/LaWdBt)

`frpc.exe` 和 `frpc.ini` 是最核心的两个文件

编辑 `frpc.ini` 文件跟我的一样，这里举例映射内网本机的 3389 端口到外网的 4567 端口保存后退出

[![LaWanI.jpg](https://s1.ax1x.com/2022/04/18/LaWanI.jpg)](https://imgtu.com/i/LaWanI)

这里举例将 3389 映射为 4567 端口

#### 现在来编辑 windows 的启动脚本

空白处右键新建一个文本文档命名为 start 编辑完成后再改后缀

`start.txt`

打开后编辑如下内容

[![LaWNjA.jpg](https://s1.ax1x.com/2022/04/18/LaWNjA.jpg)](https://imgtu.com/i/LaWNjA)

保存后关闭修改文件的后缀名为 bat 如图所示

`start.bat`

双击就开始运行，但有个黑框框感觉非常丑而且会不小心关掉，但如果你设置了开机自启就不会有黑框框了

可以参考这个教程 https://blog.csdn.net/taw19960426/article/details/109366260

[![LaWwHP.jpg](https://s1.ax1x.com/2022/04/18/LaWwHP.jpg)](https://imgtu.com/i/LaWwHP)

最后再进入`http:// 服务器公网 ip:9527`这个网址

进入 Proxies 里面的 TCP 就可以看到你映射出去的端口了

[![LaWXHx.jpg](https://s1.ax1x.com/2022/04/18/LaWXHx.jpg)](https://imgtu.com/i/LaWXHx)

## 总结

frp 非常好用，服务器位于香港延迟是真的好低，给电脑开了远程配合安卓或苹果里面的 RD Client 这个软件就可以随时远程到家里面的电脑临时应急用一下啦

















