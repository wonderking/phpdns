# PHPDNS
PHPDNS是基于DNSmasq开发的WEB界面

![](https://imgurl.org/upload/1807/7bb91bcc8bd6d328.png)

![](https://imgurl.org/upload/1807/1fc3a392274aaaf3.png)

### 环境要求
* CentOS 6/7
* PHP 5.6+(需要支持PDO组件)
* SQLite 3

### 阅读前准备
* 掌握Linux基础知识，熟悉Linux基本命令
* 熟悉网络基础

### 关于DNSmasq
DNSmasq是一个小巧且方便地用于配置DNS和DHCP的工具，适用于小型网络，它提供了DNS功能和可选择的DHCP功能。使用DNSmasq可以很方便的搭建递归DNS（公共DNS），如类似的`119.29.29.29`

### 自建DNS的好处
* 自定义DNS解析
* 屏蔽广告
* 防止DNS劫持

### 适用场景
* 适合公司、家庭等适量用户的小型网络

### 使用说明
请参考帮助文档： [https://doc.xiaoz.me/docs/phpdns/](https://doc.xiaoz.me/docs/phpdns/)

安装DNSmasq
DNSmasq配置本身已经很简单，不过为了方便还是写了一个一键脚本。

一键安装DNSmasq
环境要求：CentOS 6/7
先使用ifconfig命令查看服务器IP，并记录，比如下图中的192.168.0.4

执行下面的命令安装DNSmasq
#安装epel源
yum -y install epel-release
#安装DNSmasq
wget https://raw.githubusercontent.com/helloxz/dnsmasq/master/dns.sh --no-check-certificate
chmod +x dns.sh
#注意后面填写ifconfig看到的IP
./dns.sh 192.168.0.4
如果是阿里云等服务器，注意防火墙还要放行tcp/udp 53端口。输入netstat -apn|grep 'dnsmasq'可查看DNSmasq是否运行正常。
手动安装DNSmasq
需要手动安装DNSmasq请参考《Linux安装DNSmasq搭建自己的公共DNS》

常用命令
启动：service dnsmasq start
停止：service dnsmasq stop
重启：service dnsmasq restart

安装PHPDNS
PHPDNS是基于DNSmasq开发的WEB界面

运行原理
PHPDNS生成DNSmasq格式的配置文件
服务器crontab定时检测配置文件变化，若有改动则重启DNSmasq使其生效
环境要求
操作系统：Linux
PHP 5.6+（需要PDO组件支持）
SQLite 3
安装PHPDNS
访问master.zip下载最新源码，并解压到站点根目录，同时注意站点目录所属用户权限可读可写。（新手易犯此错误）
编辑application/helpers/check_helper.php设置用户名、密码，里面有注释说明。
访问您的域名http://domain.com/ 登录测试
Nginx伪静态设置
如果是Apache已经自带了.htaccess规则，无需额外设置。如果是Nginx请再server段内添加：

location ^~ /application {
        deny all;
}
location ^~ /system {
        deny all;
}
location ^~ /(application|system) {
        deny all;
}
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
添加完成后别忘记重载一次nginx

编写Shell脚本
PHPDNS通过shell脚本检测DNSmasq文件变化，使用vi reload.sh命令新建Shell脚本，并写入以下内容，路径请自行修改。

CentOS 7
#!/bin/bash
find /data/wwwroot/xxx.com/application/conf/ -name '*.conf' -mmin -1 -exec /usr/bin/systemctl restart dnsmasq.service {} \;
CentOS 6
#!/bin/bash
find /data/wwwroot/xxx.com/application/conf/ -name '*.conf' -mmin -1 -exec /sbin/service dnsmasq restart {} \;
/data/wwwroot/xxx.com/application/conf/是DNSmasq配置文件目录，改为自己的目录。
/usr/bin/systemctl是CentOS 7 systemctl的目录
/sbin/service是CentOS 6的service目录
别忘记赋予脚本执行权限：chmod +x reload.sh
设置crontab定时任务
#安装crontab
yum install crontabs
#新建定时任务
crontab -e
#写入下面的内容，注意路径
*/1 * * * * /root/shell/reload.sh
#重载crontab
service crond reload
/root/shell/reload.sh 是上面shell脚本的绝对路径，请注意修改。

建立软连接
软连接默认已经生成好了，直接登录PHPDNS后台，讲命令复制到Linux终端执行即可。



dig测试
dig 是一个 Linux 下用来 DNS 查询信息的工具，全称是Domain Information Groper，与 nslookup 类似，但比 nslookup 功能更强大。Windows 下只有 nslookup，如果也想用到 dig 命令，可以手动安装。

CentOS安装dig
直接yum搞定：

yum install bind-utils
Windows安装dig
请参考CentOS安装BIND-UTILS使用dig等命令
测试方法
进入PHPDNS后台首页可以看到您自己搭建的DNS IP，并记录下来。



假如我们的DNS IP是·119.29.29.29，输入命令：dig @119.29.29.29 www.xiaoz.me即可，如下截图。



@后面跟指定的DNS，上面是119.29.29.29您还可以改成114.114.114.114试试效果。



您可以让dig指定自己刚刚搭建的DNS，从而测试是否生效。


### 获取捐赠版
PHPDNS捐赠版功能上没有任何区别，PHPDNS捐赠版可提供首次安装调试。

![](https://imgurl.org/upload/1712/cb349aa4a1b95997.png)

### 联系我
* Blog:[https://www.xiaoz.me/](https://www.xiaoz.me/)
* QQ:337003006
