# mysql密码忘记如何做？

[TOC]



## 0.摘要

在我们使用mysql的时候有可能会忘记root密码，今天就记录下如何重新设置密码。本人用的是mac。 使用的安装方式是brew install mysql，安装的mysql是8.0.15版本。

## 1.步骤

- 1.使用`brew services stop msyql ` 
- 2.进入 `/usr/local/Cellar/mysql/8.0.15/bin` 目录
- 3.执行`./mysql.server stop`
- 4.执行`./mysqld_safe --skip-grant-tables`
- 5.另外一个终端执行`mysql -u root `
- 6.登录msyql后执行`UPDATE mysql.user SET authentication_string='' WHERE User='root';` 重新设置密码。
- 7.重启 mysql 服务，mac 里直接命令把服务关闭。 `./mysql.server stop`   然后执行 `./mysql.server start`。
- 8.用 root 用户登录，因为已经把 authentication_string 设置为空，所以可以免密码登录。 `mysql -u root -p `。
- 9.进入 mysql 库，使用 ALTER 修改 root 用户密码(注意：**我偷懒，没有设置密码， 所以可以直接 mysql -u root登录**)

