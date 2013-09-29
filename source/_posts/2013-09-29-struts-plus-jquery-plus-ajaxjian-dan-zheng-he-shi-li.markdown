---
layout: post
title: "Struts+jquery+ajax简单整合示例"
date: 2012-02-23 21:28
comments: true
categories: Tec
---
jsp文件：

	function clickButton()  
	            {      
	                $.ajax({  
	                   url :'../ajaxRequest.action',  //后台处理程序  
	                   type:'post',    //数据发送方式  
	                   dataType:'json',  //接受数据格式  
	                   data:{result:"RE",request:"REQ"},  //要传递的数据  
	                   success:callbackFun  //回传函数(这里是函数名)  
	                   });  
	             }   
	//url：响应aciton；params：传入参数；callbackFun：响应完成后的回调函数；  
	            function callbackFun(data)  
	            {  
	                 alert("SUCCESS");  
	                 alert(data.result);  
	                 alert(data.request);  
	                 alert(data.q[1]);  
	            }
	            
<!--more-->	            
或者：
	
	function clickButton()  
	            {      
	                var url = '../ajaxRequest.action';  
	                var params = {result:"RESULT",request:"REQUEST" }; //通过id获得输入值  
	                 jQuery.post(url, params, callbackFun, 'json');  
	               }   
	//url：响应aciton；params：传入参数；callbackFun：响应完成后的回调函数；  
	            function callbackFun(data)  
	            {  
	                 alert("SUCCESS");  
	                 alert(data.result);  
	                 alert(data.request)  
	            } 
	            
html部分：

	<input type="button" onclick="clickButton();" value="clickButton"/>	            
	            
struts代码：	            
	            
	<package name="ajax" extends="json-default">  
	        <action name="ajaxRequest"  class="com.action.ajax" method="test">  
	            <result type="json"></result>  
	        </action>  
	</package>

java代码：

	package com.action;  
	  
	import com.opensymphony.xwork2.ActionSupport;  
	  
	public class ajax extends ActionSupport{  
	    private String request;  
	    private String result;  
	    private String [] q={"a","b","c"};   
	      
	    public String test(){  
	        System.out.println("ajax");  
	        request="request";  
	        result="result";  
	        return SUCCESS;  
	    }  
	      
	    public String getRequest() {  
	        return request;  
	    }  
	    public void setRequest(String request) {  
	        this.request = request;  
	    }  
	    public String getResult() {  
	        return result;  
	    }  
	    public void setResult(String result) {  
	        this.result = result;  
	    }  
	  
	    public String[] getQ() {  
	        return q;  
	    }  
	  
	    public void setQ(String[] q) {  
	        this.q = q;  
	    }  
	      
	  
	}












	            
	            
	            
	            
