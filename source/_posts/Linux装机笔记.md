title: Linux+Go装机笔记

tags: linux

categories: others

------

** {{ title }}：** <Excerpt in index | 首页摘要>

Linux+Go装机笔记

<!-- more -->

[TOC]

- 安装WMware时选择硬盘，一定要选择单一文件模式，后期好扩容，最好50G以上

- 基本环境WMware 15 + Ubuntu 18.04

- 点虚拟机菜单，安装WMware tools , 

  ```bash
  $ cp WMware tools .tz ~/ 
  $ sudo ./wm***.pl
  ```

- 彩色命令行

  ```shell
  export PS1="\[\e[0;32;40m\]-\#-[\[\e[1;32;40m\]\u\[\e[0;32;40m\]@\h]\[\e[0;36;40m\]\A \[\e[1;35;40m\]\w \[\e[0;32;40m\]\[\e[1;31;40m\]$ \[\e[0;33;40m\]"
  ```

- 更新apt源

  ```shell
  $ sudo apt update
  $ sudo apt upgrade
  ```

## 搜狗输入法的安装

1. 官网下载Linux版搜狗输入法

2. 打开ubuntu的应用商店，搜索**`fcitx`**,将搜到的**`fcitx`**程序也就是小企鹅图标的全部安装上

   ```shell
   $ sudo dpkg -i sogoupinyin_xxx_xxx.deb
   # 会报错，接着执行下面命令
   $ sudo apt --fix-broken install
   ```

3. 安装完成后，我们再双击刚下载的deb程序文件就可以安装了

   ```shell
   $ sudo dpkg -i sougoupinyin_xxx_xxx.deb
   ```

4. 搜狗输入法安装完毕后我们打开命令行

   ```shell
   $ sudo apt remove ibus*
   ```

1. 我们打开设置 -> 区域和语言 -> 管理已安装的语言 -> fcitx -> 应用到整个系统

## MySQL 安装

[https://dev.mysql.com/downloads/file/?id=482263](https://dev.mysql.com/downloads/file/?id=482263)

下载一个mysql-apt-config_0.8.11-1_all.deb

```shell
$ sudo dpkg -i mysql-apt-config_0.8.11-1_all.deb
$ sudo apt update
$ sudo apt upgrade
$ sudo apt-get install mysql-server
```

**中途会让设置密码**

- 卸载

  ```shell
  $ sudo apt-get autoremove --purge mysql-server-5.0
  $ sudo apt-get remove mysql-server
  $ sudo apt-get autoremove mysql-server
  $ sudo apt-get remove mysql-common 
  ```

## Redis 安装

```shell
$ wget http://download.redis.io/releases/redis-4.0.9.tar.gz
$ sudo tar zxvf redis-4.0.9.tar.gz
$ sudo mv redis-4.0.9 /usr/local/redis
$ wget http://downloads.sourceforge.net/tcl/tcl8.6.9-src.tar.gz
$ sudo tar zxvf tcl8.6.9-src.tar.gz
$ sudo mv tcl8.6.9 /usr/local/tcl
$ cd  /usr/local/tcl/unix/
$ sudo ./configure
$ sudo make（时间比较长）
$ sudo make install 

```

## MongoDB 安装

## golang 安装

```shell
$ sudo tar -C /usr/local -xzf go1.11.linux-amd64.tar.gz
$ export PATH=$PATH:/usr/local/go/bin
```





## goland 安装

127.0.0.1            account.jetbrains.com

## sublime text3 安装

- 直接应用商店安装
- 安装控制台

```shell
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

- 常用插件

  - #####  Emmet

    > 一种快速编写html/css的方法
    >
    > 注意：安装Emmet的同时，也会自动安装其依赖PyV8 binary库，安装PyV8库会用较长时间，可以在Sublime左下角看到安装进程状态

  - ##### html5

    > 支持hmtl5规范的插件包
    >
    > 注意：与Emmet插件配合使用，效果更好
    >
    > 使用方法：新建html文档>输入html5>敲击Tab键>自动补全html5规范文档

  - #####  jQuery

    > 支持JQuery规范的插件包

  - #####  SideBarEnhancements

    > 侧栏右键功能增强，非常实用
    >
    > 使用方法(参考链接内容)：<http://www.w3cfuns.com/notes/13810/d9b9ed2fb80785dae88a5344ef0f30d4.html>

  - ##### Ctags

    > 函数跳转，我的电脑上是Alt+点击 函数名称，会跳转到相应的函数

  - ##### Alignment

    > 代码对齐，如写几个变量，选中这几行，Ctrl+Alt+A，哇，齐了

  - ##### SublimeLinter

    > 一个支持lint语法的插件，可以高亮linter认为有错误的代码行，也支持高亮一些特别的注释，比如“TODO”，这样就可以被快速定位。（IntelliJ IDEA的TODO功能很赞，这个插件虽然比不上，但是也够用了吧）

  - ##### ChineseLocalizations

    > 中文语言包

  - ##### A File Icon

    > 图标美化

  - ##### BracketHighlighter

    > 类似于代码匹配，可以匹配括号，引号等符号内的范围。
    >
    > 使用方法：系统默认为白色高亮，可以使用链接所述方法进行自定义配置
    >
    > <http://www.360doc.com/content/14/1111/15/15077656_424301780.shtml>

> ```tx
> 127.0.0.1       www.sublimetext.com
> 127.0.0.1       license.sublimehq.com
> Windows : c:/windows/system32/drivers/etc/hosts
> 
> Linux : /etc/hosts
> 
> Mac : /Private/etc
> ----- BEGIN LICENSE -----
> sgbteam
> Single User License
> EA7E-1153259
> 8891CBB9 F1513E4F 1A3405C1 A865D53F
> 115F202E 7B91AB2D 0D2A40ED 352B269B
> 76E84F0B CD69BFC7 59F2DFEF E267328F
> 215652A3 E88F9D8F 4C38E3BA 5B2DAAE4
> 969624E7 DC9CD4D5 717FB40C 1B9738CF
> 20B3C4F1 E917B5B3 87C38D9C ACCE7DD8
> 5F7EF854 86B9743C FADC04AA FB0DA5C0
> F913BE58 42FEA319 F954EFDD AE881E0B
> ------ END LICENSE ------
> ```

## vim go语言插件

## 安装OpenSSL

## 安装Chrome

1. 将下载源添加到系统源

   ```shell
   $ sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
   ```

2. 导入

   ```shell
   $ wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
   $ sudo apt-get update
   ```

3. 稳定版的安装

   ```shell
   $ sudo apt-get install google-chrome-stable
   ```

4. 启动Chrome，添加收藏夹

   ```shell
   $ ./usr/bin/google-chrome-stable
   ```






