---
layout: post
title: "垂直发布—Httpclient模拟登陆获取页面信息"
date: 2012-02-13 18:24
comments: true
categories: Tec
---
过年回了趟老家，回来后寒假还有好大一截，闲着也是没事就这两天初步了解了下垂直发布的东西，收获一点点吧，感觉离预期还差很远。趁着还有印象写下来，步步为营呗。网上搜索，看到很多模拟登录都是用Httpclient的，另外一个是HttpURLConnection。
HttpURLConnection是java的标准类，HttpClient 是 Apache Jakarta Common 下的子项目，用来提供支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。因为是初学，就图着简单易用的目的选择了Httpclient，感觉还是挺容易上手的，不过这两天遇到的问题也不少。

<!--more-->

既然是模拟正规的登录，那首先要抓包，通过一次正常的登录来获取表单提交的信息，既而自己在后台把相应的参数填好发出。

网上的抓包工具不少，刚开始也下了好几个，像Wireshark，SmartSniff都是好多人推荐的。开始用的Wireshark，确实好多功能，还有过滤什么的，但贪图方便用着用着就转去SmartSniff了。就是准备登录时点一下开始键，然后登录，再停止抓包，挨个找一下这段时间的TCP，根据登录网址，很容易就找到目标信息了。

就像这样
![image](http://i1001.photobucket.com/albums/af134/mxiaochi/blogsource/100001114023512_zpsa214f4b2.jpg)

可以看到蓝色字的最后一行就是该网站正常登录的表单提交信息了。顺便一提，上面的蓝色是浏览器发送的请求，下面的粉色则是服务器返回来的数据，一般情况下HTTP/1.1 200就表示登录正常了，这是一个判定标准。

因为开始4.x网上用的人不多，出的时间不久的缘故吧，所以搜到的信息很多时候都是3.x的写法，自己也就照葫芦画瓢了，后来看到说4.x基本上是重写的，跟之前的3.x有很大不同，所以第二天又重新用4.x试了一遍，写法上稍微复杂了点，但方式都差不多感觉。

这里我是用的biketo网站做的小白鼠，平常喜欢骑下单车再估计这网站也不会做得很有技术难度估计能好模拟一点就用这个了，哈哈。

	POST /bbs/member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1 HTTP/1.1User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1Host: www.biketo.comContent-Length: 107Content-Type: application/x-www-form-urlencodedfastloginfield=username&username=&cookietime=2592000&password=&quickforward=yes&handlekey=1s
	
首先正常登录时，发送的post是这个


`POST /bbs/member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1 HTTP/1.1
Host: www.biketo.com
Connection: keep-alive
Content-Length: 107
Cache-Control: max-age=0
Origin: http://www.biketo.com
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://www.biketo.com/bbs/forum-73-1.html
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8
Accept-Charset: GBK,utf-8;q=0.7,*;q=0.3
Cookie: rtime=5; ltime=1326617596891; cnzz_eid=83708416-1318074453-; pgv_pvi=8849333319; pgv_info=ssi=s8902215826; eucsfmldoactive=136495; XFJp_2132_lastvisit=1329121650; XFJp_2132_forum_lastvisit=D_73_1329125252; XFJp_2132_visitedfid=73; XFJp_2132_sid=Q3V96V; __utma=114141336.153132218.1318078021.1329105199.1329125251.12; __utmb=114141336.4.10.1329125251; __utmc=114141336; __utmz=114141336.1319874537.4.2.utmcsr=google.com.hk|utmccn=(referral)|utmcmd=referral|utmcct=/; XFJp_2132_lastact=1329125252%09home.php%09misc; XFJp_2132_sendmail=1
fastloginfield=username&username=&cookietime=2592000&password=&quickforward=yes&handlekey=ls`

然后就是模拟

	PostMethod post = new PostMethod("http://www.biketo.com/bbs/member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1");
	
	post.addParameter("fastloginfield", "username");
	post.addParameter("username", "uername");
	post.addParameter("cookietime", "2592000");
	post.addParameter("password", "password");
	post.addParameter("quickforward", "yes");
	post.addParameter("handlekey", "1s");
	
	HttpClient client = new HttpClient();
	//client.getParams().setCookiePolicy(CookiePolicy.RFC_2109);
	client.getParams().setParameter(HttpMethodParams.USER_AGENT,"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1");//设置信息 
	client.executeMethod(post);
	GetMethod get = new GetMethod("http://www.biketo.com/bbs/home.php?mod=spacecp");
	CookieSpec cookiespec = CookiePolicy.getDefaultSpec();
	        Cookie[] cookies = cookiespec.match("www.biketo.com", 80/*端口*/, "/" , false , client.getState().getCookies());
	        for (Cookie cookie : cookies) {
	            System.out.println(cookie.getName() + ":" + cookie.getValue());
	        }
	client.executeMethod(get);
	System.out.println(new String(get.getResponseBody()));

这里要注意一下

client.getParams().setParameter(HttpMethodParams.USER_AGENT
的那行代码，因为我们是用的java来发的，不是浏览器，所以默认的user-agent是Jakarta Commons-HttpClient/3.0.1，很多服务器看到这个就直接不鸟你了，不过biketo嘛，没有这层过滤，呵呵。但一般情况下还是要伪装成浏览器的信息，这样才容易得手~
这个样子发出的包便是

`POST /bbs/member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1 HTTP/1.1User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1Host: www.biketo.comContent-Length: 107Content-Type: application/x-www-form-urlencodedfastloginfield=username&username=&cookietime=2592000&password=&quickforward=yes&handlekey=1s`

对比之下会发现比正常的浏览器似乎少了很多信息，转了一篇文章写得很详细，大家有兴趣自己看下吧http://blog.csdn.net/mxiaochi/article/details/7252582
post是请求，get便是验证下登录有没有成功，找一个只有登录才能访问的页面放进GetMethod 里，发送请求，然后outprint出来，贴到记事本里用浏览器打开就可以验证一下啦
这是同样功能4.x的代码，有兴趣也可以试一下

`// The HttpClient is used in one session  
    private HttpResponse response;  
    private DefaultHttpClient httpclient = new DefaultHttpClient(); 
private boolean login() throws IOException{ 
        HttpPost httpost = new HttpPost("http://www.biketo.com/bbs/member.php?mod=logging&action=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1"); 
        httpost.setHeader("User-Agent",
"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1");
       /* httpost.setHeader("Content-Type", "application/x-www-form-urlencoded");
        httpost.setHeader("Referer", "http://my.xxxxxx.com/my/login?history=aHR0cDovL3NoZW56aGVuLmFuanVrZS5jb20v");
        httpost.setHeader("Content-Type", "application/x-www-form-urlencoded");
        httpost.setHeader("Cache-Control", "max-age=0");
        httpost.setHeader("Origin", "http://my.xxxxx.com");
        httpost.setHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,\*\/*;q=0.8");
        httpost.setHeader("Accept-Encoding", "gzip,deflate,sdch");
        httpost.setHeader("Accept-Language", "zh-CN,zh;q=0.8");
        httpost.setHeader("Accept-Charset", "GBK,utf-8;q=0.7,*;q=0.3");*/
        // All the parameters post to the web site  
        List<NameValuePair> nvps = new ArrayList<NameValuePair>();  
        nvps.add(new BasicNameValuePair("fastloginfield", "username"));  
        nvps.add(new BasicNameValuePair("username", ""));  
        nvps.add(new BasicNameValuePair("cookietime", "2592000"));  
        nvps.add(new BasicNameValuePair("password", ""));  
        nvps.add(new BasicNameValuePair("quickforward", "yes"));  
        nvps.add(new BasicNameValuePair("handlekey", "1s")); ` 
       
        httpost.setEntity(new UrlEncodedFormEntity(nvps, HTTP.UTF_8));  
        response = httpclient.execute(httpost);  
        List<Cookie> cookies = httpclient.getCookieStore().getCookies();
        if (cookies.isEmpty()) {
            System.out.println("None");
        } else {
            for (int i = 0; i < cookies.size(); i++) {
                System.out.println("- " + cookies.get(i).toString());
            }
        }
        //Header locationHeader = response.getFirstHeader("Location"); 
        //System.out.println(locationHeader.getValue());
        httpost.abort(); 
        HttpGet httpget = new HttpGet("http://www.biketo.com/bbs/home.php?mod=spacecp"); 
        httpget.setHeader("User-Agent",
`"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1");`
       
        ResponseHandler<String> responseHandler = new BasicResponseHandler(); 
        String responseBody = "";
        responseBody = httpclient.execute(httpget, responseHandler);   
        httpget.abort();  
        httpclient.getConnectionManager().shutdown();  
        System.out.println(responseBody);`


前面注释的一大段就是模仿3.x中缺少的那些浏览器信息了，可以根据正常登录抓回来的信息自己慢慢匹配。
刚刚说的这些都是些很简单的东西，仔细看看不到一天就可以大概掌握了，下面的就是些自己遇到的问题。
我第二个尝试的是一个房地产的网站，发现它的页面是会自动跳转的，再请求的话则会退回登录页面很是苦恼。

`HttpPost httpost = new HttpPost("http://my.anjuke.com/usercenter/login.action"); 
        httpost.setHeader("User-Agent",
"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1");
       /* httpost.setHeader("Content-Type", "application/x-www-form-urlencoded");
        httpost.setHeader("Referer", "http://my.anjuke.com/my/login?history=aHR0cDovL3NoZW56aGVuLmFuanVrZS5jb20v");
        httpost.setHeader("Content-Type", "application/x-www-form-urlencoded");
        httpost.setHeader("Cache-Control", "max-age=0");
        httpost.setHeader("Origin", "http://my.anjuke.com");
        httpost.setHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,\*\/*;q=0.8");
        httpost.setHeader("Accept-Encoding", "gzip,deflate,sdch");
        httpost.setHeader("Accept-Language", "zh-CN,zh;q=0.8");
        httpost.setHeader("Accept-Charset", "GBK,utf-8;q=0.7,*;q=0.3");*/
        // All the parameters post to the web site  
        List<NameValuePair> nvps = new ArrayList<NameValuePair>();  
        nvps.add(new BasicNameValuePair("loginpost", "1"));  
        nvps.add(new BasicNameValuePair("formhash", ""));  
        nvps.add(new BasicNameValuePair("sid", "anjuke"));  
        nvps.add(new BasicNameValuePair("url", "aHR0cDovL3NoZW56aGVuLmFuanVrZS5jb20v"));  
        nvps.add(new BasicNameValuePair("systemtime", "1329038088"));  
        nvps.add(new BasicNameValuePair("username", "")); 
        nvps.add(new BasicNameValuePair("password", ""));
        nvps.add(new BasicNameValuePair("remember", "1"));
        nvps.add(new BasicNameValuePair("submit", ""));`
       
        httpost.setEntity(new UrlEncodedFormEntity(nvps, HTTP.UTF_8));  
        response = httpclient.execute(httpost);  
        List<Cookie> cookies = httpclient.getCookieStore().getCookies();
        if (cookies.isEmpty()) {
            System.out.println("None");
        } else {
            for (int i = 0; i < cookies.size(); i++) {
                System.out.println("- " + cookies.get(i).toString());
            }
        }
        //Header locationHeader = response.getFirstHeader("Location"); 
        //System.out.println(locationHeader.getValue());
        httpost.abort(); 
        HttpGet httpget = new HttpGet("http://shenzhen.anjuke.com/ajax/checklogin/"); 
        httpget.setHeader("User-Agent",
`"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1");`
       
        ResponseHandler<String> responseHandler = new BasicResponseHandler(); 
        String responseBody = "";
        responseBody = httpclient.execute(httpget, responseHandler);  
        httpget = new HttpGet("http://my.anjuke.com/marked/3221775");
        responseBody = httpclient.execute(httpget, responseHandler);
        
        httpget.abort();  
        httpclient.getConnectionManager().shutdown();  
        System.out.println(responseBody);


抓包是这样的


`POST /usercenter/login.action HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1
Content-Length: 149
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Host: my.anjuke.com
Connection: Keep-Alive
loginpost=1&formhash=&sid=anjuke&url=aHR0cDovL3NoZW56aGVuLmFuanVrZS5jb20v&systemtime=1329038088&username=mxchenry&password=mxc0818&remember=1&submit=
HTTP/1.1 200 OK
Server: ajk_server
Date: Mon, 13 Feb 2012 10:04:18 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/5.2.14
ajk: m=app10-102, v=2011_51_02
P3P: CP=CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR
Set-Cookie: ajk_member_id=3221775; expires=Tue, 12-Feb-2013 10:04:18 GMT; path=/; domain=.anjuke.com
Set-Cookie: ajk_member_name=mxchenry; expires=Tue, 12-Feb-2013 10:04:18 GMT; path=/; domain=.anjuke.com
Set-Cookie: ajk_member_key=84fb0dfc2a3c64f13c264cb39b072a65; expires=Tue, 12-Feb-2013 10:04:18 GMT; path=/; domain=.anjuke.com
Set-Cookie: ajk_member_time=1360663440; expires=Tue, 12-Feb-2013 10:04:18 GMT; path=/; domain=.anjuke.com
Content-Length: 3349`

可以看到是能正常取回cookie的，但当我尝试get去别的页面的时候都被弹到登录页面了，可是我已经是带上cookie访问的了，不知道这有没有扯上网站安全之类的，这方面没什么经验


`GET /marked/3221775 HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1
Host: my.anjuke.com
Connection: Keep-Alive
Cookie: ajk_member_id=3221775; ajk_member_key=e04e5c298906768adaaa257013991b86; ajk_member_name=mxchenry; ajk_member_time=1360663920
Cookie2: $Version=1`


`HTTP/1.1 302 Found
Server: ajk_server
Date: Mon, 13 Feb 2012 10:12:50 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
ajk: m=app10-034, pv=2012_06_04, jv=ga
Set-Cookie: sessid=1134ABF3-173C-AE40-DBA8-B2C3C1D4B30D; path=/; domain=.anjuke.com
Set-Cookie: aQQ_ajkguid=B68D65A8-9F66-7F9D-7614-02590CB8BF8E; expires=Tue, 12-Feb-2013 10:12:50 GMT; path=/; domain=.anjuke.com
Set-Cookie: aQQ_ajklogintime=1329127970; expires=Tue, 14-Feb-2012 10:12:50 GMT; path=/; domain=.anjuke.com
Set-Cookie: ajk_mem_id=0; expires=Tue, 14-Feb-2012 10:12:50 GMT; path=/; domain=.anjuke.com
Set-Cookie: lastlanded=1329127970; expires=Wed, 14-Mar-2012 10:12:50 GMT; path=/; domain=.anjuke.com
P3P: CP="CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR"
location: http://my.anjuke.com/my/login?history=aHR0cDovL215LmFuanVrZS5jb20vbWFya2VkLzMyMjE3NzU=
GET /my/login?history=aHR0cDovL215LmFuanVrZS5jb20vbWFya2VkLzMyMjE3NzU= HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.126 Safari/535.1
Host: my.anjuke.com
Connection: Keep-Alive
Cookie: aQQ_ajkguid=B68D65A8-9F66-7F9D-7614-02590CB8BF8E; aQQ_ajklogintime=1329127970; ajk_mem_id=0; ajk_member_id=3221775; ajk_member_key=e04e5c298906768adaaa257013991b86; ajk_member_name=mxchenry; ajk_member_time=1360663920; lastlanded=1329127970; sessid=1134ABF3-173C-AE40-DBA8-B2C3C1D4B30D
Cookie2: $Version=1`


`HTTP/1.1 200 OK
Server: ajk_server
Date: Mon, 13 Feb 2012 10:12:50 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
ajk: m=app10-035, pv=2012_06_04, jv=ga
Set-Cookie: ajk_mem_id=0; expires=Tue, 14-Feb-2012 10:12:50 GMT; path=/; domain=.anjuke.com
Set-Cookie: lastlanded=1329127970; expires=Wed, 14-Mar-2012 10:12:50 GMT; path=/; domain=.anjuke.com`
或是用session？个人猜测。













