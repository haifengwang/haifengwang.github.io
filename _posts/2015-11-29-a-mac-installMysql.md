---
layout: post
title: Mac 下安装 MySQL
category: mac
description: 在 Mac 下安装 MySQL,本来很简单的过程，由于追求更简洁的配置，因而遇到好多无法预知的问题，最后发现了系统的的秘密。 
keywords: MAC,MySQL,安装,配置
---

在 Mac 下安装开发类的软件，首先想起使用 `Homebrew`,由于看到论坛上说会存在一系列配置性的问题，就选择了从[官网](http://dev.mysql.com/downloads/mysql/)直接下载，傻瓜安装。

### 下载安装

下载有一个比较繁琐的过程，必须注册为 Oracle 的用户，注册需要的信息很多，我只是为了下载一个数据库安装包至于吗？(*也许这传言 MySQL 会闭源，用户好感降低的原因之一吧*)。

下载完成后安装，整个过程很简单直接下一步。到最后一步要注意弹出的**临时密码**，记录下来，用于初次登陆。（*PS:这是最新的版本，之前 MySQL 默认密码为空*）。

### 设置

在 iTerm 下设置

```
~  /usr/local/mysql/bin/mysql -u root -p
Enter password:
（零时密码）
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.7.9

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```

操作：

```
mysql> select database();

```

出现错误：

```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

```

提示必须修改初始密码

修改密码：

```
mysql> SET PASSWORD = PASSWORD('123456');

```

执行后，得到以下信息：

```
Query OK, 0 rows affected, 1 warning (0.00 sec)

```

密码修改成功，下次登录就可以使用新密码了。

现在开始执行一段 `sql` 语句:

```

mysql> select 1;

```

结果：

```
+---+
| 1 |
+---+
| 1 |
+---+
1 row in set (0.00 sec)

```

用刚才设置新密码重新登录，创建一个数据库。

```

mysql> create database wwdb;

```

问题出现了❗️

```
ERROR 1006 (HY000): Can't create database 'wwdb' (errno: 242350896)
```

go 一下大多说权限等问题，我并没有继续针对问题探索，直接卸载重新安装。


####卸载

使用以下脚本

```

sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
vim /etc/hostconfig  (and removed the line MYSQLCOM=-YES-)
rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*

```

执行后查看，清理结束后重新安装。按照以上的配置走了一遍，再创建数据库。

```
mysql> create database firstDb;

```

结果：

```
Query OK, 1 row affected (0.00 sec)
```

成功了😊！

###配置

目前在 `iTerm` 进入 `MySql` 中需要用 `/usr/local/mysql/bin/mysql` 一长段命令，首先想到做一个 `link`.

```
ln -s /usr/local/mysql/bin/mysql /usr/bin

```

结果很失望😞

```
ln: /usr/bin/mysql: Operation not permitted

```

于是尝试管理员权限：

```
sudo ln -s /usr/local/mysql/bin/mysql /usr/bin

```

结果令人沮丧💔

```
ln: /usr/bin/mysql: Operation not permitted

```

尝试另一种方式：通过运行 `sudo vi /etc/bashrc`，在 `bash` 的配置文件中加入 `mysql` 和 `mysqladmin` 的别名


```
alias mysql='/usr/local/mysql/bin/mysql'

alias mysqladmin='/usr/local/mysql/bin/mysqladmin'

```

添加后 `source bashrc` ,当前可用，重新打开一个 tab ，仍然不可用😞。

不知道为什么，有点绝望，查了好多了资料，不但没有解决我的问题，反而给我一种这是正确的错觉。最后想到大概是操作系统的问题，我是用最新版的 **EI Capitan** .

最后发现了这篇文章 [**Mac OS X El Capitan 的 SIP 安全策略**](http://www.sunzhongwei.com/mac-el-capitan-system-integrity-protection.html),虽然没有帮我解决问题，至少知道了什么原因（*PS:就这样自我安慰*）。

文章提到 Mac OS X 10.11 El Capitan 引入的一项安全特性。**SIP**（System Integrity Protection）保护系统进程、文件及目录不被其他进程篡改，即使是 root 用户及拥有 sudo 权限的用户也不可以。 因为大苹果坚信拥有 root 权限的用户是系统安全的隐患。 SIP 限制的目录有 `/System, /bin, /sbin and /usr`。

###MySQL GUI Tools

起初我选择了号称开源免费小巧玲珑的  [**Sequel Pro**](http://sequelpro.com/) 。尝试着新建一个数据库，新建一个表，做了一些简单的操作，作为一个 **SQL Server** 的长期用户，这个工具的确是太简单了。

有没有更好的呢？答『有』， MySQL 官方有一个客户端工具 [**MySQLWorkbench**](http://dev.mysql.com/downloads/workbench/)。使用体验和 `SQL Server` 客户端差不多，功能也很强大，就用它了。

>**注意**：用 MySQLWorkbench 建表时，对字符串类型，需要自定编码，不然会无法插入中文（*错误警告*）。

至此 MySQL 安装完了，准备愉快的玩耍了😊。
