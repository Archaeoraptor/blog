---
title: 爬虫笔记
tags:
  - 爬虫
abbrlink: fd19
date: 2019-09-27 23:44:49
categories:
  - 笔记
---

未完待续
<!--more-->

#### 浏览器F12调试和抓包

略

#### urllib库

##### 打开并读取url

```python
urllib.request.``urlopen`(*url*, *data=None*, [*timeout*,]***,*cafile=None*, *capath=None*, *cadefault=False*, *context=None*)
```

常用的函数read()读取, info()返回头信息,getcode()返回状态码,geturl()返回url

一个简单的读写文件:

```python
response = urllib.request.urlopen('https://zhangjk98.xyz', None, 2)
# 读取返回的内容
html = response.read().decode('utf8')
# 写入txt
f = open('code1.txt', 'w', encoding='utf8')
f.write(html)
f.close()
```

##### 完整的url请求

使用urllib.request.Request处理完整请求

```python
urllib.request.Request(url, data=None, headers={}, 	origin_req_host=None, unverifiable=False, method=None)
```

请求方式method有get和post两种, headers为请求头信息

##### 使用代理

使用urllib.request.ProyxHandler()

```python
# 设置代理IP
proxy_handler = urllib.request.ProxyHandler({
    'http': '218.56.132.157:8080',
    'https': '183.30.197.29:9797'})
# 必须使用build_opener()函数来创建带有代理IP功能的opener对象
opener = urllib.request.build_opener(proxy_handler)
response = opener.open(url)
```

##### cookies

使用HTTPCookiePorcesser()模块处理,使用MozillaCookieJar读写
```python
from http import cookiejar
filename = 'cookie.txt'
cookie = cookiejar.MozillaCookieJar(filename)
handler = urllib.request.HTTPCookieProcessor(cookie)
# 创建自定义opener
opener = urllib.request.build_opener(handler)
# open方法打开网页
response = opener.open('https://movie.douban.com/')
# 保存cookie文件
cookie.save()
# 读取cookie
print(cookie)
```

#### requests库

主要参数url, 请求头信息headers, 是否开启证书验证verify,超时timeout, 代理proxies和cookies, 文件files(字典格式)

```python
# 格式
import requests
url = 'https://google.com'
r = requests.get('url,headers=headers, proxies=proxies,verify=False, cookies=cookies)
r = requests.post('url, data= data, files=files, headers=headers, proxies=proxies,verify=False, cookies=cookies)
```

#### 等待填坑
