---
title: Python 爬坑记-数据库操作
date: 2019-09-24 11:13:02
category:
- 随记
tags:
- Python
- Mysql
---

> 背景：最近有个小需求，需要爬取某个网站的内容放到数据库，方便其他地方查看使用。所以利用这个时机也学习一下 `Python` 对数据库的一些操作

从文档可以了解到，`Python` 的 `DB-API` 已经为大多数数据库实现了接口。大致流程跟其他语言没啥区别

* 引入 `API` 模块
* 连接数据库
* 执行 `SQL` 语句
* 关闭数据连接

这里，我使用 `Mysql` ，所以我们需要用到 `Python` 的 `MySQLdb` 模块

### 安装 `MySQLdb` 模块

> 我用的是 win10，python 3.7

到 [<https://www.lfd.uci.edu/~gohlke/pythonlibs/>](<https://www.lfd.uci.edu/~gohlke/pythonlibs/>) ，下载对应系统的版本的 whl 文件，下载完毕之后，到 whl 所在目录执行：`pip install mysqlclient-1.4.2-cp37-cp37m-win32.whl`

```shell
$ pip install mysqlclient-1.4.2-cp37-cp37m-win32.whl
Processing d:\▒▒▒▒\mysqlclient-1.4.2-cp37-cp37m-win32.whl
Installing collected packages: mysqlclient
Successfully installed mysqlclient-1.4.2

```

这样就算安装成功了

### 插入数据

一切都准备就绪，那就可以到代码来了，废话不多说直接上代码

```python
import requests
import MySQLdb
import time
from bs4 import BeautifulSoup

# 推荐页面
recommUrl = 'https://www.gushiwen.org/';

# 获取诗词
def index():
    page = requests.get(recommUrl)

    soup = BeautifulSoup(page.text, "html.parser")

    main3 = soup.find(class_='main3')
    left = main3.find(class_='left')

    param = []
    for sons in left.find_all('div', class_='sons'):
        cont = sons.find('div', class_='cont')

        # 标题
        title = cont.p.a.get_text()

        # 朝代 作者
        source  = cont.find('p', class_='source').find_all('a')
        dynasty = source[0].get_text()
        author = source[1].get_text()

        # 内容
        contson = cont.find('div', class_='contson').get_text()

        # 标签
        tagText = ''
        if sons.find('div', class_='tag') != None:
            tags = sons.find('div', class_='tag').find_all('a')
            for tag in tags:
                tagText = tagText + tag.get_text() + ','
        created_at = updated_at = time.strftime(
            "%Y-%m-%d %H-%M-%S", time.localtime())
        param.append([title, dynasty, author, contson,
                      tagText, created_at, updated_at])

    return param



def insertDataBases(params):
    db  = connectDb()
    cur = db.cursor()

    sql = 'insert into mxp_poetries (title, dynasty, author, content, tags, created_at, updated_at) Values (%s, %s, %s, %s, %s, %s, %s)'

    try:
        cur.executemany(sql, params)
        db.commit()
    except Exception as identifier:
        print(identifier)
        db.rollback()

    # 关闭连接
    db.close()
    print('插入成功')

def connectDb():
    db = MySQLdb.connect(host="127.0.0.1", port=3306, user='root',
                         password='root', db='bl', charset="utf8")

    return db

if __name__ == '__main__':
    params = index()
    insertDataBases(params)

```

这里我爬取的是一个古诗文的内容，可以看到执行插入的时候我用的 `executemany` ，而不是 `execute` ，这样可以同时插入多条，提高性能





### 问题

#### 问题1

> 端口问题

代码：

```python
db = MySQLdb.connect(host="127.0.0.1", port='3306', user='root', password='root',  db='bl')
cur = db.cursor()

sql = 'select * from poetries'

cur.execute(sql)

results = cur.fetchall()

print(results)
```


出现以下代码：

```shell
Traceback (most recent call last):
  File "main.py", line 60, in <module>
    insertDataBases()
  File "main.py", line 47, in insertDataBases
    db = MySQLdb.connect(host="127.0.0.1", port='3306', user='root', password='root', db='blog')
  File "C:\Users\StarFish\AppData\Local\Programs\Python\Python37-32\lib\site-packages\MySQLdb\__init__.py", line 84, in Connect
    return Connection(*args, **kwargs)
  File "C:\Users\StarFish\AppData\Local\Programs\Python\Python37-32\lib\site-packages\MySQLdb\connections.py", line 164, in __init__
    super(Connection, self).__init__(*args, **kwargs2)
TypeError: an integer is required (got type str)
```

查看源码之后：

`port` 是要 `int` 类型的。好严格啊

所以端口号改成 `int` 类型，解决

```python
db = MySQLdb.connect(host="127.0.0.1", port=3306, user='root', password='root',  db='bl')
cur = db.cursor()

sql = 'select * from poetries'

cur.execute(sql)

results = cur.fetchall()

print(results)
```



#### 问题2

> 插入数据，数据库编码跟数据源编码不一样报错

**报错信息**

```shell
latin-1' codec can't encode characters in position 0-4: ordinal not in range(256)
```

**解决**

在连接数据库的时候，先声明编码格式

```python
db = MySQLdb.connect(host="127.0.0.1", port=3306, user='root',
                         password='root', db='bl', charset="utf8")
```