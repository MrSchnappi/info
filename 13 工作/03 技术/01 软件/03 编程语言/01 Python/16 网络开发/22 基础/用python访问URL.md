
# 用 Python 访问 URL



## 用 Python 访问 URL

用 Python 访问 URL，比如需要访问[http://www.oschina.net](http://www.oschina.net/)这个 URL，其实就是构造一个合法的 HTTP 请求，再通过 TCP 协议发送到 www.oschina.net这台主机的 TCP80 端口。接着会从 www.oschina.net等待一个 HTTP 协议的报文，最后关掉 TCP 连接。

### 访问方式

```
import urllib2
strHtml = urllib2.urlopen('http://www.oschina.net').read()12
```

直接引入 urllib2 这个包之后，马上就可以调用 urlopen 这个方法打开这个网址。urlopen返回的是一个具有 file 对象行为的对象，直接使用 read 方法就可以读出内容。

### 异常处理

在实际中是可能会出现各种异常现象的，如常见的 3 种：
**404 NOT FOUND**
**403 FORBIDDEN**
**500 Internal Server Error**
404，403, 500这些是 HTTP 协议的状态码。 HTTP 1.1协议规定了 5 种不同的状态码，分别是
1xx表示临时状态。常见有 100。
2xx表示成功访问。常见有 200 206。
3xx表示跳转。常见有 302 304。
4xx表示客户端请求不正确。常见的有 400 403 404 。
5xx表示服务器内部出错。常见有 500。

在我们写程序的时候，一定要考虑到服务器会出现下面的意外。我们可以通过下面的方法来处理：

```py
import urllib2
try:
    s = urllib2.urlopen("http://www.oschina.net").read()
except urllib2.HTTPError,e:
   print e.code12345
```

如果访问 url 失败，代码会抛出 urllib2.HTTPError这个异常。而这个异常的 code 属性就是 HTTP 的状态码。

考虑到 HTTP 协议可能返回错误的结果是否就已经够了呢？答案是不够的。因为这个世界上实在是有太多异常情况的发生，特别是需要访问网络资源的时候需要额外小心。比如当网络连接超时或者是失败的时候，代码会抛出 urllib2.URLError这个异常。我们也要处理这个异常。

```py
import urllib2
try:
    s = urllib2.urlopen("http://www.oschina.net").read()
except urllib2.HTTPError,e:
   print e.code
except urllib2.URLErrror,e:
    print str(e)1234567
```

我们在访问 url 时候，千万要注意一件事情，就是 URL 中的参数是不能含有一些特殊字符的。URL中参数的字符比如’?=&’会破坏整个 URL。因此我们需要将这些参数转义，用%十六进制字符表示。

例如 URL
[http://www.oschina.net?a=](http://www.oschina.net/?a=)^&b=&^^&

实际上是有两个参数
a=^
b=&^^&

这两个参数都含有非法字符，如何处理呢。

```py
>>> import urllib
>>> urllib.urlencode({'a':'^','b':'&^^&'}）
'a=%5E&b=%26%5E%5E%26'
```

# 相关

- [用 Python 访问 URL](https://blog.csdn.net/wang725/article/details/50747460)