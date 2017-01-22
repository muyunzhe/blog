---
title: python HTTP库 urllib和urllib2
comments: true
categories: [technology,programming]
tags: [programming,python]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-07-29 18:28:07
---
urllib和urllib2模块都做与请求URL相关的操作，但他们提供不同的功能。
urllib2.urlopen accepts an instance of the Request class or a url, （whereas urllib.urlopen only accepts a url 中文意思就是：urllib2.urlopen可以接受一个Request对象或者url，（在接受Request对象时候，并以此可以来设置一个URL的headers），urllib.urlopen只接收一个url
urllib 有urlencode,urllib2没有，这也是为什么总是urllib，urllib2常会一起使用的原因
<!--more-->
# Proxy 的设置
urllib2 默认会使用环境变量 http_proxy 来设置 HTTP Proxy。如果想在程序中明确控制 Proxy 而不受环境变量的影响，可以使用下面的方式
```
import urllib2

enable_proxy = True
proxy_handler = urllib2.ProxyHandler({"http" : 'http://some-proxy.com:8080'})
null_proxy_handler = urllib2.ProxyHandler({})

if enable_proxy:
    opener = urllib2.build_opener(proxy_handler)
else:
    opener = urllib2.build_opener(null_proxy_handler)

urllib2.install_opener(opener)
```
这里要注意的一个细节，使用 urllib2.install_opener() 会设置 urllib2 的全局 opener 。这样后面的使用会很方便，但不能做更细粒度的控制，比如想在程序中使用两个不同的 Proxy 设置等。比较好的做法是不使用 install_opener 去更改全局的设置，而只是直接调用 opener 的 open 方法代替全局的 urlopen 方法。
# Timeout 设置
```
import urllib2
response = urllib2.urlopen('http://www.google.com', timeout=10)
```
# 在 HTTP Request 中加入特定的 Header
在 HTTP Request 中加入特定的 Header
要加入 header，需要使用 Request 对象:
```
request = urllib2.Request(uri)
request.add_header('User-Agent', 'fake-client')
response = urllib2.urlopen(request)
```
>User-Agent : 有些服务器或 Proxy 会通过该值来判断是否是浏览器发出的请求
Content-Type : 在使用 REST 接口时，服务器会检查该值，用来确定 HTTP Body 中的内容该怎样解析。常见的取值有：
  application/xml ： 在 XML RPC，如 RESTful/SOAP 调用时使用
  application/json ： 在 JSON RPC 调用时使用
  application/x-www-form-urlencoded ： 浏览器提交 Web 表单时使用
在使用服务器提供的 RESTful 或 SOAP 服务时， Content-Type 设置错误会导致服务器拒绝服务

# Redirect
urllib2 默认情况下会针对 HTTP 3XX 返回码自动进行 redirect 动作，无需人工配置。要检测是否发生了 redirect 动作，只要检查一下 Response 的 URL 和 Request 的 URL 是否一致就可以了。
```
response = urllib2.urlopen('http://www.google.cn')
redirected = response.geturl() == 'http://www.google.cn'
```
如果不想自动 redirect，除了使用更低层次的 httplib 库之外，还可以自定义 HTTPRedirectHandler 类。
# Cookie
```
cookie = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
response = opener.open('http://www.google.com')
for item in cookie:
    if item.name == 'some_cookie_item_name':
        print item.value
```
# 使用 HTTP 的 PUT 和 DELETE 方法
urllib2 只支持 HTTP 的 GET 和 POST 方法，如果要使用 HTTP PUT 和 DELETE ，只能使用比较低层的 httplib 库。虽然如此，我们还是能通过下面的方式，使 urllib2 能够发出 PUT 或 DELETE 的请求：
```
request = urllib2.Request(uri, data=data)
request.get_method = lambda: 'PUT' # or 'DELETE'
response = urllib2.urlopen(request)
```
# 得到 HTTP 的返回码
```
try:
    response = urllib2.urlopen('http://restrict.web.com')
except urllib2.HTTPError, e:
    print e.code

```
# Debug Log
# 异常处理
## URLError异常
通常引起URLError的原因是：无网络连接（没有到目标服务器的路由）、访问的目标服务器不存在。在这种情况下，异常对象会有reason属性（是一个（错误码、错误原因）的元组）。
```
url="http://www.baidu.com/"
try:
    response=urllib2.urlopen(url)
except urllib2.URLError,e:
    print e.reason
```
## HTTPError
每一个从服务器返回的HTTP响应都有一个状态码。其中，有的状态码表示服务器不能完成相应的请求，默认的处理程序可以为我们处理一些这样的状态码（如返回的响应是重定向，urllib2会自动为我们从重定向后的页面中获取信息）。有些状态码，urllib2模块不能帮我们处理，那么urlopen函数就会引起HTTPError异常,其中典型的有404/401。
HTTPError异常的实例有整数类型的code属性，表示服务器返回的错误状态码。
urllib2模块默认的处理程序可以处理重定向（状态码是300范围），而且状态码在100-299范围内表示成功。因此，能够引起HTTPError异常的状态码范围是：400-599.
当引起错误时，服务器会返回HTTP错误码和错误页面。你可以将HTPError实例作为返回页面，这意味着，HTTPError实例不仅有code属性，还有read、geturl、info等方法。
```
url="http://cs.scu.edu.cn/~duanlei"
try:
    response=urllib2.urlopen(url)
except urllib2.HTTPError,e:
    print e.code
    print e.read()
```
# 总结
如果想在代码中处理URLError和HTTPError有两种方法，代码如下：
方法一
```
url="xxxxxx"  #需要访问的URL
try:
    response=urllib2.urlopen(url)
except urllib2.HTTPError,e:    #HTTPError必须排在URLError的前面
    print "The server couldn't fulfill the request"
    print "Error code:",e.code
    print "Return content:",e.read()
except urllib2.URLError,e:
    print "Failed to reach the server"
    print "The reason:",e.reason
else:
    #something you should do
    pass  #其他异常的处理
```
方法二
```
url="http://xxx"  #需要访问的URL
try:
    response=urllib2.urlopen(url)
except urllib2.URLError,e:
    if hasattr(e,"reason"):
        print "Failed to reach the server"
        print "The reason:",e.reason
    elif hasattr(e,"code"):
        print "The server couldn't fulfill the request"
        print "Error code:",e.code
        print "Return content:",e.read()
else:
    pass  #其他异常的处理
```

# urllib2各方法实例
```
import urllib2
response = urllib2.urlopen('http://python.org/')
print "Response:", response

# Get the URL. This gets the real URL.
print "The URL is: ", response.geturl()

# Getting the code
print "This gets the code: ", response.code

# Get the Headers.
# This returns a dictionary-like object that describes the page fetched,
# particularly the headers sent by the server
print "The Headers are: ", response.info()

# Get the date part of the header
print "The Date is: ", response.info()['date']

# Get the server part of the header
print "The Server is: ", response.info()['server']

# Get all data
html = response.read()
print "Get all data: ", html

# Get only the length
print "Get the length :", len(html)

# Showing that the file object is iterable
for line in response:
 print line.rstrip()

# Note that the rstrip strips the trailing newlines and carriage returns before
# printing the output.
```

发送get请求demo例子：
```
#!/usr/bin/python
# -*- coding: utf-8 -*-
# hello_http_get.py

import httplib

#请求头
headers = {
    "cityId":"1337",
    "userId":"1234",
    "token":"abcd",
    "device-id":"778E5992-6DD7-4AB9-B2CC-741F31134D6C",
    "caller-id":"python-test",
    "request-id":"2298760f18a846969725a7752331e3b8",
    "platform":"5",
    "platformVersion":"21",
    "apiVersion":"21",
    "Content-type":"application/json"
}
#请求参数
params=''
#请求资源
resource_uri='/helloworld'
#请求方法
method_type='GET'

httpClient=None

try:
    print headers
    print params
    httpClient=httplib.HTTPConnection('localhost',9020,timeout=30)
    httpClient.request(method_type,resource_uri,params,headers)

    #获取返回数据对象
    response=httpClient.getresponse()
    print response.status #HTTP返回状态
    print response.reason #HTTP返回状态描述
    print response.read() #输出返回数据
except Exception,e:
    print e
finally:
    if httpClient:
        httpClient.close()
```

发送post请求demo例子：
```
#!/usr/bin/python
# -*- coding: utf-8 -*-
# hello_http_post.py

import httplib
import urllib
import json

#请求头
headers = {
    "cityId":"1337",
    "userId":"1234",
    "token":"abcd",
    "device-id":"778E5992-6DD7-4AB9-B2CC-741F31134D6C",
    "caller-id":"python-test",
    "request-id":"2298760f18a846969725a7752331e3b8",
    "platform":"5",
    "platformVersion":"21",
    "apiVersion":"21",
    "Content-type":"application/json"
}
#请求参数
params={
    "houseId" : "24230",
    "houseTypeId" : "0",
    "userPhone" : "13866554433",
    "userName" : "张三",
    "houseName" : "helloworld测试楼盘",
}
#服务端约定为json格式
params=json.dumps(params)
#服务端约定为body:k1=v1&k2=v2格式
#params=urllib.urlencode(params)
#请求资源
resource_uri='/helloworld'
#请求方法
method_type='POST'

httpClient=None

try:
    print headers
    print params
    httpClient=httplib.HTTPConnection('localhost',9020,timeout=30)
    httpClient.request(method_type,resource_uri,params,headers)

    #获取返回数据对象
    response=httpClient.getresponse()
    print response.status #HTTP返回状态
    print response.reason #HTTP返回状态描述
    print response.read() #输出返回数据
except Exception,e:
    print e
finally:
    if httpClient:
        httpClient.close()
  ```
