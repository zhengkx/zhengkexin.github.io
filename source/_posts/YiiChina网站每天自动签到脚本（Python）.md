---
title: YiiChina网站每天自动签到脚本（Python）
date: 2019-05-18 09:09:41
categories:
- 随记
tags:
- Python
- 脚本
- Linux
---



刚好最近工作需要使用 `yii`  这个框架，那必不可少的会经常逛 [YII 中文网](https://www.yiichina.com) 这个网站，也就注册了会员，然后就有每日签到这个东西啦。本着能不动手就尽量不动手的原则。使用了不是很熟练的 `Python` 语言来写这个脚本。

其实，也很简单，就一个登陆操作和一个签到操作。废话不多说，代码如下：

```python
import requests

from bs4 import BeautifulSoup

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.157 Safari/537.36"
}

# 构造签到的header头
headers2 = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.157 Safari/537.36",
    'X-Requested-With': "XMLHttpRequest"
}

logUrl  = "https://www.yiichina.com/login"
signUrl = "https://www.yiichina.com/registration"
r_session = requests.Session()

def sign():
    # 登陆部分
    page = r_session.get(logUrl)

    soup = BeautifulSoup(page.text, ("html.parser"))

    _csrf = soup.find('input', {'name': '_csrf'}).attrs['value']

    data = {
        "_csrf": _csrf,
        "LoginForm[username]" : "********@qq.com", # 换成自己的帐号
        "LoginForm[password]" : "*******",		   # 换成自己的密码
        "LoginForm[rememberMe]": 0,
        "LoginForm[rememberMe]": 1,
        "login-button": ""
    }

    r_session.post(logUrl, data=data, headers=headers)
    # print(r_session.get('https://www.yiichina.com/user').text)
	
    # 签到部分
    page2 = r_session.get(signUrl)

    soupSign = BeautifulSoup(page2.text, ("html.parser"))

    _csrf2 = soupSign.find('meta', {"name": "csrf-token"}).attrs['content']

    signData  = {
        "_csrf" : _csrf2
    }

    r_session.post(signUrl, data=signData, headers=headers2)


if __name__ == "__main__":
    sign()

```

都说是**自动**，那肯定就不止这个脚本的。

这里使用 Linux 的**定时任务**，不清楚的先直接  google 一下

使用 `crontab -e` 进入编辑模式，把下面的命令添加到里面去

```shell
# 每天早上九点自动签到 yii
0 9 * * * /usr/bin/python3.6 ~/yii_sign.py > /dev/null 2>&1 &
```

> 要使用自己相对应的路径