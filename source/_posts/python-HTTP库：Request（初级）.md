---
title: python HTTP库：Request（初级）
comments: true
categories: [technology,programming]
tags: [programming,python]
keywords: '牧云者,人工智能,云计算,数据挖掘,hexo,blog'
date: 2016-07-29 19:09:07
---
Requests 使用的是 urllib3，继承了urllib2的所有特性。Requests支持HTTP连接保持和连接池，支持使用cookie保持会话，支持文件上传，支持自动确定响应内容的编码，支持国际化的 URL 和 POST 数据自动编码。
<!--more-->
# 发送请求
## get请求
` r = requests.get('https://github.com/timeline.json')`
## post请求
`r = requests.post("http://httpbin.org/post")`
现在，我们有一个名为 r 的 Response 对象。我们可以从这个对象中获取所有我们想要的信息。
那么其他 HTTP 请求类型：PUT，DELETE，HEAD 以及 OPTIONS 又是如何的呢？都是一样的简单：
```
r = requests.put("http://httpbin.org/put")
r = requests.delete("http://httpbin.org/delete")
r = requests.head("http://httpbin.org/get")
r = requests.options("http://httpbin.org/get")
```

# 传递 URL 参数
举例来说，如果你想传递 key1=value1 和 key2=value2 到 httpbin.org/get ，那么你可以使用如下代码：
```
 payload = {'key1': 'value1', 'key2': 'value2'}
 r = requests.get("http://httpbin.org/get", params=payload)
```
通过打印输出该 URL，你能看到 URL 已被正确编码：
```
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
```
你还可以将一个列表作为值传入：
```
>>> payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
>>> r = requests.get('http://httpbin.org/get', params=payload)
>>> print(r.url)
http://httpbin.org/get?key1=value1&key2=value2&key2=value3
```

# 响应内容
## 文本编码
```
>>> import requests
>>> r = requests.get('https://github.com/timeline.json')
>>> r.text
u'[{"repository":{"open_issues":0,"url":"https://github.com/...
```
Requests 会自动解码来自服务器的内容。大多数 unicode 字符集都能被无缝地解码。
## 二进制响应内容
你也能以字节的方式访问请求响应体，对于非文本请求：
```
>>> r.content
b'[{"repository":{"open_issues":0,"url":"https://github.com/...
```
Requests 会自动为你解码 gzip 和 deflate 传输编码的响应数据。
例如，以请求返回的二进制数据创建一张图片，你可以使用如下代码：
```
>>> from PIL import Image
>>> from io import StringIO

>>> i = Image.open(StringIO(r.content))
```
## JSON 响应内容
```
r = requests.get('https://github.com/timeline.json')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
```
如果 JSON 解码失败， r.json 就会抛出一个异常。例如，相应内容是 401 (Unauthorized)，尝试访问 r.json 将会抛出 ValueError: No JSON object could be decoded 异常。
## 原始响应内容
在罕见的情况下，你可能想获取来自服务器的原始套接字响应，那么你可以访问 r.raw。 如果你确实想这么干，那请你确保在初始请求中设置了 stream=True。具体你可以这么做：
```
>>> r = requests.get('https://github.com/timeline.json', stream=True)
>>> r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```
但一般情况下，你应该以下面的模式将文本流保存到文件：
```
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)
```
使用 Response.iter_content 将会处理大量你直接使用 Response.raw 不得不处理的。 当流下载时，上面是优先推荐的获取内容方式。

# 定制请求头
如果你想为请求添加 HTTP 头部，只要简单地传递一个 dict 给 headers 参数就可以了。
例如，在前一个示例中我们没有指定 content-type:
```
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}

>>> r = requests.get(url, headers=headers)
```
注意: 定制 header 的优先级低于某些特定的信息源，例如：

如果在 .netrc 中设置了用户认证信息，使用 headers= 设置的授权就不会生效。而如果设置了 auth= 参数，``.netrc`` 的设置就无效了。
如果被重定向到别的主机，授权 header 就会被删除。
代理授权 header 会被 URL 中提供的代理身份覆盖掉。
在我们能判断内容长度的情况下，header 的 Content-Length 会被改写。
更进一步讲，Requests 不会基于定制 header 的具体情况改变自己的行为。只不过在最后的请求中，所有的 header 信息都会被传递进去。

注意: 所有的 header 值必须是 string、bytestring 或者 unicode。尽管传递 unicode header 也是允许的，但不建议这样做。

# 更加复杂的 POST 请求
通常，你想要发送一些编码为表单形式的数据——非常像一个 HTML 表单。要实现这个，只需简单地传递一个字典给 data 参数。你的数据字典在发出请求时会自动编码为表单形式：
```
>>> payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```
很多时候你想要发送的数据并非编码为表单形式的。如果你传递一个 string 而不是一个 dict，那么数据会被直接发布出去。

例如，Github API v3 接受编码为 JSON 的 POST/PATCH 数据：
```
>>> import json

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```
此处除了可以自行对 dict 进行编码，你还可以使用 json 参数直接传递，然后它就会被自动编码。这是 2.4.2 版的新加功能：
```
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, json=payload)
```

# 响应状态码
我们可以检测响应状态码：
```
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
```
为方便引用，Requests还附带了一个内置的状态码查询对象：
```
>>> r.status_code == requests.codes.ok
True
```
如果发送了一个错误请求(一个 4XX 客户端错误，或者 5XX 服务器错误响应)，我们可以通过 Response.raise_for_status() 来抛出异常：
```
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404

>>> bad_r.raise_for_status()
Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
```
但是，由于我们的例子中 r 的 status_code 是 200 ，当我们调用 raise_for_status() 时，得到的是：
```
>>> r.raise_for_status()
None
```

# 响应头
我们可以查看以一个 Python 字典形式展示的服务器响应头：
```
>>> r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
```
但是这个字典比较特殊：它是仅为 HTTP 头部而生的。根据 RFC 2616， HTTP 头部是大小写不敏感的。
因此，我们可以使用任意大写形式来访问这些响应头字段：
```
>>> r.headers['Content-Type']
'application/json'

>>> r.headers.get('content-type')
'application/json'
```
它还有一个特殊点，那就是服务器可以多次接受同一 header，每次都使用不同的值。但 Requests 会将它们合并，这样它们就可以用一个映射来表示出来，参见 RFC 7230:

A recipient MAY combine multiple header fields with the same field name into one "field-name: field-value" pair, without changing the semantics of the message, by appending each subsequent field value to the combined field value in order, separated by a comma.

接收者可以合并多个相同名称的 header 栏位，把它们合为一个 "field-name: field-value" 配对，将每个后续的栏位值依次追加到合并的栏位值中，用逗号隔开即可，这样做不会改变信息的语义。

# Cookie
如果某个响应中包含一些 cookie，你可以快速访问它们：
```
>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'
```
要想发送你的cookies到服务器，可以使用 cookies 参数：
```
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

# 重定向与请求历史
默认情况下，除了 HEAD, Requests 会自动处理所有重定向。
可以使用响应对象的 history 方法来追踪重定向。
Response.history 是一个 Response 对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从最老到最近的请求进行排序。
例如，Github 将所有的 HTTP 请求重定向到 HTTPS：
```
>>> r = requests.get('http://github.com')
>>> r.url
'https://github.com/'

>>> r.status_code
200

>>> r.history
[<Response [301]>]
```
如果你使用的是GET、OPTIONS、POST、PUT、PATCH 或者 DELETE，那么你可以通过 allow_redirects 参数禁用重定向处理：
```
>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]
```
如果你使用了 HEAD，你也可以启用重定向：
```
>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[<Response [301]>]
```

# 超时
你可以告诉 requests 在经过以 timeout 参数设定的秒数时间之后停止等待响应：
```
>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```
>注意
timeout 仅对连接过程有效，与响应体的下载无关。 timeout 并不是整个下载响应的时间限制，而是如果服务器在 timeout 秒内没有应答，将会引发一个异常（更精确地说，是在 timeout 秒内没有从基础套接字上接收到任何字节的数据时）

# 错误与异常
遇到网络问题（如：DNS 查询失败、拒绝连接等）时，Requests 会抛出一个 ConnectionError 异常。
如果 HTTP 请求返回了不成功的状态码， Response.raise_for_status() 会抛出一个 HTTPError 异常。
若请求超时，则抛出一个 Timeout 异常。
若请求超过了设定的最大重定向次数，则会抛出一个 TooManyRedirects 异常。
所有Requests显式抛出的异常都继承自 requests.exceptions.RequestException 。

# 缺陷
requests不是python自带的库，需要另外安装 easy_install or pip install
requests缺陷:直接使用不能异步调用，速度慢（from others）。官方的urllib可以替代它。
不建议使用requests模块
