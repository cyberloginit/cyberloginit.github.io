---
layout: post
title: WordPress XML Backup to Markdown
---

记录把WordPress博客备份XML文件导出成Markdown格式遇到的问题。

使用工具[dreikanter/wp2md](https://github.com/dreikanter/wp2md),

首先，新建virtualenv环境，`mkdir wp2md && cd wp2md && virtualenv ENV`, 
接下来安装[dreikanter/wp2md](https://github.com/dreikanter/wp2md)，
`pip install git+https://github.com/dreikanter/wp2md`, 
然后运行wp2md，`wp2md -d /export/path/ wordpress-dump.xml`, 
`/export/path/`为输出目录，wordpress-dump.xml为WordPress导出文件。
遇到的问题是

![TypeError](/images/TypeError.png "TypeError")

定位到文件`/home/master/Work/wp2md/ENV/local/lib/python2.7/site-packages/wp2md/wp2md.py`
的382行的`content = meta.get('description', '') + '\n\n'`出错。
因为该XML文件中post的description tag为空，`content = meta.get('description', '')`
返回的NoneType与字符串`'\n\n'`进行连接`+`运算自然会报错。
~~搜索`meta.get`可以发现前两处`meta.get`的用法为`meta.get('[tag name]', None)`，故此处应为typo，
将382行的`content = meta.get('description', '') + '\n\n'`修改为`content = meta.get('description', None) + '\n\n'`即可。~~

一个ugly的解决方法是，既然`description`为空，那就修改XML文件，强行为`<description></description>`中加入一个空格。
这样`content = meta.get('description', '')`可以获得返回值空格字符`' '`，但这种做法不仅会为post文件Front Matter中description变量引入额外的空格，而且逼格太低。

![可恶的额外空格](/images/ExtraSpace.png "可恶的额外空格")

Hacker的做法是，直接修改程序源码（反正是开源的），修改后的代码部分如下：

```
382    if not meta.get('description', ''):
383        content = '\n\n'
384    else:
385        content = meta.get('description', '') + '\n\n'
```

~~接写来当然是给作者提个pull request啦。~~
Pull Request已提交。