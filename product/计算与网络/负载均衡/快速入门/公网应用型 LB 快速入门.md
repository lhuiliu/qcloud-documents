腾讯云公网应用型负载均衡的推出，可以让用户配置基于域名/URL转发规则，将请求转发到不同的后端服务器中。此外，公网应用型负载均衡的重定向功能可以将http请求重定向为https请求，通过LoadBalance代理，使一些手机端的http请求自动返回https的respond。下面我们就通过一步步的配置，来验证一下公网应用型负载均衡的新功能。

## 1. 创建云主机，搭建nginx服务。

### 1.1 购买云主机
在云主机的[选购页面](https://buy.qcloud.com/cvm)选择适合自己的机型和镜像等，设置主机的初始密码，配置安全组（这里为了测试方便，可以先选择放通全部端口，后续再做限制）。另外，在购买云主机时注意开通公网流量，否则会导致后续关联LB后访问不通。

![](https://mc.qcloudimg.com/static/img/1ee252e5b02350e91e9aeec6bbf47e9c/001.png)

本次测试使用的云主机环境参数如下，共购买了两台：
>**主机信息**
地域	广州
可用区	广州三区
主机计费模式	按量计费
网络计费模式	按流量计费
所属网络	基础网络

>**机器配置**
操作系统	CentOS 6.8 64位
CPU	1核
内存	2GB
系统盘	20GB(云硬盘)	
数据盘	380GB(高性能云硬盘)
公网带宽	1Mbps
### 1.2 搭建环境
购买完成后，在云主机的详情页面，点击【登录】按钮，可以直接登录云主机，输入自己的用户名密码后，开始搭建nginx环境。这里采用了最简单的方式安装了nginx。如果需要安装最新版的nginx，可以去官网下载后上传解压安装。
安装nginx:
```
yum -y install nginx  
```
启动nginx，发现出现报错
```
service nginx start
```
修改配置文件
```
vim /etc/nginx/conf.d/default.conf

listen       80 default_server; 
listen       [::]:80 default_server;

修改为:
listen       80;   #侦听80端口
#listen       [::]:80 default_server;
```
重启nginx
```
sudo service nginx restart
```
现在访问该云服务器的公网ip地址，可以出现如下页面：

![](https://mc.qcloudimg.com/static/img/7bd527bd16e7a2c7a6f93d0e00e897ea/002.png)

nginx的默认根目录root在/usr/share/nginx/html位置，因此我们直接通过修改或移动html下的index.html静态页面，用来标识这个页面的特殊性。
```
vim /usr/share/nginx/html/index.html
```
由于应用型负载均衡可以根据后端服务器的路径进行请求转发，因此，在不同的路径下配置服务，可以便于后面负载均衡做请求分发。我们将分别在CVM1的/image/路径以及CVM2的/text/路径下部署静态页面，如下所示：

相关命令如下：
```
cd /usr/share/nginx/html/
mkdir image/
cp -r index.html image/    # 对另一台云主机 可以将页面部署到text路径下
```

### 1.3 验证服务
此时，通过访问云服务器的公网ip+路径，如果可以显示出您部署好的页面的话，证明第一步的部署成功。
CVM1 的/image页面

![](https://mc.qcloudimg.com/static/img/2e10d11bb030dc2627ade5656690a483/003.png)

CVM2 的/text页面

![](https://mc.qcloudimg.com/static/img/9f063edb7307936199eb44ba72958e8a/004.png)

## 2. 购买并配置公网应用型LB

### 2.1 购买应用型负载均衡
在负载均衡的[购买页](https://buy.qcloud.com/lb)选择公网应用型负载均衡。需要注意的是，选取某一地域的负载均衡后（如广州区域的LB），当前该负载均衡下只支持绑定同一地域、不同可用区的后端云服务器（支持绑定广州二区、广州三区的CVM）。创建完成后，就可以体验应用型LB丰富的能力了！

![](https://mc.qcloudimg.com/static/img/0dc79d07fec5a1998b98f37bd17334d5/005.png)

### 2.2 配置监听器、转发组和转发规则，绑定云主机
购买完成后，在【LB详情】-【监听器管理】页面，可以查看该LB实例绑定的监听器信息，单击【新建】创建一个HTTP监听器

![](https://mc.qcloudimg.com/static/img/847950e967c54f04d736f7cb7fac9c89/006.png)

创建七层HTTP监听器时，填写监听器名称和监听器监听的端口，这里我们默认填写了80端口。创建完成后，点击【创建转发规则】可以为监听器配置域名+URL，这里的域名和URL支持通配和正则，但存在一定的限制条件，详见[配置说明](https://www.qcloud.com/document/product/214/6744)。均衡方式可以选择按权重轮询的方式，如果不希望连接落到同一台后端云服务器时，可以在配置的第三步默认关闭会话保持。

![](https://mc.qcloudimg.com/static/img/effd493443791f91e88a5cb661ab0809/008.png)

创建完成后，可以看到该监听器下已经配置了www.example.com/image/的转发组和转发规则，接下来可以通过【绑定云主机】来选取我们刚才配置好服务的机器了。绑定云主机时，我们默认也监听了后端80端口。由于应用型负载均衡配置灵活，可以在同一监听器下绑定不同后端端口的云服务器。

![](https://mc.qcloudimg.com/static/img/7b48c5c5f1cd6c98605cadb6e99a1326/009.png)

之后，我们可以继续创建一个HTTPS监听器，在创建HTTPS监听器时，至少需要提供服务器证书来进行单向认证。这里我们可以通过自行上传证书、选取已有证书或在SSL证书平台侧申请证书来取得。HTTPS协议我们默认配置为443端口。

![](https://mc.qcloudimg.com/static/img/c20fae06728afae71c1692e7b2a3d200/010.png)

之后的监听器配置步骤类似，在配置完成后，我们可以看一下该LB下的结构图：

![](https://mc.qcloudimg.com/static/img/62c9e3c9d392673c8757abe70f65ea17/011.png)


### 2.2 验证服务
配置完成后，下面我们可以验证一下该架构是否生效。首先，我们需要对两个监听器的域名做hosts，在C:\Windows\System32\drivers\etc中，修改hosts文件，把LB实例的vip映射到两个域名上。

![](https://mc.qcloudimg.com/static/img/dbdf71ada5ab53a98150a1677e700770/012.png)

为了验证是否hosts成功，可以在本机开cmd 用ping命令探测一下该域名是否成功绑定了VIP，如果发现有数据包，则证明绑定成功。

![](https://mc.qcloudimg.com/static/img/8aa703e2da557010210f23b52da56f87/013.png)

接下来，可以分别输入http://www.example.com/image/和 https://www.example2.com/text/ 来测试请求是否能通过LB访问后端服务器。（注意，image/和text/后面的/很重要哦，因为这个代表了image和text是两个默认的目录，而不是名为image和text的文件）

![](https://mc.qcloudimg.com/static/img/82360f96ea78984030d7378b35ee48e0/014.png)

![](https://mc.qcloudimg.com/static/img/bd9bc8b9d925c8cb54740d448eead43c/015.png)

上图的结果表明，我们已经可以通过一个LB实例下不同的 ***域名+URL*** 访问不同的后端云服务器了，也就是实现了***“内容路由（content-based routing）”***的功能。那么接下来，如果遇到如下两个场景时，重定向功能就可以发挥其作用了：

1、强制https：PC、手机浏览器等以http请求访问web服务，希望LoadBalance代理后，返回https的respond。默认强制浏览器以https访问网页。

2、自定义重定向：当出现web业务需要临时下线（如电商售罄、页面维护，更新升级时）会需要重定向能力。如果不做重定向，用户的收藏和搜索引擎数据库中的旧地址只能让访客得到一个404/503错误信息页面降低了用户体验度，导致访问流量白白丧失。不仅如此，之前该页面积累的搜索引擎评分也浪费了。

接下来，我们可以通过实际操作来体验下该功能，将刚才配置好的HTTP监听器中请求，重定向到HTTPS监听器上。

## 3. 配置重定向功能

重定向配置分为手动重定向和自动重定向两种，自动重定向主要针对域名下路径较多的情况，需要系统自动为已经存在的HTTPS:443监听器创建HTTP监听器进行转发。创建成功后可以通过HTTP:80地址自动跳转为HTTPS:443地址进行访问。本次实践通过手动配置即可。更多内容可以通过[重定向配置](https://www.qcloud.com/document/product/214/8839)做进一步了解。

### 3.1 手动重定向配置
在LB详情也选取重定向配置，新建一个手动重定向配置

![](https://mc.qcloudimg.com/static/img/eeba873c140531e1555d5bd5b736325e/016.png)

之后分别选取原访问的协议、端口和域名，再指定目的协议、端口和域名。

![](https://mc.qcloudimg.com/static/img/b44d09f2ff05cd3ec2540098a534077a/017.png)

单击【下一步】后，可以选取原访问路径和重定向后的访问路径，域名下路径较多时，可以添加多条路径进行重定向，需要注意的是，路径的配置不允许回环（也就是A->B B->C的情况），且当前只允许在同一个LB实例中进行。

![](https://mc.qcloudimg.com/static/img/e5282652048b382b19c3278ade549255/018.png)

重定向策略配置完成后，可以在LB重定向配置详情页查看策略。此外，我们发现原有的监听器树状图中，HTTP监听器的路径下增加了一个重定向标识，用于说明，该路径下绑定的后端服务器将不会再收到请求，因为请求会被重定向到刚才配置的HTTPS监听器中。

![](https://mc.qcloudimg.com/static/img/17a214beb19fc8593e4efc14574761cf/019.png)


### 3.2 验证服务
最后一步，我们可以通过访问http://www.example.com/image/ 来验证，是否请求会被自动重定向到如下地址https://www.example2.com/text/
如果输入http://www.example.com/image/ 之后，出现如下页面，那么恭喜你，重定向配置也完成了！

![](https://mc.qcloudimg.com/static/img/591798ef620f8a72d9904197ca06c9a2/020.png)
