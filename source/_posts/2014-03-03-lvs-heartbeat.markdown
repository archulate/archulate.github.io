---
layout: post
title: "使用LVS+Heartbeat配置Linux群集"
date: 2014-02-03 19:10:02 +0800
comments: true
categories: 
---
<!--more-->
<p>【实验的基本环境】<br />

服务器系统：CentOS-5.6<br />

LVS主节点（lvs-master）：192.168.2.250(eth0) 心跳直连接口：192.168.1.250(eth1)<br />

LVS备节点（lvs-backup）：192.168.2.251(eth0) 心跳直连接口：192.168.1.251(eth1)<br />

Web Server-1：192.168.2.252<br />

Web Server-2：192.168.2.253<br />

VIP（虚拟IP）：192.168.2.254<br />

注：4台服务器已经配置好LNMP（Linux+Nginx+PHP(FastCGI模式)+MySQL）网站运行环境，当然使用Apache也可以。这里主备节点也当作真实服务器使用，所以也配置了网站环境。<br />

<br />

主备调度器eth1接口使用交叉线相连（理论上是这样）<br />

线序为： 一头为568A标准：白绿，绿；白橙，蓝；白蓝，橙；白棕，棕<br />

另一头为568B标准：白橙，橙；白绿，蓝；白蓝，绿；白棕，棕<br />

<br />

在LVS主节点和备节点的/etc/hosts中加入以下内容：<br />

#vim /etc/hosts<br />

192.168.2.250 lvs-master<br />

192.168.2.251 lvs-backup<br />

<br />

修改主机名：<br />

# vim /etc/sysconfig/network<br />

NETWORKING=yes<br />

NETWORKING_IPV6=no<br />

HOSTNAME=lvs-master<br />

GATEWAY=192.168.2.1<br />

<br />

# vim /etc/hosts<br />

# Do not remove the following line, or various programs<br />

# that require network functionality will fail.<br />

127.0.0.1 lvs-master localhost.localdomain localhost<br />

::1 localhost6.localdomain6 localhost6<br />

<br />

这2个文件都要修改，修改完后重启生效，备份机修改方法一样，不在重述。<br />

<br />

【下载软件】<br />

[libnet]<br />

wget http://search.cpan.org/CPAN/authors/id/G/GB/GBARR/libnet-1.22.tar.gz<br />

[ipvsadm]<br />

wget http://www.linuxvirtualserver.org/software/Kernel-2.6/ipvsadm-1.24.tar.gz<br />

[Heartbeat]<br />

wget http://hg.linux-ha.org/heartbeat-STABLE_3_0/archive/STABLE-3.0.4.tar.bz2<br />

[Cluster Glue]<br />

wget http://hg.linux-ha.org/glue/archive/glue-1.0.7.tar.bz2<br />

[Resource Agents]<br />

wget https://download.github.com/ClusterLabs-resource-agents-agents-1.0.4-0-gc06b6f3.tar.gz<br />

<br />

一、 配置LVS主节点（lvs-master）<br />

1. 安装libnet<br />

# tar zxvf libnet-1.1.2.2.tar.gz<br />

# cd libnet<br />

# ./configure<br />

# make &amp;&amp; make install<br />

# cd ..<br />

<br />

2. 安装ipvsadm<br />

# yum install kernel-devel //安装对应内核的kernel-devel<br />

# tar zxvf ipvsadm-1.24.tar.gz<br />

# cd ipvsadm-1.24<br />

# ln -s /usr/src/kernels/`ls /usr/src/kernels/` /usr/src/linux //将当前使用内核连接到/usr/src/linux<br />

# make &amp;&amp; make install<br />

# cd..<br />

<br />

3. 安装Heartbeat<br />

3.1 确认系统已经安装以下软件（系统光盘中有）<br />

libxslt、libxslt-devel、libgcrypt-devel、autoconf、automake、pkgconfig、 libgpg-error-devel、libtool、sgml-common、opensp、openjade、xml-common、 docbook-dtds、docbook-style<br />

如果在编译安装过程中出错，很有可能是因为缺少了相关的软件包<br />

<br />

3.2 安装glue<br />

# groupadd haclient<br />

# useradd -g haclient -M -s /sbin/nologin hacluster<br />

# tar jxvf glue-1.0.7.tar.bz2<br />

# cd Reusable-Cluster-Components-glue--glue-1.0.7/<br />

# ./autogen.sh<br />

# ./configure<br />

# make &amp;&amp; make install<br />

# cd ..<br />

<br />

3.3 安装 agents<br />

# tar zxvf ClusterLabs-resource-agents-agents-1.0.4-0-gc06b6f3.tar.gz<br />

# cd ClusterLabs-resource-agents-c06b6f3/<br />

# ./autogen.sh<br />

# ./configure<br />

# make &amp;&amp; make install<br />

# cd..<br />

<br />

3.4 安装Heartbeat<br />

# tar jxvf Heartbeat-3-0-STABLE-3.0.4.tar.bz2<br />

# cd Heartbeat-3-0-STABLE-3.0.4<br />

# ./ConfigureMe configure<br />

# gmake &amp;&amp; make install<br />

# cd..<br />

<br />

4. 配置lvs启动脚本<br />

本实验采用的是lvs-DR模式，该模式的特点是客户端的请求从主/备节点进入分配到web server，然后web server的响应是直接交付给客户端的。<br />

# vim /etc/init.d/lvs<br />

#!/bin/sh<br />

#chkconfig: 2345 20 80<br />

#description: start_lvs_of_dr<br />

VIP1=192.168.2.254<br />

RIP1=192.168.2.250<br />

RIP2=192.168.2.251<br />

RIP3=192.168.2.252<br />

RIP4=192.168.2.253<br />

./etc/rc.d/init.d/functions<br />

case &quot;$1&quot; in<br />

start)<br />

echo &quot;开启LVS DirectorServer...&quot;<br />

#设置虚拟IP地址<br />

#LVS启动时添加VIP的网口eth0:0<br />

/sbin/ifconfig eth0:0 $VIP1 broadcast $VIP1 netmask 255.255.255.255 up<br />

/sbin/route add -host $VIP1 dev eth0:0<br />

#清除IPVS表<br />

/sbin/ipvsadm -C<br />

#设置LVS<br />

/sbin/ipvsadm -A -t $VIP1:80 -s lc<br />

/sbin/ipvsadm -a -t $VIP1:80 -r $RIP1:80 -g<br />

/sbin/ipvsadm -a -t $VIP1:80 -r $RIP2:80 -g<br />

/sbin/ipvsadm -a -t $VIP1:80 -r $RIP3:80 &ndash;g<br />

/sbin/ipvsadm -a -t $VIP1:80 -r $RIP4:80 -g<br />

#使用ipvsadm来转发客户端请求。-s lc为最小连接数算法，-g是采用DR模式。有多少RIP就添加几条记录。<br />

#启动LVS<br />

/sbin/ipvsadm<br />

;;<br />

stop)<br />

echo &quot;停止LVS DirectorServer...&quot;<br />

#关闭时清除ipvsadm表<br />

/sbin/ipvsadm &ndash;C<br />

;;<br />

*)<br />

echo &quot;Usage: $0 {start|stop}&quot;<br />

exit 1<br />

esac<br />

<br />

注：这个脚本不要使用chkconfig管理，放入/etc/init.d内即可。<br />

<br />

5. 安装ldirectord相关组件<br />

默认安装完上面的三个软件包之后，ldirectord已经安装到系统中，默认路径在 /usr/etc/  ，由于ldirectord是由perl语言编写的，所以必须安装相关的软件包：Socket6、libwww-perl、URI、MailTools、 HTML-Parser<br />

# yum -y install perl-Socket6 perl-libwww-perl perl-URI perl-MailTools perl-HTML-Parser<br />

<br />

也可以在http://search.cpan.org/网站下载相关软件包安装<br />

安装方法：<br />

# perl Makefile.PL<br />

# make &amp;&amp; make install<br />

<br />

6. 配置heartbeat<br />

# cp -a /usr/etc/ha.d/ /etc/<br />

# rm -fr /usr/etc/ha.d/<br />

# ln -s /etc/ha.d/ /usr/etc/<br />

# cp /usr/share/doc/haresources /etc/ha.d/<br />

# cp /usr/share/doc/authkeys /etc/ha.d/<br />

# cp /usr/share/doc/ha.cf /etc/ha.d/<br />

# chmod 600 /etc/ha.d/authkeys //这个文件的权限必须是600<br />

<br />

6.1 修改配置文件<br />

6.1.1 authkeys文件配置（authkeys文件的作用是用来设置心跳信息的加密方式）<br />

vim /etc/ha.d/authkeys<br />

auth 1<br />

1 crc<br />

#2 sha1 HI!<br />

#3 md5 Hello!<br />

<br />

此设置是使用crc循环冗余校验，不采用加密的方式。<br />

<br />

6.1.2 ha.cf为heartbeat的主配置文件，修改下面配置。<br />

# vim /etc/ha.d/ha.cf<br />

#日志文件位置<br />

logfile /var/log/ha-log<br />

#指定主备服务器多久发送一次心跳<br />

keepalive 2<br />

#指定30秒没有收到对方心跳就认为对方已经down机<br />

deadtime 30<br />

#10秒没有收到心跳，便发出警报。<br />

warntime 10<br />

#对方DOWN后120秒重新检测一次。<br />

initdead 120<br />

#指定监听端口<br />

udpport 694<br />

#心跳监听网口，这里为eth1<br />

bcast eth1 //去掉后面#linux<br />

#备份机的心跳线接口与接口IP<br />

ucast eth1 192.168.1.251<br />

#主节点恢复后，自动收回资源。<br />

auto_failback on<br />

#指定主备服务器的主机名称，即在hosts文件中指定的。第一个node为主服务器，第二个node为备服务器。<br />

node lvs-master //服务器的主机名<br />

node lvs-backup<br />

#当192.168.2.1、192.168.2.2这两个IP都不能ping通时，对方即开始接管资源。<br />

ping_group group1 192.168.2.1 192.168.2.2<br />

#启用ipfail脚本<br />

respawn root /usr/lib/heartbeat/ipfail<br />

#指定运行ipfail的用户。<br />

apiauth ipfail gid=root uid=root<br />

<br />

6.1.3 haresources文件配置，这个文件是指定虚拟IP和改主机控制的脚本。<br />

# vim /etc/ha.d/haresources<br />

lvs-master 192.168.2.254 lvs ldirectord<br />

// master.lvs.net可为主节点主机名，192.168.2.254为虚拟IP<br />

<br />

6.1.4 ldirectord.cf是ldirectord进程的配置文件，该进程用来监视web server的运行状况，如果web server不能响应请求则把它排除在转发列表外。<br />

复制安装文件ldirectord目录上的ldirectord.cf 到/etc/ha.d/conf下，如果找不到可以查找一下：find / -name ldirectord.cf<br />

# mkdir /etc/ha.d/conf<br />

# cp ldirectord.cf /etc/ha.d/conf<br />

# vim /etc/ha.d/conf/ldirectord.cf<br />

# Global Directives<br />

#设置真实web server的超时时间<br />

checktimeout=30<br />

#监视真实web server的时间间隔<br />

checkinterval=10<br />

#如全部真实web server失败，则转发至本地<br />

fallback=127.0.0.1:80<br />

#改变配置文件内容，不需要重新ldirectord<br />

autoreload=yes<br />

#指定日志位置<br />

logfile=&quot;/var/log/ldirectord.log&quot;<br />

quiescent=no<br />

# A sample virual with a fallback that will override the gobal setting<br />

#指定虚拟IP<br />

virtual=192.168.2.254:80<br />

#指定真实web server IP及监听端口<br />

real=192.168.2.250:80 gate<br />

real=192.168.2.251:80 gate<br />

real=192.168.2.252:80 gate<br />

real=192.168.2.253:80 gate<br />

fallback=127.0.0.1:80 gate<br />

service=http<br />

#指定转发算法<br />

scheduler=lc //这里的算法要和LVS脚本的算法一样<br />

protocol=tcp<br />

#监视VIP服务器的方法<br />

checktype=negotiate<br />

checkport=80<br />

#监听测试页面名称，这个页面放入真实web server web服务的根目录<br />

request=&quot;lvs_testpage.html&quot;<br />

#指定测试页面返回内容<br />

receive=&quot;Test Page&quot;<br />

virtualhost= lvstest.net<br />

<br />

配置文件中的lvs_testpage.html必须存在网站根目录下，校验一下配置：<br />

# ldirectord -d /etc/ha.d/conf/ldirectord.cf start //按Ctrl+C结束<br />

# cp /etc/ha.d/shellfuncs /usr/lib/ocf/resource.d/heartbeat/.ocf-shellfuncs<br />

<br />

以上lvs和heartbeat配置完成。<br />

LVS备节点（lvs-backup）的配置和LVS主节点（lvs-master）完全一样。<br />

只是在/etc/ha.d/ha.cf中&ldquo;ucast eth1 192.168.1.251&rdquo;此配置地址不一样。<br />

<br />

二、配置真实web server脚本<br />

在每台web server的/etc/init.d目录下放置realserver脚本，这里主备节点同时也作为web server使用。<br />

# vim /etc/init.d/realserver<br />

#!/bin/bash<br />

# chkconfig: 2345 20 80<br />

# description: lvs_dr_realserver<br />

#指定虚拟IP<br />

VIP=192.168.2.254<br />

host=`/bin/hostname`<br />

case &quot;$1&quot; in<br />

start)<br />

# Start LVS-DR real server on this machine.<br />

/sbin/ifconfig lo down<br />

/sbin/ifconfig lo up<br />

#修改相关内核参数<br />

echo &quot;1&quot; &gt;/proc/sys/net/ipv4/conf/lo/arp_ignore<br />

echo &quot;2&quot; &gt;/proc/sys/net/ipv4/conf/lo/arp_announce<br />

echo &quot;1&quot; &gt;/proc/sys/net/ipv4/conf/all/arp_ignore<br />

echo &quot;2&quot; &gt;/proc/sys/net/ipv4/conf/all/arp_announce<br />

/sbin/ifconfig lo:0 $VIP netmask 255.255.255.255 up<br />

/sbin/route add -host $VIP dev lo:0<br />

;;<br />

stop)<br />

# Stop LVS-DR real server loopback device(s).<br />

/sbin/ifconfig lo:0 down<br />

;;<br />

status)<br />

# Status of LVS-DR real server.<br />

islothere=`/sbin/ifconfig lo:0 | grep $VIP`<br />

isrothere=`netstat -rn | grep &quot;lo&quot; | grep $VIP`<br />

if [ ! &quot;$islothere&quot; -o ! &quot;$isrothere&quot; ];<br />

then<br />

# Either the route or the lo:0 device<br />

# not found.<br />

echo &quot;LVS-DR real server Stopped.&quot;<br />

else<br />

echo &quot;LVS-DR Running.&quot;<br />

fi<br />

;;<br />

*)<br />

# Invalid entry.<br />

echo &quot;$0: Usage: $0 {start|status|stop}&quot;<br />

exit 1<br />

;;<br />

esac<br />

<br />

# chmod +x /etc/init.d/lvs<br />

# chmod +x /etc/init.d/realserver<br />

# service heartbeat start //主备LVS调度器上执行<br />

# /etc/init.d/lvs start //主备LVS调度器上执行<br />

# /etc/init.d/realserver start //真实web服务器上执行<br />

<br />

# chkconfig --level 35 heartbeat on<br />

# echo &ldquo;/etc/init.d/lvs start&rdquo; &gt;&gt; /etc/rc.local //开机启动<br />

# echo &ldquo;/etc/init.d/realserver start&rdquo; &gt;&gt; /etc/rc.local //开机启动<br />

<br />

三、测试<br />

配置已经测试过了，但是不同系统环境可能会出一些意料之外的事情。</p>

<div align="right">【责任编辑：<a class="ln" href="mailto:yangsai@51cto.com">枯木</a> TEL：（010）68476606】</div>

<p></p><br> 
<p class="blank10"></p> 
 
