---
layout: post
title: "mysql参数优化"
date: 2013-10-13 18:55:28 +0800
comments: true
categories: 
---
<!--more-->
   <div class="Blog_wz1" style='word-wrap: break-word;'>

			Mysql参数优化对于新手来讲，是比较难懂的东西，其实这个参数优化，是个很复杂的东西，对于不同的网站，及其在线量，访问量，帖子数量，<span href="http://bbs.chinaunix.net/tag.php?name=%CD%F8%C2%E7" onclick="tagshow(event)" class="t_tag">网络</span>情况，以及机器硬件配置都有关系，优化不可能一次性

完成，需要不断的观察以及调试，才有可能得到最佳效果。<br>

<br>

　　下面先说我的<span href="http://bbs.chinaunix.net/tag.php?name=%B7%FE%CE%F1%C6%F7" onclick="tagshow(event)" class="t_tag">服务器</span>的硬件以及论坛情况，<br>

<br>

　　CPU: 2颗四核Intel Xeon 2.00GHz<br>

<br>

　　<span href="http://bbs.chinaunix.net/tag.php?name=%C4%DA%B4%E6" onclick="tagshow(event)" class="t_tag">内存</span>: 4GB DDR<br>

<br>

　　<span href="http://bbs.chinaunix.net/tag.php?name=%D3%B2%C5%CC" onclick="tagshow(event)" class="t_tag">硬盘</span>: SCSI 146GB<br>

<br>

　　论坛：在线会员 一般在 5000 人左右 - 最高记录是 13264.<br>

<br>

　　下面，我们根据以上硬件配置结合一份已经做过一次优化的my.cnf进行分析说明：有些参数可能还得根据论坛的变化情况以及程序员的程序进行再调整。<br>

<br>

　　[<span href="http://bbs.chinaunix.net/tag.php?name=mysql" onclick="tagshow(event)" class="t_tag">mysql</span>d]<br>

<br>

　　port = 3306<br>

<br>

　　<span href="http://bbs.chinaunix.net/tag.php?name=server" onclick="tagshow(event)" class="t_tag">server</span>id = 1<br>

<br>

　　socket = /tmp/mysql.sock<br>

<br>

　　skip-locking # 避免MySQL的外部锁定，减少出错几率增强稳定性。 skip-name-resolve<br>

<br>

　　禁止MySQL对外部连接进行<span href="http://bbs.chinaunix.net/tag.php?name=DNS" onclick="tagshow(event)" class="t_tag">DNS</span>解析，使用这一选项可以消除MySQL进行DNS

解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求!<br>

<br>

　　back_log = 500<br>

<br>

　　要求 MySQL 

能有的连接数量。当主要MySQL线程在一个很短时间内得到非常多的连接请求，这就起作用，然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。<br>

<br>

　　back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。只有如果期望在一个短时间内有很多连接，你需要增

加它，换句话说，这值对到来的TCP/IP连接的侦听队列的大小。你的操作<span href="http://bbs.chinaunix.net/tag.php?name=%CF%B5%CD%B3" onclick="tagshow(event)" class="t_tag">系统</span>在这个队列大小上有它自己的限制。试图设定

back_log高于你的操作系统的限制将是无效的。当你观察你的主机进程列表，发现大量 264084 | unauthenticated user

 | xxx.xxx.xxx.xxx | NULL | Connect | NULL | login | NULL 的待连接进程时，就要加大 

back_log 的值了。默认数值是50，我把它改为500。<br>

<br>

　　key_buffer_size = 384M<br>

<br>

　　# 

key_buffer_size指定用于索引的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)，到你能负担得起那样多。如果你使它太大，系

统将开始换页并且真的变慢了。对于内存在4GB左右的<span href="http://bbs.chinaunix.net/tag.php?name=%B7%FE%CE%F1" onclick="tagshow(event)" class="t_tag">服务</span>器该参数可设置为384M或512M。通过检查状态

值Key_read_requests和Key_reads,可以知道key_buffer_size设置是否合理。比例key_reads / 

key_read_requests应该尽可能的低，至少是1:100，1:1000更好(上述状态值可以使用SHOW STATUS LIKE 

‘key_read%’获得)。注意：该参数值设置的过大反而会是服务器整体效率降低!<br>

<br>

　　max_allowed_packet = 32M<br>

<br>

　　增加该变量的值十分安全，这是因为仅当需要时才会分配额外内存。例如，仅当你发出长查询或mysqld必须返回大的结果行时mysqld才会分配更多

内存。该变量之所以取较小默认值是一种预防措施，以捕获客户端和服务器之间的错误信息包，并确保不会因偶然使用大的信息包而导致内存溢出。<br>

<br>

　　table_cache = 512<br>

<br>

　　table_cache指定表高速缓存的大小。每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问

表内容。通过检查峰值时间的状态值Open_tables和Opened_tables，可以决定是否需要增加table_cache的值。如果你发现 

open_tables等于table_cache，并且opened_tables在不断增长，那么你就需要增加table_cache的值了(上述状

态值可以使用SHOW STATUS LIKE 

‘Open%tables’获得)。注意，不能盲目地把table_cache设置成很大的值。如果设置得太高，可能会造成<span href="http://bbs.chinaunix.net/tag.php?name=%CE%C4%BC%FE" onclick="tagshow(event)" class="t_tag">文件</span>描述符不足，从而造成性能不稳定或者连接失败。<br>

<br>

sort_buffer_size = 4M<br>

<br>

　　查询排序时所能使用的缓冲区大小。注意：该参数对应的分配内存是每连接独占!如果有100个连接，那么实际分配的总共排序缓冲区大小为100 × 4

 = 400MB。所以，对于内存在4GB左右的服务器推荐设置为4-8M。<br>

<br>

　　read_buffer_size = 4M<br>

<br>

　　读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每连接独享!<br>

<br>

　　join_buffer_size = 8M<br>

<br>

　　联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享!<br>

<br>

　　myisam_sort_buffer_size = 64M<br>

<br>

　　MyISAM表发生变化时重新排序所需的缓冲<br>

<br>

　　query_cache_size = 64M<br>

<br>

　　指定MySQL查询缓冲区的大小。可以通过在MySQL控制台执行以下命令观察：<br>

<br>

　　# &gt; SHOW VARIABLES LIKE '%query_cache%'; # &gt; SHOW STATUS LIKE 

'Qcache%'; # 如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况;<br>

<br>

　　如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓

冲;Qcache_free_blocks，如果该值非常大，则表明缓冲区中碎片很多。<br>

<br>

　　thread_cache_size = 64<br>

<br>

　　可以复用的保存在中的线程的数量。如果有，新的线程从缓存中取得，当断开连接的时候如果有空间，客户的线置在缓存中。如果有很多新的线程，为了提高性

能可以这个变量值。通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用<br>

<br>

　　tmp_table_size = 256M<br>

<br>

　　max_connections = 1000<br>

<br>

　　指定MySQL允许的最大连接进程数。如果在访问论坛时经常出现Too Many Connections的错误提示，则需要增大该参数值。<br>

<br>

　　max_connect_errors = 10000000<br>

<br>

　　对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接。如需对该主机进行解禁，执行：FLUSH HOST;。<br>

<br>

　　wait_timeout = 10<br>

<br>

　　指定一个请求的最大连接时间，对于4GB左右内存的服务器可以设置为5-10。<br>

<br>

　　thread_concurrency = 8<br>

<br>

　　该参数取值为服务器逻辑CPU数量×2，在本例中，服务器有2颗物理CPU，而每颗物理CPU又支持H.T超线程，所以实际取值为4 × 2 = 8<br>

<br>

　　skip-networking<br>

<br>

　　开启该选项可以彻底关闭MySQL的TCP/IP连接方式，如果WEB服务器是以远程连接的方式访问MySQL<span href="http://bbs.chinaunix.net/tag.php?name=%CA%FD%BE%DD" onclick="tagshow(event)" class="t_tag">数据</span>库服务器则不要开启该选项!否则将无法正常连接!<br>

<br>

　　long_query_time = 10<br>

<br>

　　log-slow-queries =<br>

<br>

　　log-queries-not-using-indexes<br>

<br>

　　开启慢查询日志( slow query log )<br>

<br>

　　慢查询日志对于跟踪有问题的查询非常有用。它记录所有查过long_query_time的查询，如果需要，还可以记录不使用索引的记录。下面是一个

慢查询日志的例子：<br>

<br>

　　开启慢查询日志，需要设置参数log_slow_queries、long_query_times、log-queries-not-using-

indexes。<br>

<br>

　　log_slow_queries指定日志文件，如果不提供文件名，MySQL将自己产生缺省文件名。long_query_times指定慢查询的

阈值，缺省是10秒。log-queries-not-using-indexes是4.1.0以后引入的参数，它指示记录不使用索引的查询。设置 

long_query_time=10<br>

<br>

<br>

　另外附上使用show status命令查看mysql状态相关的值及其含义：<br>

<br>

　　使用show status命令<br>

<br>

　　含义如下:<br>

<br>

　　aborted_clients 客户端非法中断连接次数<br>

<br>

　　aborted_connects 连接mysql失败次数<br>

<br>

　　com_xxx xxx命令执行次数,有很多条<br>

<br>

　　connections 连接mysql的数量<br>

<br>

　　Created_tmp_disk_tables 在磁盘上创建的临时表<br>

<br>

　　Created_tmp_tables 在内存里创建的临时表<br>

<br>

　　Created_tmp_files 临时文件数<br>

<br>

　　Key_read_requests The number of requests to read a key block from the 

cache<br>

<br>

　　Key_reads The number of physical reads of a key block from disk<br>

<br>

　　Max_u<span href="http://bbs.chinaunix.net/tag.php?name=sed" onclick="tagshow(event)" class="t_tag">sed</span>_connections 同时使用的连接数<br>

<br>

　　Open_tables 开放的表<br>

<br>

　　Open_files 开放的文件<br>

<br>

　　Opened_tables 打开的表<br>

<br>

　　Questions 提交到server的查询数<br>

<br>

　　Sort_merge_passes 如果这个值很大,应该增加my.cnf中的sort_buffer值<br>

<br>

　　Uptime 服务器已经工作的秒数<br>

<br>

　　提升性能的建议:<br>

<br>

　　1.如果opened_tables太大,应该把my.cnf中的table_cache变大<br>

<br>

　　2.如果Key_reads太大,则应该把my.cnf中key_buffer_size变大.可以用

Key_reads/Key_read_requests计算出cache失败率<br>

<br>

　　3.如果Handler_read_rnd太大,则你写的SQL语句里很多查询都是要扫描整个表,而没有发挥索引的键的作用<br>

<br>

　　4.如果Threads_created太大,就要增加my.cnf中thread_cache_size的值.可以用

Threads_created/Connections计算cache命中率<br>

<br>

　　5.如果Created_tmp_disk_tables太大,就要增加my.cnf中tmp_table_size的值,用基于内存的临时表代替基

于磁盘的

		

		

		           </div>

