---
layout: post
title: " linux下关于mySQL的安装和配置 "
date: 2014-03-03 18:50:40 +0800
comments: true
categories: 数据库 
---
<!--more-->
一、引言 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 想使用Linux已经很长时间了，由于没有硬性任务一直也没'/>

<script language="javascript">

//用户是否在线

var isOnLine = '';

$(document).ready(function(){

	var blog = {'name': '', 'name_url': '', 'brief': ''};

	//消息通知显示和隐藏控制

	$('#show_message_slide_button').hover(

		function(){

			$('#message_slide_div').slideDown(100);											   

		},

		function(){

			

		}

	);



	$('#message_slide_div').hover(

		function(){

			

		},

		function(){

			$('#message_slide_div').slideUp(100);

		}

	);

	

	//编辑博客名

	$('#editbna').click(function(){

		blog.name = $('#bnaspan a').text();

		var val = '<input id="bnainput" type="text" style="float:left" value="" rel="' +$(this).attr('rel') + '" /><input id="bnasub" type="button" style="float:left" class="btn1"><input id="bnacanl" type="button" style="float:left" class="btn2"><div class="clear"></div>';

		$('#bnaspan').html(val);

		$('#bnainput').val(blog.name);

		$(this).parent().hide();

	});

	

	$('#bnasub').live('click', function(){

	    var rel = eval('({' + $('#bnainput').attr('rel') + '})');

		var name = $('#bnainput').val();

		if(name != blog.name){

	        $.ajax({

			    type: "POST",

			    url: rel.url,	

			    data: {

				    'name' : name

			    },

			    success:function(data){

				    if(data == 0){

						$('#bnaspan').html(blog.name);

						$('#bnaspan').html('<a href="' + rel.href + '">' + $('#bnaspan').html() + '</a>');

				    }else{

						$('#bnaspan').html(data);

						$('#bnaspan').html('<a href="' + rel.href + '">' + $('#bnaspan').html() + '</a>');

				    }

					$('#editbna').parent().show();

				}

			});

		}else{

		    $('#bnaspan').text(blog.name).html();

			$('#bnaspan').html('<a href="' + rel.href + '">' + $('#bnaspan').html() + '</a>');

			$('#editbna').parent().show();

		}

	});

	

	$('#bnacanl').live('click', function(){

		var rel = eval('({' + $('#bnainput').attr('rel') + '})');

		$('#bnaspan').html('<a href="' + rel.href + '">' + blog.name + '</a>');

		$('#editbna').parent().show();

	});

	

	//编辑签名

	$('#editbrief').click(function(){

	    blog.brief = $('#briefem').text();

		var val = '<input id="brfinput" type="text" style="float:left" value="" rel="' + $(this).attr('rel') + '" /><input id="brfsub" style="float:left"  type="button" class="btn1"><input style="float:left"  id="brfcanl" type="button" class="btn2"><div class="clear"></div>';

		$('#briefem').html(val);

		$('#brfinput').val(blog.brief);

		$(this).parent().hide();

	});

	

	$('#brfsub').live('click', function(){

	    var url = $('#brfinput').attr('rel');

		var brief = $('#brfinput').val();

		if(brief != blog.brief){

	        $.ajax({

			    type: "POST",

			    url: url,	

			    data: {

				    'brief' : brief

			    },

			    success:function(data){

				    if(data == 0){

				        $('#briefem').html(blog.brief);

				    }else{

						$('#briefem').html(data);

				    }

					$('#editbrief').parent().show();

				}

			});

		}else{

		    $('#briefem').text(blog.brief).html();

			$('#editbrief').parent().show();

		}

	});

	

	$('#brfcanl').live('click', function(){

		$('#briefem').html(blog.brief);

		$('#editbrief').parent().show();

	});



});

</script>

</head>

<body>

<div class="box">

  <!-- 一级导航 -->

  <div class="Blog_nav1">

    <div class="Blog_nav1_2"><a href="/"><img src="/image/default/1.png"></a><a href="http://www.chinaunix.net" class="Blog_a1">Chinaunix首页</a>　| 　<a href="http://bbs.chinaunix.net" target="_blank">论坛</a>　| 　<a href="http://ask.chinaunix.net" target="_blank">问答</a>　| 　<a href="http://blog.chinaunix.net" target="_blank">博客</a><span class="Blog_span1"></span>

            <a href="/site/login.html" class="Blog_a1">登录</a> | <a href="http://u.it168.com/Register?webid=5&returnUrl=http%3A%2F%2Fblog.chinaunix.net%2Fuid-20019131-id-1987421.html" class="Blog_a1">注册</a>

          </div>

	<!--自动提示层-->

	<style>

	.bor13221{border:1px #bbb solid;width:206px;position:absolute;top:34px;left:0;background:#fff; z-index:9999;display:none}

	.bor13221 li{height:26px;line-height:26px;padding-left:6px;color:#555;font-size:14px;cursor:pointer;}

	.here{background:#f3f3f3;}

    </style>



	<!--自动提示层-->

    <div class="Blog_nav1_3" style="position:relative; z-index:9999;">

	 <div class="bor13221">

      <ul>

      </ul>

    </div>

	  <form action='/site/search.html' method='post'>

		<input type="text"  autocomplete="off"  class="Blog_txt1" id='search_input_id' name='keywords'>

		<select class="Bolg_sel1" name='type' id='search_type_blog'>

		  <option value='blog'>博文</option>

		  <option value='author'>博主</option>

		</select>

		<input type="submit" value='' name='submit' class="Blog_btn1">

	 </form>

    </div>

    <div class="clear"></div>

    <div class="Blog_nav1_layer1" id="message_slide_div" style="display:none;">

	    <ul>

	    	<li><a href="/message/private.html">私人消息()</a></li>

	    	<li><a href="/message/system.html">系统消息()</a></li>

	    	<li><a href="/member/request.html">好友请求()</a></li>

	    	<li><a href="/member/notification.html">通知管理()</a></li>

	    </ul>

    </div>

  </div>

   <script type="text/javascript">

  	$(function(){

		//点击添加进文本框

		$(".bor13221 li").live( 'click' , function(e){

			if ( e && e.stopPropagation )

			{

				//因此它支持W3C的stopPropagation()方法

				e.stopPropagation();

			}

			else

			{

				//否则，我们需要使用IE的方式来取消事件冒泡

				window.event.cancelBubble = true; 

			}

			$('#search_input_id').val($(this).text());

			$(".bor13221 ul").html('');

			$(".bor13221").hide();

		});

		$(".bor13221 ul li").live({

			mouseenter:

			function()

			{

				$(".bor13221 ul li").removeClass("here");

				$(this).addClass('here');

			},

			mouseleave:

			function()

			{

				$(".bor13221 ul li").removeClass("here");

				$(this).removeClass('here');

			}

		});

		//自动提示

		$('#search_input_id').keyup(function(event){

			//取消博主的提示

			var search_type_blog = $('#search_type_blog').val();

			if(search_type_blog == 'author') return false;



			var key = $(this).val();

			//获取键值

			var keycode = event.which; //38 上 40 下

			var count = $('.bor13221 ul li').length;

			if(key != '' && keycode != 38 && keycode != 40)

			{

				$.getJSON("http://api.sou.it168.com/autoWenKuCloud?jsoncallback=?",{"ty":"json","offset":"0","limit":"10","q":key}, function(result)

					{

						var arr = result.data;



						var html ='';

						for (i=0;i<arr.length ;i++ )   

						{   

							html += '<li>'+arr[i]+'</li>';

						} 

						

						$('.bor13221 ul').html(html);

						(arr.length > 1) ?  $(".bor13221").show() : $(".bor13221").hide();

					}

				);

			}

			else if(keycode == 38)

			{

				if(count > 0)

				{

					//遍历li

					var curr_li_num;

					$('.bor13221 ul li').each(function(index , dom){

						if($(dom).attr('class') == 'here')

						{

							curr_li_num = index;

							return false;

						}

					}); 

					var next_li_num;

					if(typeof(curr_li_num) == 'undefined')

					{

						next_li_num = count - 1;

					}

					else

					{

						if(curr_li_num == 0)

						{

							next_li_num = count - 1;

						}

						else

						{

							next_li_num = curr_li_num - 1;

						}

					}

					$(".bor13221 ul li").removeClass("here");

					$(".bor13221 ul li:eq(" + next_li_num + ")").addClass("here");

					$('#search_input_id').val($(".bor13221 ul li:eq(" + next_li_num + ")").text());

				}

			}

			else if(keycode == 40)

			{

				if(count > 0)

				{

					//遍历li

					var curr_li_num;

					$('.bor13221 ul li').each(function(index , dom){

						if($(dom).attr('class') == 'here')

						{

							curr_li_num = index;

							return false;

						}

					}); 

					var next_li_num;

					if(typeof(curr_li_num) == 'undefined')

					{

						next_li_num = 0;

					}

					else

					{

						if(curr_li_num == count - 1)

						{

							next_li_num = 0;

						}

						else

						{

							next_li_num = curr_li_num + 1;

						}

					}

					$(".bor13221 ul li").removeClass("here");

					$(".bor13221 ul li:eq(" + next_li_num + ")").addClass("here");

					$('#search_input_id').val($(".bor13221 ul li:eq(" + next_li_num + ")").text());

				}

			}

		});

		$(document).click(function(e){

			$(".bor13221").hide();

		});



	});

  </script>

  <!-- 头 -->

  <!-- 推荐博客-->

  <div class="Blog_header1">

	    <div class="Blog_header1_1">

      <p class="Blog_p1" ><em><a href="/uid/20019131.html">运维 虚拟化 XEN 安全 linux IT管理</a></em>      <p class="Blog_p2" style="color:#125A94">我的CIO之路</p>

    </div>

        <div class="Blog_header1_2" id="hide_div1">

    	<span class="Blog_span3"></span>

    	<div class="float_div1" style="white-space:nowrap;" onmouseover="javascript:isMove=false" onmouseout="javascript:isMove=true">

	    <ul id="noticev2">  

		    		    <li><a href="http://blog.chinaunix.net/uid-24789255-id-4019728.html" target="_blank">【原创评选】12-02月原创博文评选</a></li>

		    		    <li><a href="http://blog.chinaunix.net/uid-24789255-id-3938601.html" target="_blank">2013第三季度“ChinaUnix博客之星”评选</a></li>

		    		    <li><a href="http://dtcc.it168.com/" target="_blank">2014中国数据库技术大会</a></li>

		    	    </ul>

	    </div>

    </div>

            <div class="Blog_header1_3"><a href="/uid/20019131.html">首页</a>　| 　<a href="/uid/20019131/abstract/1.html">博文目录</a>　| 　<a href="/member/profile/uid/20019131.html">关于我</a></div>

  </div>

    

  <!-- 内容部分 -->

  	<script type="text/javascript" src="/highlight/scripts/XRegExp.js"></script> <!-- XRegExp is bundled with the final shCore.js during build -->

<script type="text/javascript" src="/highlight/scripts/shCore.js"></script>

<script type="text/javascript" src="/highlight/scripts/shAutoloader.js"></script>

<link type="text/css" rel="stylesheet" href="/highlight/styles/shCore.css"/>

<link type="text/css" rel="Stylesheet" href="/highlight/styles/shThemeDefault.css" />

<link href="/code/css/fck_editorarea.css" rel="stylesheet" type="text/css" />

  <div class="Blog_contain"> 

    <!-- 左 -->

	<script language="javascript">

$(document).ready(function(){

	$('#ConcernBtn').bind('click',function(){

			var cuid = '20019131';

			var url =  '/member/concern.html';

			var type = $(this).attr('rel');

		

			if(type == 'addConcern'){

				$.ajax({

					type : 'get',

					url  : url,

					data : {'op' : 'ajaxadd' , 'cuid' : cuid, 'random' : Math.random()},

					success : function(msg){	

					   if(msg == -1){

						   showErrorMsg('参数错误！');

					   } else if (msg == 0){

						   showErrorMsg('关注失败，没有该用户！');

					   } else if (msg == 1){

						   showErrorMsg('关注失败，您已经关注了该用户！');

					   } else if (msg == 2){

						   $('#ConcernBtn').val('已关注');

						   $('#ConcernBtn').attr('rel','delConcern');

						   showSucceedMsg('关注成功!');

					   } else if (msg == 3){

						   showErrorMsg('未知错误');

					   }

					}

				});	

			} else if ( type == 'delConcern'){

				$.ajax({

					type : 'get',

					url  : url,

					data : {'op' : 'ajaxdel' , 'cuid' : cuid, 'random' : Math.random()},

					success : function(msg){

					   if(msg == 0){

						   showErrorMsg('参数错误！','消息提示');

					   } else if (msg == 1){

						   showErrorMsg('操作失败，请尝试刷新页面重试！','消息提示');

					   } else if (msg == 2){

						   $('#ConcernBtn').val('加关注');

						   $('#ConcernBtn').attr('rel','addConcern');

						   showSucceedMsg('成功取消关注！','消息提示');

					   } else if (msg == 3){

						   showErrorMsg('未知错误！','消息提示'); 

					   }

					}

				});	

			}

	});					   

});



//加好友

function addFriend(fuid, url){

	if(fuid == '' || fuid.length == 0){

		showErrorMsg('缺少参数！','信息提示');

		return false;

	}

	$.ajax({

		   type : 'get',

		   url : url,

		   data : {'op' : 'add', 'fuid' : fuid , 'random' : Math.random()},

		   success : function(msg){

				if(msg == -1){

					showErrorMsg('参数错误！','消息提示');

				} else if (msg == -2){

					showErrorMsg('添加好友失败,没有该用户的信息！','消息提示');

				} else if (msg == -3){

					showErrorMsg('添加好友失败,你不能添加自己为好友！','消息提示');

				} else if (msg == -4){

					showErrorMsg('添加好友未知错误,该错误已被记录！','消息提示');

				} else if (msg == -5){

					showErrorMsg('添加好友失败,你之前已经发送过好友请求,请耐心等待对方同意申请！','消息提示');

				} else if (msg == -6){

					showErrorMsg('添加好友失败,你们已经是好友了！','消息提示');

				} else {

					$.cover(true);

					asyncbox.open({

						id : 'addFriend',

						title : '添加好友',

						url : url,

						data : {'op' : 'add', 'fuid' : fuid , 'random' : Math.random()},

						width : 490,

						height : 180,

						scroll : 'no',

						callback : function(action) {

							if (action == 'close'){

								$.cover(false);

							}	

						}

					});	

				}

		   }

	});	

	

}



//发送短消息

function postMessage(msguid, url){

	if(msguid == '' || msguid.length == 0){

		showErrorMsg('缺少参数！','信息提示');

		return false;

	}

	

	$.ajax({

		   type : 'post',

		   url : url,

		   data : {'op' : 'ajaxpost', 'msguid' : msguid , 'random' : Math.random()},

		   success : function(msg){

				if(msg == -1){

					showErrorMsg('发送失败，缺少收件人对象！','消息提示');

				} else if(msg == -2){

					showErrorMsg('发送失败，自己不能给自己发送短消息！','消息提示');

				} else {

					$.cover(true);

					asyncbox.open({

						id : 'postMessage',

						title : '发送短消息',

						url : url,

						data : {'op' : 'ajaxpost', 'msguid' : msguid , 'random' : Math.random()},

						width : 510,

						height : 255,

						scroll : 'no',

						callback : function(action) {

							if (action == 'close'){

								$.cover(false);

							}	

						}

					});	

				}

		   }

	});	

}



</script>

<div class="Blog_left">

      <div class="Blog_left1 Blog_bg1">

        <div class="Blog_left1_1">

			<!-- 专家博客-->

			<a href="/uid/20019131.html"><img src="http://passport.ixpub.net/data/avatar/020/01/91/31_avatar_middle.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_middle.gif'" /></a>

            <p><a href="/uid/20019131.html">solaris小兵</a></p>

        </div>

        <ul class="Blog_ul1 Blog_noline1">

          <li>博客访问： 306993 </li>

          <li>博文数量： 142 </li>

          <li>博客积分： 7242 </li>

          <!--<li>专家积分： 180</li>-->

          <li>博客等级： 少将 </li>

		  <li>技术积分： 1430 </li>

          <li>用  户  组：  普通用户</li>

          <li>注册时间： 2005-06-06 14:59 </li>

                  </ul>  





        

                <div class="HT_line3 HT_line3_1"></div>

        <ul class="Blog_ul2">

          <li><input type="button" value="加关注" id="ConcernBtn" onclick="showErrorMsg('操作失败,您需要先登录!', '消息提示', '/site/login.html')"></li>

          <li><input type="button" value="短消息" id="postMessageBtn" onclick="showErrorMsg('操作失败,您需要先登录!', '消息提示', '/site/login.html')"></li>

          <li><input type="button" value="论坛" onclick="location.href='http://bbs.chinaunix.net'"></li>

          <li><input type="button" value="加好友" id="addFriendBtn" onclick="showErrorMsg('操作失败,您需要先登录!', '消息提示', '/site/login.html')"></li>

        </ul>

              </div>

        

         

      <div class="Blog_left2 Blog_bg1">

        <div class="Blog_tit1">文章分类</div>

        <div class="Blog_left2_1">

          <p class="Blog_p4"><a href="/uid/20019131/list/1.html">全部博文</a>（142）</p>

          <ul id="blogCla">

                            <li><span class="Blog_jia1"></span><a href="/uid/20019131/cid-49302-list-1.html" title="win">win</a>（4）

                                    <div style="display:none;" class="zk">

					                        <p><a href="/uid/20019131/sid-49303-list-1.html" title="windows 虚拟化">windows 虚拟化</a>（3）</p>

                                        </div>

                                </li>

                            <li><a href="/uid/20019131/cid-49301-list-1.html" title="篮球NBA">篮球NBA</a>（4）

                                </li>

                            <li><a href="/uid/20019131/cid-49297-list-1.html" title="IT管理 ITSM ITIL">IT管理 ITSM ITIL</a>（21）

                                </li>

                            <li><a href="/uid/20019131/cid-49300-list-1.html" title="性能测试">性能测试</a>（5）

                                </li>

                            <li><a href="/uid/20019131/cid-49299-list-1.html" title="mysql">mysql</a>（8）

                                </li>

                            <li><a href="/uid/20019131/cid-49298-list-1.html" title="memcache">memcache</a>（5）

                                </li>

                            <li><a href="/uid/20019131/cid-49296-list-1.html" title="虚拟化">虚拟化</a>（17）

                                </li>

                            <li><a href="/uid/20019131/cid-49295-list-1.html" title="安全">安全</a>（10）

                                </li>

                            <li><a href="/uid/20019131/cid-49294-list-1.html" title="nagios">nagios</a>（5）

                                </li>

                            <li><a href="/uid/20019131/cid-49293-list-1.html" title="jboss">jboss</a>（6）

                                </li>

                            <li><a href="/uid/20019131/cid-49292-list-1.html" title="weblogic">weblogic</a>（1）

                                </li>

                            <li><a href="/uid/20019131/cid-49289-list-1.html" title="suse">suse</a>（0）

                                </li>

                            <li><a href="/uid/20019131/cid-49288-list-1.html" title="solaris">solaris</a>（1）

                                </li>

                            <li><a href="/uid/20019131/cid-49291-list-1.html" title="linux">linux</a>（38）

                                </li>

                            <li><a href="/uid/20019131/cid-49290-list-1.html" title="心情随想">心情随想</a>（16）

                                </li>

                            <li><a href="/uid/20019131/cid--1-list-1.html" title="未分配的博文">未分配的博文</a>（1）

                                </li>

                      </ul>

        </div>

      </div>

      	              <div class="Blog_left2 Blog_bg1">

        <div class="Blog_tit1">文章存档</div>

        <div class="Blog_left2_1" id="blogdtr">

                    <p class="Blog_p4"><span class="Blog_jia1"></span><a href="/uid/20019131/year-2011-list-1.html">2011年</a>（1）</p>

          <ul style="display:none;" class="Blog_ul3 zk">

                        <li><a href="/uid/20019131/year-201107-list-1.html">2011年07月</a>（1）</li>

                      </ul>

                    <p class="Blog_p4"><span class="Blog_jia1"></span><a href="/uid/20019131/year-2010-list-1.html">2010年</a>（12）</p>

          <ul style="display:none;" class="Blog_ul3 zk">

                        <li><a href="/uid/20019131/year-201010-list-1.html">2010年10月</a>（1）</li>

                        <li><a href="/uid/20019131/year-201009-list-1.html">2010年09月</a>（1）</li>

                        <li><a href="/uid/20019131/year-201006-list-1.html">2010年06月</a>（3）</li>

                        <li><a href="/uid/20019131/year-201005-list-1.html">2010年05月</a>（2）</li>

                        <li><a href="/uid/20019131/year-201003-list-1.html">2010年03月</a>（2）</li>

                        <li><a href="/uid/20019131/year-201002-list-1.html">2010年02月</a>（2）</li>

                        <li><a href="/uid/20019131/year-201001-list-1.html">2010年01月</a>（1）</li>

                      </ul>

                    <p class="Blog_p4"><span class="Blog_jia1"></span><a href="/uid/20019131/year-2009-list-1.html">2009年</a>（9）</p>

          <ul style="display:none;" class="Blog_ul3 zk">

                        <li><a href="/uid/20019131/year-200909-list-1.html">2009年09月</a>（1）</li>

                        <li><a href="/uid/20019131/year-200908-list-1.html">2009年08月</a>（2）</li>

                        <li><a href="/uid/20019131/year-200901-list-1.html">2009年01月</a>（6）</li>

                      </ul>

                    <p class="Blog_p4"><span class="Blog_jia1"></span><a href="/uid/20019131/year-2008-list-1.html">2008年</a>（115）</p>

          <ul style="display:none;" class="Blog_ul3 zk">

                        <li><a href="/uid/20019131/year-200812-list-1.html">2008年12月</a>（16）</li>

                        <li><a href="/uid/20019131/year-200811-list-1.html">2008年11月</a>（10）</li>

                        <li><a href="/uid/20019131/year-200810-list-1.html">2008年10月</a>（3）</li>

                        <li><a href="/uid/20019131/year-200809-list-1.html">2008年09月</a>（7）</li>

                        <li><a href="/uid/20019131/year-200808-list-1.html">2008年08月</a>（28）</li>

                        <li><a href="/uid/20019131/year-200807-list-1.html">2008年07月</a>（51）</li>

                      </ul>

                    <p class="Blog_p4"><span class="Blog_jia1"></span><a href="/uid/20019131/year-2006-list-1.html">2006年</a>（3）</p>

          <ul style="display:none;" class="Blog_ul3 zk">

                        <li><a href="/uid/20019131/year-200604-list-1.html">2006年04月</a>（3）</li>

                      </ul>

                    <p class="Blog_p4"><span class="Blog_jia1"></span><a href="/uid/20019131/year-2005-list-1.html">2005年</a>（2）</p>

          <ul style="display:none;" class="Blog_ul3 zk">

                        <li><a href="/uid/20019131/year-200507-list-1.html">2005年07月</a>（1）</li>

                        <li><a href="/uid/20019131/year-200506-list-1.html">2005年06月</a>（1）</li>

                      </ul>

                  </div>

      </div>

      	  	        <div class="Blog_left2 Blog_bg1">

        <div class="Blog_tit1">我的朋友</div>

        <ul class="Blog_ul4">

                    <li><a href="/uid/7944836.html"><img src="http://passport.ixpub.net/data/avatar/007/94/48/36_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/7944836.html" title="send_linux">send_lin</a></p>

          </li>

                    <li><a href="/uid/792340.html"><img src="http://passport.ixpub.net/data/avatar/000/79/23/40_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/792340.html" title="forrestrun">forrestr</a></p>

          </li>

                    <li><a href="/uid/14174896.html"><img src="http://passport.ixpub.net/data/avatar/014/17/48/96_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/14174896.html" title="ahjobusb">ahjobusb</a></p>

          </li>

                    <li><a href="/uid/20012471.html"><img src="http://passport.ixpub.net/data/avatar/020/01/24/71_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/20012471.html" title="linuxe">linuxe</a></p>

          </li>

                    <li><a href="/uid/20511150.html"><img src="http://passport.ixpub.net/data/avatar/020/51/11/50_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/20511150.html" title="liang3391">liang339</a></p>

          </li>

                  </ul>

      </div>

	  	        <div class="Blog_left2 Blog_bg1">

        <div class="Blog_tit1">最近访客</div>

        <ul class="Blog_ul4">

                    <li><a href="/uid/29483716.html"><img src="http://passport.ixpub.net/data/avatar/029/48/37/16_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/29483716.html" title="liucencen">liucence</a></p>

          </li>

                    <li><a href="/uid/28336510.html"><img src="http://passport.ixpub.net/data/avatar/028/33/65/10_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/28336510.html" title="ydjh460">ydjh460</a></p>

          </li>

                    <li><a href="/uid/10363961.html"><img src="http://passport.ixpub.net/data/avatar/010/36/39/61_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/10363961.html" title="cybermerman">cybermer</a></p>

          </li>

                    <li><a href="/uid/16587062.html"><img src="http://passport.ixpub.net/data/avatar/016/58/70/62_avatar_small.jpg" onerror="this.onerror=null;this.src='http://passport.ixpub.net/images/noavatar_small.gif'" /></a>

            <p><a href="/uid/16587062.html" title="siyan019">siyan019</a></p>

          </li>

                  </ul>

      </div>

	        <div class="Blog_left2 Blog_bg1">

        <div class="Blog_tit1">订阅</div>

        <ul class="Blog_ul5">

          <li><a href="/blog/rss/uid/20019131.html" class="Blog_a4"></a></li>

          <li><a href="http://www.google.com/ig/add?feedurl=http%3A%2F%2Fblog.chinaunix.net%2Fuid%2F20019131.html" class="Blog_a5"></a></li>

        </ul>

      </div>

      <div class="Blog_left2 Blog_left3 Blog_bg1">

        <div class="Blog_tit1">推荐博文</div>

        <ul class="Blog_ul6">

				  			<li>·<a href="/uid-10661836-id-4124968.html" title="详解Percona的XtraBackup备份工具(下篇)">详解Percona的XtraBackup备份...</a></li>

		  			<li>·<a href="/uid-22312037-id-4126734.html" title="HBase初探之安装与配置">HBase初探之安装与配置...</a></li>

		  			<li>·<a href="/uid-22788794-id-4126665.html" title="yii2权限(RBAC)">yii2权限(RBAC)</a></li>

		  			<li>·<a href="/uid-25472509-id-4126539.html" title="[PB]数据窗口自动换行">[PB]数据窗口自动换行...</a></li>

		  			<li>·<a href="/uid-29068468-id-4126424.html" title="nginx变量 ">nginx变量 </a></li>

		  		        </ul>

      </div>

      <div class="Blog_left2 Blog_left3 Blog_bg1">

        <div class="Blog_tit1">热词专题</div>

        <ul class="Blog_ul6">

                        <li >·<a href="/zt/1009/linuxftpjujiang_1009741.shtml" target='blank' title='linux---tftp设置'>linux---tftp设置</a></li>

                        <li >·<a href="/zt/1050/suselinuxhongfuraidgong_1050177.shtml" target='blank' title='RAID组中的“Foreign”状态磁盘'>RAID组中的“Foreign”状态磁...</a></li>

                        <li >·<a href="/zt/1042/windowsduoha_1042227.shtml" target='blank' title='Windows服务编程'>Windows服务编程</a></li>

                        <li >·<a href="/zt/1037/undotbsjin_1037965.shtml" target='blank' title='如果undo表空间undotbs不能释放空间,重建之'>如果undo表空间undotbs不能释...</a></li>

                        <li class="no_line1">·<a href="/zt/1006/ubuntu1110qq_1006698.shtml" target='blank' title='ubuntu11.10 安装mkimage工具'>ubuntu11.10 安装mkimage工具...</a></li>

                  </ul>

      </div>

	  



	</div>

<script language="javascript">

$(document).ready(function(){

    /*目录树JS效果*/

	$('#blogCla li > span').click(function(){

		var cla = $(this).attr('class');

		if(cla == 'Blog_jia1'){

			//$('#blogCla li > span').removeClass('Blog_jian1').addClass('Blog_jia1');

			//$('#blogCla li > .zk').css('display', 'none');

				

			$(this).removeClass('Blog_jia1').addClass('Blog_jian1');

            $(this).parent().children('.zk').css('display', 'block');

		}else{

			$(this).removeClass('Blog_jian1').addClass('Blog_jia1');

            $(this).parent().children('.zk').css('display', 'none');

		}

	});

		

	$('#blogdtr > p > span').click(function(){

		var cla = $(this).attr('class');

		if(cla == 'Blog_jia1'){

			//$('#blogdtr > .Blog_p4 > span').removeClass('Blog_jian1').addClass('Blog_jia1');

			//$('#blogdtr ul').css('display', 'none');

				

			$(this).removeClass('Blog_jia1').addClass('Blog_jian1');

            $(this).parent().next().css('display', 'block');

		}else{

			$(this).removeClass('Blog_jian1').addClass('Blog_jia1');

            $(this).parent().next().css('display', 'none');

		}

	});

});

</script>    <!-- 右 -->

    <div class="Blog_right1">

      <div class="Blog_right1_1 Blog_right1_11">

        <div class="Blog_right1_2 ">

			<!--推荐博文-->

          <div class="Blog_tit4 Blog_tit5">

                        <b class="Blog_b1"></b>

            <a href="/uid-20019131-id-1987421.html">linux下关于mySQL的安装和配置</a>

            <em>2008-07-28 14:03:56</em>

          </div>

          <div class="Blog_con2">

            <div class="Blog_con3">

              <p>分类： <span>LINUX</span></p>

			                </p>

            </div>

           <div class="Blog_wz1" style='word-wrap: break-word;'>

			<DIV>linux下关于mySQL的安装和配置 </DIV>

<DIV><BR>这篇博文很经典！推荐给大家！</DIV>

<DIV>一、引言 <BR>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 想使用Linux已经很长时间了，由于没有硬性任务一直也没有系统学习，近日由于工作需要必须使用Linux下的MySQL。本以为有Windows下使用SQL Server的经验，觉得在</DIV>

<DIV>Linux下安装MySql应该是易如反掌的事，可在真正安装和使用MySQL时走了很多弯路，遇见很多问题，毕竟Linux 和Windows本身就有很大区别。为了让和我一样的初学者在学习的</DIV>

<DIV>过程中少走弯路，尽快入门，写了此文，希望对您有所帮助。本文的Linux环境是 Red Hat 9.0，MySQL是4.0.16。</DIV>

<DIV>二、安装Mysql <BR>　　 1、下载MySQL的安装文件 <BR>　　 安装MySQL需要下面两个文件： <BR>　　 MySQL-server-4.0.16-0.i386.rpm　　　 <BR>　　 MySQL-client-4.0.16-0.i386.rpm <BR>　　 下载地址为：<A href="http://www.mysql.com/downloads/mysql-4.0.html">http://www.mysql.com/downloads/mysql-4.0.html</A>，打开此网页，下拉网页找到“Linux x86 RPM downloads”项，找到“Server”和“Client programs”项</DIV>

<DIV>，下载需要的上述两个rpm文件。 </DIV>

<DIV>　　 2、安装MySQL <BR>　　 rpm文件是Red Hat公司开发的软件安装包，rpm可让Linux在安装软件包时免除许多复杂的手续。该命令在安装时常用的参数是 –ivh ,其中i表示将安装指定的rmp软件包，V</DIV>

<DIV>表示安装时的详细信息，h表示在安装期间出现“#”符号来显示目前的安装过程。这个符号将持续到安装完成后才停止。<BR>&nbsp;<BR>1）安装服务器端 <BR>　　 在有两个rmp文件的目录下运行如下命令： <BR>　　 [root@test1 local]# rpm -ivh MySQL-server-4.0.16-0.i386.rpm <BR>　　 显示如下信息。 <BR>　　　 warning: MySQL-server-4.0.16-0.i386.rpm: V3 DSA signature: NOKEY, key ID 5072e1f5 <BR>　　 Preparing...　　　　　　　########################################### [100%] <BR>　　 1:MySQL-server　　　　　########################################### [100%] <BR>　　　 。。。。。。（省略显示） <BR>　　 /usr/bin/mysqladmin -u root password 'new-password' <BR>　　 /usr/bin/mysqladmin -u root -h test1 password 'new-password' <BR>　　　 。。。。。。（省略显示） <BR>　　 Starting mysqld daemon with databases from /var/lib/mysql <BR>　　 如出现如上信息，服务端安装完毕。测试是否成功可运行netstat看Mysql端口是否打开，如打开表示服务已经启动，安装成功。Mysql默认的端口是3306。<BR>&nbsp;<BR>[root@test1 local]# netstat -nat <BR>　　 Active Internet connections (servers and established) <BR>　　 Proto Recv-Q Send-Q Local Address　　　　　 Foreign Address　　　　 State　　　 <BR>　　 tcp　　0　　0 0.0.0.0:3306　　　　 0.0.0.0:*　　　　　 LISTEN　　　 <BR>　　 上面显示可以看出MySQL服务已经启动。 <BR>　　 2）安装客户端 <BR>　　 运行如下命令： <BR>　　 [root@test1 local]# rpm -ivh MySQL-client-4.0.16-0.i386.rpm <BR>　　 warning: MySQL-client-4.0.16-0.i386.rpm: V3 DSA signature: NOKEY, key ID 5072e1f5 <BR>　　 Preparing...　　　　########################################### [100%] <BR>　　 1:MySQL-client　 ########################################### [100%] <BR>　　 显示安装完毕。 <BR>　　 用下面的命令连接mysql,测试是否成功。</DIV>

<DIV>三、登录MySQL </DIV>

<DIV>　　 登录MySQL的命令是mysql， mysql 的使用语法如下： <BR>　　 mysql [-u username] [-h host] [-p[password]] [dbname] <BR>　　 username 与 password 分别是 MySQL 的用户名与密码，mysql的初始管理帐号是root，没有密码，注意：这个root用户不是Linux的系统用户。MySQL默认用户是root，由于</DIV>

<DIV>初始没有密码，第一次进时只需键入mysql即可。 <BR>　　 [root@test1 local]# mysql <BR>　　 Welcome to the MySQL monitor.　Commands end with ; or \g. <BR>　　 Your MySQL connection id is 1 to server version: 4.0.16-standard <BR>　　 Type 'help;' or '\h' for help. Type '\c' to clear the buffer. <BR>　　 mysql&gt; <BR>　　 出现了“mysql&gt;”提示符，恭喜你，安装成功！ <BR>　　 增加了密码后的登录格式如下： <BR>　　 mysql -u root -p <BR>　　 Enter password: (输入密码) <BR>　　 其中-u后跟的是用户名，-p要求输入密码，回车后在输入密码处输入密码。 </DIV>

<DIV>　　 注意：这个mysql文件在/usr/bin目录下，与后面讲的启动文件/etc/init.d/mysql不是一个文件。</DIV>

<DIV>四、MySQL的几个重要目录 </DIV>

<DIV>　　 MySQL安装完成后不象SQL Server默认安装在一个目录，它的数据库文件、配置文件和命令文件分别在不同的目录，了解这些目录非常重要，尤其对于Linux的初学者，因为 </DIV>

<DIV>Linux本身的目录结构就比较复杂，如果搞不清楚MySQL的安装目录那就无从谈起深入学习。 </DIV>

<DIV>　　 下面就介绍一下这几个目录。 </DIV>

<DIV>　　 1、数据库目录 <BR>　　 /var/lib/mysql/ </DIV>

<DIV>　　 2、配置文件 <BR>　　 /usr/share/mysql（mysql.server命令及配置文件） </DIV>

<DIV>　　 3、相关命令 <BR>　　 /usr/bin(mysqladmin mysqldump等命令) </DIV>

<DIV>　　 4、启动脚本 <BR>　　 /etc/rc.d/init.d/（启动脚本文件mysql的目录）</DIV>

<DIV>五、修改登录密码 </DIV>

<DIV>　　 MySQL默认没有密码，安装完毕增加密码的重要性是不言而喻的。 <BR>　　 1、命令 <BR>　　 usr/bin/mysqladmin -u root password 'new-password' <BR>　　 格式：mysqladmin -u用户名 -p旧密码 password 新密码 <BR>　　 2、例子 <BR>　　 例1：给root加个密码123456。 <BR>　　 键入以下命令 ： <BR>　　 [root@test1 local]# /usr/bin/mysqladmin -u root password 123456 <BR>　　 注：因为开始时root没有密码，所以-p旧密码一项就可以省略了。</DIV>

<DIV>3、测试是否修改成功 <BR>　　 1）不用密码登录 <BR>　　 [root@test1 local]# mysql <BR>　　 ERROR 1045: Access denied for user: <A href="mailto:'root@localhost'">'root@localhost'</A> (Using password: NO) <BR>　　 显示错误，说明密码已经修改。 <BR>　　 2）用修改后的密码登录 <BR>　　 [root@test1 local]# mysql -u root -p <BR>　　 Enter password: (输入修改后的密码123456) <BR>　　 Welcome to the MySQL monitor.　Commands end with ; or \g. <BR>　　 Your MySQL connection id is 4 to server version: 4.0.16-standard <BR>　　 Type 'help;' or '\h' for help. Type '\c' to clear the buffer. <BR>　　 mysql&gt; <BR>　　 成功！ <BR>　　 这是通过mysqladmin命令修改口令，也可通过修改库来更改口令。</DIV>

<DIV><BR>六、启动与停止 </DIV>

<DIV>　　 1、启动 <BR>　　 MySQL安装完成后启动文件mysql在/etc/init.d目录下，在需要启动时运行下面命令即可。 <BR>　　 [root@test1 init.d]# /etc/init.d/mysql start </DIV>

<DIV>　　 2、停止 <BR>　　 /usr/bin/mysqladmin -u root -p shutdown </DIV>

<DIV>　　 3、自动启动 <BR>　　 1）察看mysql是否在自动启动列表中 <BR>　　 [root@test1 local]#　/sbin/chkconfig –list <BR>　　 2）把MySQL添加到你系统的启动服务组里面去 <BR>　　 [root@test1 local]#　/sbin/chkconfig　– add　mysql <BR>　　 3）把MySQL从启动服务组里面删除。 <BR>　　 [root@test1 local]#　/sbin/chkconfig　– del　mysql<BR>&nbsp;<BR>七、更改MySQL目录 </DIV>

<DIV>　　 MySQL默认的数据文件存储目录为/var/lib/mysql。假如要把目录移到/home/data下需要进行下面几步： </DIV>

<DIV>　　 1、home目录下建立data目录 <BR>　　 cd /home <BR>　　 mkdir data </DIV>

<DIV>　　 2、把MySQL服务进程停掉： <BR>　　 mysqladmin -u root -p shutdown </DIV>

<DIV>　　 3、把/var/lib/mysql整个目录移到/home/data <BR>　　 mv /var/lib/mysql　/home/data/ <BR>　　 这样就把MySQL的数据文件移动到了/home/data/mysql下 </DIV>

<DIV>　　 4、找到my.cnf配置文件 <BR>　　 如果/etc/目录下没有my.cnf配置文件，请到/usr/share/mysql/下找到*.cnf文件，拷贝其中一个到/etc/并改名为my.cnf)中。命令如下： <BR>　　 [root@test1 mysql]# cp /usr/share/mysql/my-medium.cnf　/etc/my.cnf </DIV>

<DIV>　　 5、编辑MySQL的配置文件/etc/my.cnf <BR>　　 为保证MySQL能够正常工作，需要指明mysql.sock文件的产生位置。 修改socket=/var/lib/mysql/mysql.sock一行中等号右边的值为：/home/mysql/mysql.sock 。操作如下</DIV>

<DIV>： <BR>　　 vi　 my.cnf　　　 (用vi工具编辑my.cnf文件，找到下列数据修改之) <BR>　　 # The MySQL server <BR>　　　 [mysqld] <BR>　　　 port　　　= 3306 <BR>　　　 #socket　 = /var/lib/mysql/mysql.sock（原内容，为了更稳妥用“#”注释此行） <BR>　　　 socket　 = /home/data/mysql/mysql.sock　　　（加上此行） </DIV>

<DIV>　　 6、修改MySQL启动脚本/etc/rc.d/init.d/mysql <BR>　　 最后，需要修改MySQL启动脚本/etc/rc.d/init.d/mysql，把其中datadir=/var/lib/mysql一行中，等号右边的路径改成你现在的实际存放路径：home/data/mysql。 <BR>　　 [root@test1 etc]# vi　/etc/rc.d/init.d/mysql <BR>　　 #datadir=/var/lib/mysql　　　　（注释此行） <BR>　　 datadir=/home/data/mysql　　 （加上此行） </DIV>

<DIV>　　 7、重新启动MySQL服务 <BR>　　 /etc/rc.d/init.d/mysql　start <BR>　　 或用reboot命令重启Linux <BR>　　 如果工作正常移动就成功了，否则对照前面的7步再检查一下。<BR>&nbsp; <BR>八、MySQL的常用操作 </DIV>

<DIV>　　 注意：MySQL中每个命令后都要以分号；结尾。 </DIV>

<DIV>　　 1、显示数据库 <BR>　　 mysql&gt; show databases; <BR>　　 +----------+ <BR>　　 | Database | <BR>　　 +----------+ <BR>　　 | mysql　　| <BR>　　 | test　　 | <BR>　　 +----------+ <BR>　　 2 rows in set (0.04 sec) <BR>　　 Mysql刚安装完有两个数据库：mysql和test。mysql库非常重要，它里面有MySQL的系统信息，我们改密码和新增用户，实际上就是用这个库中的相关表进行操作。 </DIV>

<DIV>　　 2、显示数据库中的表 <BR>　　 mysql&gt; use mysql; （打开库，对每个库进行操作就要打开此库，类似于foxpro ） <BR>　　 Database changed </DIV>

<DIV>　　 mysql&gt; show tables; <BR>　　 +-----------------+ <BR>　　 | Tables_in_mysql | <BR>　　 +-----------------+ <BR>　　 | columns_priv　　| <BR>　　 | db　　　　　　　| <BR>　　 | func　　　　　　| <BR>　　 | host　　　　　　| <BR>　　 | tables_priv　　 | <BR>　　 | user　　　　　　| <BR>　　 +-----------------+ <BR>　　 6 rows in set (0.01 sec) </DIV>

<DIV>　　 3、显示数据表的结构： <BR>　　 describe 表名; </DIV>

<DIV>　　 4、显示表中的记录： <BR>　　 select * from 表名; <BR>　　 例如：显示mysql库中user表中的纪录。所有能对MySQL用户操作的用户都在此表中。 <BR>　　 Select * from user; </DIV>

<DIV>　　 5、建库： <BR>　　 create database 库名; <BR>　　 例如：创建一个名字位aaa的库 <BR>　　 mysql&gt; create databases aaa; <BR>6、建表： <BR>　　 use 库名； <BR>　　 create table 表名 (字段设定列表)； <BR>　　 例如：在刚创建的aaa库中建立表name,表中有id(序号，自动增长)，xm（姓名）,xb（性别）,csny（出身年月）四个字段 <BR>　　 use aaa; <BR>　　 mysql&gt; create table name (id int(3) auto_increment not null primary key, xm char(,xb char(2),csny date); <BR>　　 可以用describe命令察看刚建立的表结构。 <BR>　　 mysql&gt; describe name; </DIV>

<DIV>　　 +-------+---------+------+-----+---------+----------------+ <BR>　　 | Field | Type　　| Null | Key | Default | Extra　　　　　| <BR>　　 +-------+---------+------+-----+---------+----------------+ <BR>　　 | id　　| int(3)　|　　　| PRI | NULL　　| auto_increment | <BR>　　 | xm　　| char( | YES　|　　 | NULL　　|　　　　　　　　| <BR>　　 | xb　　| char(2) | YES　|　　 | NULL　　|　　　　　　　　| <BR>　　 | csny　| date　　| YES　|　　 | NULL　　|　　　　　　　　| <BR>　　 +-------+---------+------+-----+---------+----------------+ </DIV>

<DIV>　　 7、增加记录 <BR>　　 例如：增加几条相关纪录。 <BR>　　 mysql&gt; insert into name values('','张三','男','1971-10-01'); <BR>　　 mysql&gt; insert into name values('','白云','女','1972-05-20'); <BR>　　 可用select命令来验证结果。 <BR>　　 mysql&gt; select * from name; <BR>　　 +----+------+------+------------+ <BR>　　 | id | xm　 | xb　 | csny　　　 | <BR>　　 +----+------+------+------------+ <BR>　　 |　1 | 张三 | 男　 | 1971-10-01 | <BR>　　 |　2 | 白云 | 女　 | 1972-05-20 | <BR>　　 +----+------+------+------------+ </DIV>

<DIV>　　 8、修改纪录 <BR>　　 例如：将张三的出生年月改为1971-01-10 <BR>　　 mysql&gt; update name set csny='1971-01-10' where xm='张三'; </DIV>

<DIV>　　 9、删除纪录 <BR>　　 例如：删除张三的纪录。 <BR>　　 mysql&gt; delete from name where xm='张三'; </DIV>

<DIV>　　 10、删库和删表 <BR>　　 drop database 库名; <BR>　　 drop table 表名；</DIV>

<DIV> <BR>九、增加MySQL用户 </DIV>

<DIV>　　 格式：grant select on 数据库.* to 用户名@登录主机 identified by "密码" <BR>例1、增加一个用户user_1密码为123，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用以root用户连入MySQL，然后键入以下命令： </DIV>

<DIV>　　 mysql&gt; grant select,insert,update,delete on *.* to <A href='mailto:user_1@"%'>user_1@"%</A>" Identified by "123"; <BR>例1增加的用户是十分危险的，如果知道了user_1的密码，那么他就可以在网上的任何一台电脑上登录你的MySQL数据库并对你的数据为所欲为了，解决办法见例2。 </DIV>

<DIV>　　例2、增加一个用户user_2密码为123,让此用户只可以在localhost上登录，并可以对数据库aaa进行查询、插入、修改、删除的操作（localhost指本地主机，即MySQL数据库所</DIV>

<DIV>在的那台主机），这样用户即使用知道user_2的密码，他也无法从网上直接访问数据库，只能通过 MYSQL主机来操作aaa库。 </DIV>

<DIV>　　 mysql&gt;grant select,insert,update,delete on aaa.* to <A href="mailto:user_2@localhost">user_2@localhost</A> identified by "123"; </DIV>

<DIV>　　 用新增的用户如果登录不了MySQL，在登录时用如下命令： </DIV>

<DIV>　　 mysql -u user_1 -p　-h 192.168.113.50　（-h后跟的是要登录主机的ip地址）</DIV>

<DIV><BR>十、备份与恢复 </DIV>

