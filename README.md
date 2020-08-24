![]( https://visitor-badge.glitch.me/badge?page_id=lmc999_add_route)
# auto-add-routes

## 介绍
为使用各类全局代理VPN的windows用户提供国内国外IP/域名的分流服务。

## 文件说明：

add.txt和del.txt为写入和删除时使用的路由表；

routes-up.bat和routes-down.bat为Tunsafe在连接前和断开后调用的写入/删除路由表的批处理文件。通过Tunsafe的PreUp和PostDown命令调用。

client_pre.bat和client_down.bat为Openvpn在连接前和断开后调用的写入/删除路由表的批处理文件。Openvpn连接时会自动调用。

cmroute.dll会被上述批处理文件调用，作用是秒载/秒删路由表。即使有数千条路由表也能秒载入，秒删除。

#### `2020-2-22更新：Tunsafe集成Overture DNS服务，国内IP/域名自动使用国内DNS解析；海外域名/IP使用国外DNS解析`
#### `2020-8-15更新：OpenVPN集成Overture DNS服务，国内IP/域名自动使用国内DNS解析；海外域名/IP使用国外DNS解析`
#### `2020-8-24更新：增加其它类型服务设置使用Overture DNS服务方法，和添加一个隐藏式启动Overture的vbs`
[Overture项目地址](https://github.com/shawn1m/overture)

Overture使用方法可以参考：https://moe.best/tutorial/overture.html

## 分流原理
[请参考wiki](https://github.com/lmc999/auto-add-routes/wiki)

## 使用方法

### Wireguard
#### 1. 下载最新版本Tunsafe，建议使用带rc的版本。

#### 2. 开启Tunsafe的Pre/Post命令功能。在"Option"选择"Allow Pre/Post Commands"

#### 3. 下载[route.zip](https://raw.githubusercontent.com/lmc999/auto-add-routes/master/wireguard/route.zip)解压到Tunsafe安装目录。

#### 4. Wireguard客户端配置文件加入PreUp,Postdown命令调用批处理文件。

假设你的Tunsafe安装在D盘abc目录下,你需要在客户端配置文件添加以下五条命令

PreUp = start D:\abc\TunSafe\route\routes-up.bat

PostUp = start D:\abc\TunSafe\route\dns-up.bat

PostDown = start D:\abc\TunSafe\route\routes-down.bat

PostDown = start D:\software\TunSafe\route\dns-down.bat

ExcludedIPs = 127.0.0.1/32

`然后修改配置文件的DNS地址为本机地址即：127.0.0.1`

设置实例请参考[sample.conf](https://raw.githubusercontent.com/lmc999/auto-add-routes/master/wireguard/sample.conf)

#### 5. 正常使用Tunsafe点击connect就会⑴调用routes-up.bat将国内IP写进系统路由表，⑵启动overture DNS服务

####    断开disconnect则会⑴调用routes-down.bat删除路由表，⑵关闭overture DNS服务

连接成功后可上 http://ip111.cn/ 测试自己的IP。

### Openvpn

#### 1. 下载[client.zip](https://raw.githubusercontent.com/lmc999/auto-add-routes/master/openvpn/client.zip)解压到OPENVPN的config文件夹中，需要确保解压出的文件与你的配置文件client.ovpn保存在同一目录中。

假如你的配置文件不是client.ovpn而是abc.ovpn，你需要将client_pre.bat和client_down.bat分别改名为abc_pre.bat和abc_down.bat，否则OPENVPN无法自动调用批处理文件。

#### 2. 添加以下参数到客户端配置文件client.ovpn
    pull-filter ignore "dhcp-option DNS"
    dhcp-option DNS 127.0.0.1

#### 3. OPENVPN点击Connect连接就会调用client_pre.bat将国内IP写进系统路由表，断开disconnect则会调用client_down.bat删除路由表。

### 其它服务

### 示例ExpressVPN。
由于ExpressVPN连接默认使用了自家DNS解析，所以需要以下设置

    1，在软件设计界面--Advanced--DNS--取消Only use ExpressVPN DNS servers while connected
    
    2，设置主网卡和虚拟网卡DNS，首选DNS服务器设置为127.0.0.1，备用留空
    
    3，设置主网卡和虚拟网卡跃点，网卡协议版本IPv4属性-高级-取消自动跃点，手动设置跃点，使主网卡跃点数＞虚拟网卡跃点数即可，如100>10
    
    4，手动使用routes-up.bat和routes-down.bat写入和删除路由表，在CMD、PowerShell等terminal中使用route print查看
    
    5，启用overture，然后连接，去ip111.cn等网站查看效果
    
### 由于写入的是活动路由，可能在某些情况如重启、变更网卡等操作后需要重新设置，可以通过route print查看路由表是否存在。
### 还有一种情况，Overture由于某些原因进程消失，可能是端口冲突问题，情况比较复杂，通常重新打开就可以解决，但频繁关闭就需要检查网络原因。
### overture.vbs应放在overture-windows-amd64.exe同级目录下，如果主程序名称发生改变，自行编辑修改。
    
## 关于分流后国内访问慢，无法播放网站版权视频/音乐

#### ~~因为你访问的国内网站有海外节点，当你使用WG/OPENVPN时DNS一般默认是8.8.8.8。这是一个海外的DNS，访问有海外节点的网站时会把你解析到海外节点，所以会被认为从大陆地区以外访问，这时候访问网站会变慢或者版权视频/音乐无法播放。解决办法是不要边用WG边上这些网站，这不是域名分流！~~

#### `配搭overture可实现访问国内网站用国内DNS，海外网站使用海外DNS`
