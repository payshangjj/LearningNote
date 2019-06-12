# 1．Squid是什么? 

Squid是一种用来缓冲Internet数据的软件。它是这样实现其功能的，接受来自人们需要下载的目标（object）的请求并适当地处理这些请求。也就是说，如果一个人想下载一web页面，他请求Squid为他取得这个页面。Squid随之连接到远程[服务器](http://www.68idc.cn/)（比如：http：//squid.nlanr.net/）并向这个页面发出请求。然后，Squid显式地聚集数据到客户端机器，而且同时复制一份。当下一次有人需要同一页面时，Squid可以简单地从磁盘中读到它，那样数据迅即就会传输到客户机上。当前的Squid可以处理HTTP，FTP，GOPHER，SSL和WAIS等协议。但它不能处理如POP，NNTP，RealAudio以及其它类型的东西。

2.squid代理的作用：

![wKiom1M2ZDTieUYuAAGOJE2-l1k103.jpg](http://www.68idc.cn/help/uploads/allimg/140331/0QASH6_1.jpg "ee1.jpg")

3． 接下来我们主要探讨的是squid各种代理的定义

正向代理

a．标准的代理缓冲[服务器](http://www.68idc.cn/)

一个标准的代理缓冲服务被用于缓存静态的网页（例如：html文件和图片文件等）到本地网络上的一台主机上（即代理服务器）。当被缓存的页面被第二次访问的时候，浏览器将直接从本地代理服务器那里获取请求数据而不再向原web站点请求数据。这样就节省了宝贵的网络带宽，而且提高了访问速度。但是，要想实现这种方式，必须在每一个内部主机的浏览器上明确指明代理服务器的IP地址和端口号。客户端上网时，每次都把请求送给代理服务器处理，代理服务器根据请求确定是否连接到远程web服务器获取数据。如果在本地缓冲区有目标文件，则直接将文件传给用户即可。如果没有的话则先取回文件，先在本地保存一份缓冲，然后将文件发给客户端浏览器。

b．透明代理缓冲服务器

透明代理缓冲服务和标准代理服务器的功能完全相同。但是，代理操作对客户端的浏览器是透明的（即不需指明代理服务器的IP和端口）。透明代理服务器阻断网络通信，并且过滤出访问外部的HTTP（80端口）流量。如果客户端的请求在本地有缓冲则将缓冲的数据直接发给用户，如果在本地没有缓冲则向远程web服务器发出请求，其余操作和标准的代理服务器完全相同。对于Linux操作系统来说，透明代理使用Iptables或者Ipchains实现。因为不需要对浏览器作任何设置，所以，透明代理对于ISP（Internet服务器提供商）特别有用。

反向代理：

a. 反向代理缓冲服务器

反向代理是和前两种代理完全不同的一种代理服务。使用它可以降低原始WEB服务器的负载。反向代理服务器承担了对原始WEB服务器的静态页面的请求，防止原始服务器过载。它位于本地WEB服务器和Internet之间，处理所有对WEB服务器的请求，组织了WEB服务器和Internet的直接通信。如果互联网用户请求的页面在代理服务器上有缓冲的话，代理服务器直接将缓冲内容发送给用户。如果没有缓冲则先向WEB服务器发出请求，取回数据，本地缓存后再发送给用户。这种方式通过降低了向WEB服务器的请求数从而降低了WEB服务器的负载。

4．Squid主要组成部分

服务名：squid

主程序：/usr/sbin/squid

配置目录：/etc/squid

主配文件：/etc/squid/squid.conf

监听tcp端口号：3128

默认访问日志文件：/var/log/squid/access.log

5.squid 常用配置选项(/etc/squid/squid.conf)

http_port 3128 (还可以只监听一个一个ip http_port 192.168.0.1:3128)

cache_mem 64MB#缓存占内存大小

maximum_object_size 4096KB#最大缓存块

reply_body_max_size 1024000 allow all #限定下载文件大小

access_log /var/log/squid/access.log #访问日志存放的文件

visible_hostname proxy.test.com #可见的主机名

cache_dir ufs /var/spool/squid 100 16 256

usf:缓存数据的存储格式

/var/spool/squid 缓存目录

100 : 缓存目录占磁盘[空间](http://www.68idc.cn/)大小（M）

16 ：缓存[空间](http://www.68idc.cn/)一级子目录个数

256 ：缓存空间二级子目录个数

cache_mgr webmaster@test.com #定义管理员邮箱

http_access deny all #访问控制

6.squid中的访问控制

使用访问控制特性，可以控制在访问时根据特定的时间间隔进行缓存、访问特定站点或一组站点等等。 Squid 访问控制有两个要素：ACL 元素和访问列表。访问列表可以允许或拒绝某些用户对此服务的访问。

下面列出一些重要的 ACL 元素类型

\* src : 源地址 （即客户机IP地址）

\* dst : 目标地址 （即服务器IP地址）

\* srcdomain : 源名称 （即客户机名称）

\* dstdomain : 目标名称 （即服务器名称）

\* time : 一天中的时刻和一周内的一天

\* url_regex : URL 规则表达式匹配

\* urlpath_regex: URL-path 规则表达式匹配，略去协议和主机名

\* proxy_auth : 通过外部程序进行用户验证

\* maxconn : 单一 IP 的最大连接数

为了使用控制功能，必须先设置 ACL 规则并应用。ACL 声明的格式如下：

acl acl_element_name type_of_acl_element values_to_acl

注：

1. acl_element_name 可以是任一个在 ACL 中定义的名称。

2. 任何两个 ACL 元素不能用相同的名字。

3. 每个 ACL 由列表值组成。当进行匹配检测的时候，多个值由逻辑或运算连接；换言之，即任一 ACL bbs.bitsCN.com 元素的值被匹配，，则这个 ACL 元素即被匹配。

4. 并不是所有的 ACL 元素都能使用访问列表中的全部类型。

5. 不同的 ACL 元素写在不同行中，Squid 将把它们组合在一个列表中。

我们可以使用许多不同的访问条目。下面列出我们将要用到的几个：

\* http_access: 允许 HTTP 访问。这个是主要的访问控制条目。

\* no_cache: 定义对缓存请求的响应。

访问列表的规则由一些类似 'allow' 或 'deny' 的关键字构成，用以允许或拒绝向特定或一组 ACL 元素提供服务。

注：　

1. 这些规则按照它们的排列顺序进行匹配检测，一旦检测到匹配的规则，匹配检测就立即结束。

2. 一个访问列表可以有多条规则组成。

3. 如果没有任何规则与访问请求匹配，默认动作将与列表中最后一条规则对应。

4. 一个访问条目中的所有元素将用逻辑与运算连接：

http_access Action 声明1 AND 声明2 AND 声明 OR.

http_access Action 声明3

多个 http_access 声明间用或运算连接，但每个访问条目的元素间用与运算连接。

5. 请记住列表中的规则总是遵循由上而下的顺序。

7．下面我们来对squid几种代理进行简单配置：

标准的代理缓冲服务器的配置：

a.squid服务器上的配置

准备环境：软件包：squid（任意版本）

双网卡：eth0:192.168.1.1 eth1:10.106.34.12

如图：

![wKioL1M2ZCLBt_gEAACtrUug1mU730.jpg](http://www.68idc.cn/help/uploads/allimg/140331/0QASH6_0.jpg "ee2.png")

vi /etc/squid/squid.conf

http_port 192.168.1.1:3128 (可写多个)

cache_mem 64MB

maximum_object_size 4096KB

reply_body_max_size 1024000 allow all

access_log /var/log/squid/access.log

visible_hostname proxy.test.com

cache_mgr webmaster@test.com

http_access allow all

b.一切配置以后：

squid –z 初始化缓存

squid –k parse 检查语法

service squid start 启动squid

chkconfig squid on 加入开机启动

netstat –nltp 查看3128端口是否打开

c.客服端的配置：

ip : 192.168.1.12 gw:192.168.1.1

然后打开浏览器工具选项连接局域网设置代理服务器

地址：192.168.1.1 端口：3128

在浏览器输入http：//www.google.cn即可访问,上网了easy吧！

透明代理缓冲服务器的配置：

a. aquid服务器上的配置与标准的代理缓冲服务器几乎一样

差别就是：http_port 192.168.1.12:3128 transparent

b.添加iptables规则：

iptables -t nat -I PREROUTING -s 192.168.1.0/24 -p tcp -dport 80 -j REDIRECT --to-ports 3128

iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -p tcp --dport 53 -j SNAT -to-source 10.106.34.12

iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -p udp --dport 53 -j SNAT -to-source 10.106.34.12

squid –k parse

service squid reload

c.客服端不需要在浏览器中指定代理服务器的地址，端口

但需设置上网的DNS

好了经过上三个步骤你就可以上网了

反向代理缓冲服务器配置

注意：反向代理和透明代理不能同时使用

步骤：

a. Squid服务器的设置，修改/etc/squid/squid.conf

同样反向代理aquid服务器上的配置与标准的代理缓冲服务器几乎一样

不同之处：http_port 10.106.34.12：80 vhost

Cache_peer 192.168.1.12 parent 80 0 originserver weight=5 max-conn=30

上一行的解释：定义web服务器 web服务器地址 服务器类型 http端口 icp端口 [可选项]

squid –k parse

service squid reload

b. 客服端的设置（注意：这时的客服端就是web服务器）

开启web服务

好了通过以上配置外网即可访问你的web服务器了