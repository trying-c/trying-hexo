---
title: MySQL ERROR 1698 解决指南：root用户访问被拒绝问题
tags:
  - MySQL
  - 避坑指南
abbrlink: 32970
date: 2025-05-29 11:59:11
---

> 当在 Ubuntu 系统中安装 MySQL 后，尝试使用`mysql -u root -p`登录时，可能会遇到"ERROR 1698 (28000): Access denied for user 'root'@'localhost'"错误。本文将详细解析问题原因并提供完整解决方案。

## 问题描述

在终端执行以下命令时：

```bash
mysql -u root -p
```

系统返回错误：

```
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

## 问题原理

该问题通常发生在安装新版本 MySQL 后：

1. MySQL 为 root 账户设置了随机密码而非空密码
2. 默认认证插件设置为`auth_socket`而非`mysql_native_password`
3. `auth_socket`插件要求系统用户名与 MySQL 用户名匹配才能登录

## 完整解决方案

### 1. 使用 debian-sys-maint 用户登录

查看自动生成的维护账户信息：

```bash
sudo cat /etc/mysql/debian.cnf
```

输出示例：

```
[client]
host     = localhost
user     = debian-sys-maint
password = 7F6TVXxve2hh4EHI
socket   = /var/run/mysqld/mysqld.sock
```

使用获取的用户名和密码登录：

```bash
mysql -u debian-sys-maint -p
```

### 2. 查看用户认证插件

登录 MySQL 后执行：

```sql
USE mysql;
SELECT user, plugin FROM user;
```

输出示例：

```
+------------------+-----------------------+
| user             | plugin                |
+------------------+-----------------------+
| root             | auth_socket           |
| mysql.session    | mysql_native_password |
| mysql.sys        | mysql_native_password |
| debian-sys-maint | mysql_native_password |
+------------------+-----------------------+
```

### 3. 修改 root 认证插件

```sql
UPDATE user SET plugin='mysql_native_password' WHERE user='root';
FLUSH PRIVILEGES;
```

### 4. 设置 root 密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_new_password';
FLUSH PRIVILEGES;
```

### 5. 退出并重启服务

```sql
EXIT;
```

重启 MySQL 服务：

```bash
sudo service mysql restart
```

### 6. 使用 root 登录验证

```bash
mysql -u root -p
```

输入新设置的密码，成功登录即表示问题解决。

## 原理说明

MySQL 5.7+版本在 Ubuntu 系统中默认使用`auth_socket`插件进行 root 用户认证。该插件：

- 不验证密码，而是验证发起连接的系统用户名
- 要求系统用户名必须与 MySQL 用户名完全匹配
- 导致传统密码登录方式失效

通过将认证插件改为`mysql_native_password`，我们恢复了传统的密码验证方式，使`mysql -u root -p`命令可以正常使用。

## 总结

当遇到 ERROR 1698 访问被拒绝错误时，关键步骤是：

1. 使用`debian-sys-maint`维护账户登录
2. 修改 root 用户的认证插件
3. 设置新的 root 密码
4. 重启 MySQL 服务

此解决方案适用于 Ubuntu 系统上安装的 MySQL 5.7 及以上版本，其他 Linux 发行版可能需要调整具体步骤。

**参考资源**：[出现 ERROR 1698 (28000): Access denied for user 'root'@'localhost' 的解决方法](https://blog.csdn.net/julielele/article/details/84028405)
