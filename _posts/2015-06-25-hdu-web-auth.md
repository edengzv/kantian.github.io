---
title: "记一次问题的解决"
date: 2015-06-25 00:00:00
categories: ["Linux"]
---

### 背景

学校要求所有的机子都需要输入学号和密码才能访问外网，导致实验室的一台CentOS服务器也未能幸免。这台CentOS服务器装的是minial版本的，也就是说没有任何可视界面，因此常规的在浏览器里登录自然不行了。

### 实施

因为有过一点爬虫的基础，于是想当然的想到，写个脚本模拟登录不是就可以了？
于是便有了以下这个脚本：

```python
#!/usr/bin/python
#
# HDU WebAuth For Centos Without GNOME Desktop
# Copyright 2015 edeng.zheng@gmail.com
# 2015-05-13

import urllib
import urllib2
import getpass

#request url
url = 'http://IPAddress/webAuth/index.htm?'

#get the username and password
myusername = raw_input("Enter your username:")
mypasswd = getpass.getpass("Enter your passwd:")

#encode the post data
values = {'username':myusername,'pwd':mypasswd}
data = urllib.urlencode(values)

#post the request
req = urllib2.Request(url,data)
response = urllib2.urlopen(req)

#get the response page
ret_page = response.read()

#return succeed or failed
flag_success = "var tips="
flag_success.decode('utf8')
result = ret_page.find(flag_success)

#print result;
if result >= 0:
    print 'Login Success!'
else:
    print 'Login Failed!'
```

在借助Python类库的基础上，整个实现起来也很方便；另一方面也可见学校的验证系统并没有对脚本的模拟登录做过多的阻扰和限制；原谅我把登录成功的检测匹配写的这么恶心 = =

### 测试

登录以后就可以ping通外网了
![测试](/img/webauth.png)

### 后记

实验室的小伙伴脑洞打开，如果把用户名和密码固定，然后让脚本在每次开机启动的时候自动运行，不是就不用每次都进行麻烦的手动登录了？嗯，确实是一个好主意！