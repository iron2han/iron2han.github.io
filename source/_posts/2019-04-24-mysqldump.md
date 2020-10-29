---
layout: post
title:  "MySQL之mysqldump - 备份导出MySQL数据"
categories: MySQL
tags: MySQL 备份
excerpt: mysqldump是MySQL执行备份的工具，它可以生成SQL,CSV,XML,其他格式的备份文件
---



### MySQL之mysqldump - 备份导出MySQL数据

###### *The mysqldump client utility performs logical backups, producing a set of SQL statements that can be executed to reproduce the original database object definitions and table data. It dumps one or more MySQL databases for backup or transfer to another SQL server. The mysqldump command can also generate output in CSV, other delimited text, or XML format.*

<br>

mysqldump是MySQL执行备份的工具，它可以生成SQL,CSV,XML,其他格式的备份文件

---

#### 使用姿势 (Invocation Syntax)

+ db_name 数据库名
+ tal_name 表名
+ options&nbsp;&nbsp;&nbsp;更多姿势(插入位置)



1、备份单、多个表

&emsp;&emsp;$ mysqldump [options] db_name tal_name...

2、备份数据库

&emsp;&emsp;$ mysqldump [options] --databases 数据库名

3、备份全部数据库

&emsp;&emsp;$ mysqldump [options] --all-databases

**转储数据库不要在db_name后添加tbl_name，或者使用--databases，--all-databases**

#### 更多姿势 (Option Syntax)

##### 1、Options

| Format    | Desc             | Raw Desc                             |
| --------- | ---------------- | ------------------------------------ |
| --no-data | 不导出表内容     | Do not dump table contents           |
| --xml     | 导出格式 XML     | Produce XML output                   |
| --version | 输出版本号并退出 | Display version information and exit |

##### 2、Connection Options

| Format          | Desc                        |
| --------------- | --------------------------- |
| --host          | 连接远程MySQL服务器         |
| --compress      | 远程MySQL支持压缩则使用压缩 |
| -p[password]    | 远程MySQL登入密码           |
| -u[user_name]   | 远程MySQL登入用户名         |
| --port=port_num | 远程MySQL端口               |