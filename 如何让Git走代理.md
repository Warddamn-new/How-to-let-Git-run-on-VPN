## 如何让Git走代理

## 0.前言
前段时间用github的时候，突然发现push不下来，然后报错误如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/db31b808ba264c08aa1998ad228b5b80.png)
排除了端口的问题，发现是网络的问题。

通过 命令

```cpp
ssh -Tvvv git@github.com
```
发现github的DNS被解析成本地回环地址0.0.0.1

因此导致了网络的错误。

## 1 解决方法
解决方法有两个：分为有走代理和无代理。

## 无代理
对于无走代理的，需要通过hosts强制把github的ip地址匹配到能用的网址。

首先，通过[ip查询链接1](https://www.ipaddress.com/)、[ip查询链接2](https://ping.chinaz.com/github.com)查到能用的github.com的ip地址

然后在系统中把host匹配上

详情请参见如下链接：
https://blog.csdn.net/qq_40328147/article/details/119619632
以及
https://zhuanlan.zhihu.com/p/113454460

关于如何修改hosts参照如下链接：
①[windows修改文件权限](https://blog.csdn.net/u012102536/article/details/90751519)
②[找到host文件](https://jingyan.baidu.com/article/9113f81b49ed2f2b3214c7fa.html)
③[host修改方式](https://blog.csdn.net/qq_37878579/article/details/78314735)
④[修改host后立即生效](https://blog.csdn.net/qq_44868502/article/details/104340163)


## 有代理
因为我本身是常年挂了代理的，而挂了CFW代理，开了system proxy模式之后，所有的流量都是先通过CFW来判断是直连还是服务器。

因此，如果开了代理，连github.com的话，代理会自动帮你解析DNS为最近的且能够用的ip地址

然而，我开了代理之后，然后执行ssh -Tv git@github.ciom 命令去连github。在看CFW的Log界面时，发现根本都没有走到代理。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6d07440c1e74777a0e517a46f014af0.png)

因此说明问题出在了Git的代理配置上。

那么如何让Git走代理，从而使得在Git Bash访问Github的时候能够自动解析呢？

主要有两种设置：Http和SSH

我们知道从gtihub进行clone or push  pull操作时，有两种方式，一种是http,另一种是ssh

由于平常我两种方式都在用，因此，我希望通过http和ssh都能够走代理，然后自动解析，因此设置如下：

 - 对于Http

打开git bash

输入

```cpp
git config --global https.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890
# or
git config --global http.https://github.com.proxy http://127.0.0.1:7890
git config --global https.https://github.com.proxy http://127.0.0.1:7890

```
对应的取消代理命令为：

```cpp
git config --global --unset http.proxy
git config --global --unset https.proxy
or
git config --global --unset http.https://github.com.proxy
git config --global --unset https.https://github.com.proxy

#取消完后可以利用git config --list查看proxy参数是否已经消失删除
```

第一种命令是指，无论国内国外，只要用了git的http命令，那么都走代理【CFW的代理端口是7890】

第二种是只对github进行管理，不会对国内仓库产生影响，由于我自己也会用到国内的Gitte，所以选择了第二种方法。

设置完之后就可以美美的通过git clone https来使用github啦

详情可以参见：
① [链接1](https://www.cnblogs.com/tofengz/p/15013629.html)
② [链接2](https://zhuanlan.zhihu.com/p/113454460)

 - 对于SSH：

需要在~/.ssh/下新建一个config文件
config文件可以通过在此目录下打开git bash 输入命令 touch config 生成

然后用记事本打开或者其它软件打开

输入如下信息：

```cpp
Host github.com
# 此行要改路径 代理ip 及port
ProxyCommand "C:\Program Files\Git\mingw64\bin\connect.exe" -H 127.0.0.1:7890 %h %p
HostName %h
Port 22
User git
#此行要改路径及id_dsa 或id_rsa
IdentityFile  ~/.ssh/id_rsa 
IdentitiesOnly yes
```

然后保存，接着就可以通过ssh链接github啦

详情请参见：
①`[链接1](https://github.com/Fndroid/clash_for_windows_pkg/issues/1563) jensenojs 的回答
② [链接2](https://zhuanlan.zhihu.com/p/113454460) 

取消代理的话，直接删除config文件即可

如果修改config文件还不成功，可以参考下面链接的方法3：
③ [链接3](https://blog.csdn.net/qq_52293358/article/details/124535927)


以上，即可完美的使用git连接github

番外：
① [如何找到电脑中的git config文件](https://zhuanlan.zhihu.com/p/386905287)
② [如何配置git走代理【模糊介绍版】](https://www.jianshu.com/p/739f139cf13c)
