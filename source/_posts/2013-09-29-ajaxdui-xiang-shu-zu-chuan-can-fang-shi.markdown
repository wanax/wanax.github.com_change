---
layout: post
title: "ajax对象数组传参方式"
date: 2012-10-11 15:53
comments: true
categories: Tec
---
项目中一个页面需要的功能是动态添加信息列表，传至后台添加数据库。用的方式是ajax+struts2，因为struts本身的特性，在ajax的data里直接写数组名java后台是无法正常接收到的，最简单的理解便是它无法智能的一一匹配数组里的每一个值，所以我们在前台开始发action请求前便要组装好容易接收的数据格式。

最简单的方法是直接拼接字符串，后台split出来，但有时候用的不是简单的字符串数组而是对象数组，这时候就需要一些其它方法了。

<!--more-->

![image](http://i1001.photobucket.com/albums/af134/mxiaochi/blogsource/1349941967_9520_zpsbbaf2803.png)

如上图所示，每次提交是权限一个对象，然后关于资源对象是动态添加与删除的，很明显，这就需要一个对象数组来装这些东西

前台页面的动态添加是这样的：

		var addNewSource = function(){  
		 var htmlStr =   
		  "<tr class=\"sourcecontent\">"+  
		           "<td><input class=\"sourcename\" type=\"text\"/></td><td><input class=\"resourceurl\" type=\"text\"/></td>"+  
		           "<td><input class=\"sourcetype\" type=\"text\"/></td><td><input class=\"resourcedes\" type=\"text\"/></td>"+  
		           "<td><a href=\"#\" onclick=\"deleteNewSource(this);return false;\"><img class=\"deleteBtn\" src=\"<%=basePath%>/img/delete.png\"/></a></td>"+  
		        "</tr>";  
		     $("#resourceInfo").append(htmlStr);  
		       
		};  
		function deleteNewSource(obj){  
		 $(obj).parents('tr').remove();  
		}  
		
然后就是对对象数组的拼装与发送了
		
		var xhrBeforeSend = function(){	
			$('.status').fadeIn();
		};
		
		var xhrSuccess = function(data,status,xhr){
			if(data.tag=="T"){
				$('.status').css({"margin-left":"740px","color":"green"}).html("提交数据成功");
				$('.status').fadeOut(1000);
			}else{
				$('.status').css({"margin-left":"760px","color":"#FF0000"}).html("提交失败！");
			}	  
		};
		KindEditor.ready(function(K) {
			   K('.submitBtn').click(function() {
							var screenWidth = $(document).width()-6;
							var sourceList = '<ul>';
							$("tr.sourcecontent").each(function(){
								sourceList +='<li>'+$(this).find('.sourcename').val()+'</li>';
							});
						    sourceList +='</ul>';
							var htmlStr = '<div class="dailogbox">'+
							               '<table style="width:400px; margin:10px auto;color:#ffffff;font-size:15px;">'+
							               '<tr><td>权限名</td><td>'+$("#authname").val()+'</td></tr>'+
							               '<tr><td>权限描述</td><td>'+$("#authdes").val()+'</td></tr>'+
							               '<tr><td>父级权限</td><td>'+$("#authparent").val()+'</td></tr>'+
							               '<tr><td></td><td></td></tr>'+
							               '<tr><td>关联权限:</td><td></td></tr>'+
							               '</table>'+
							               '<div style="width:400px;margin:10px auto; color:#ffffff;">'+sourceList+ 
							               '</div>'+
							               '</div>';
							var dialog = KindEditor.dialog({
								width : screenWidth,
								height: 300,
								body  : htmlStr,
								closeBtn : {
									name : '关闭',
									click : function(e) {
										dialog.remove();
									}
								},
								yesBtn : {
									name : '确定',
									click : function(e) {
									  var data = {
											  	  authName:$("#authname").val(),
								    	          authDes:$("#authdes").val(),
								    	          authParent:$("#authparent").val()
								       };
									  var n=0;
									  $("tr.sourcecontent").each(function(){
										  data["resList[" + n + "].resourceName"] = $(this).find('.sourcename').val();
										  data["resList[" + n + "].resourceType"] = $(this).find('.sourcetype').val();
										  data["resList[" + n + "].resourceDescription"] = $(this).find('.sourcedes').val();
										  data["resList[" + n + "].resourceURL"] = $(this).find('.sourceurl').val();
										  n++;
									  });
									  Pager.initUrl("insertauthority.action");
									  Pager.xhrBeforeSend=xhrBeforeSend;
									  Pager.xhrSuccess = xhrSuccess;
									  Pager.syncLoadData(data); 
									  dialog.remove();
									}
								},
								noBtn : {
									name : '取消',
									click : function(e) {
										dialog.remove();
									}
								}
							});
						});
		     });