---
layout: post
title: jaydebeapi
subtitle: A python module to connect database by jdbc
date: 2021-07-21
author: BF
header-img: img/bf/python.jpg
catalog: true
toc: true
categories: Programs
tags:
  - python
  - database
  - jdbc
---

### JayDeBeApi

一般来说，不同的数据库针对`Python`都有对应的Module去访问，他们基本上都使用统一的Python DB-API,除了连接上会有点区别，别的使用方式都基本一样。

| 数据库     | 模块        |
| ---------- | ----------- |
| mysql      | pymysql     |
| sql server | pymssql     |
| Oracle     | cx_Oracle   |
| Teradata   | teradatasql |
| etc...     | etc...      |

但不可避免的，有些库需要系统安装一些依赖，比如Oracle，就需要安装有对应的oralce client，cx_Oracle才能正常使用。 此外很多数据库提供odbc driver，那么就可以在安装driver之后，统一使用`pyodbc`去连接。在PA的写的在windows下query hive和impala数据的小工具，使用的就是`pyodbc`的方式。

但是当需要安装除模块本身外的依赖都会显得有些麻烦，今天主要是记录一下在python中使用`jdbc`(jar包)的方式连接`sql server`例子，使用的module就是[`JayDeBeApi`](https://pypi.org/project/JayDeBeApi/)

> The JayDeBeApi module allows you to connect from Python code to databases using Java JDBC. It provides a Python DB-API v2.0 to that database.
>
> It works on ordinary Python (cPython) using the JPype Java integration or on Jython to make use of the Java JDBC driver.
>
> In contrast to zxJDBC from the Jython project JayDeBeApi let’s you access a database with Jython AND Python with only minor code modifications. JayDeBeApi’s future goal is to provide a unique and fast interface to different types of JDBC-Drivers through a flexible plug-in mechanism.

<!-- more -->
### 安装

```shell
pip install JayDeBeApi
```

### 使用
针对不同的数据库，保证连接的字符串、注册的drvier class和对应的Jar包都对应的上就行。

这边我本地的sql server 2017的版本，所以我去[官网](https://docs.microsoft.com/zh-CN/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017)就下了`mssql-jdbc-7.4.1.jre8.jar`.
```Python
import jaydebeapi
url = 'jdbc:sqlserver://localhost:1433;databasename=BFTest'
user = 'xiche'
password = 'xxxxxx'
dirver = 'com.microsoft.sqlserver.jdbc.SQLServerDriver'
jar = './lib/mssql-jdbc-7.4.1.jre8.jar'
sql = 'select * from Test_Output'
conn = jaydebeapi.connect(dirver, url, [user, password], jar)
curs=conn.cursor()
curs.execute(sql)
result=curs.fetchall()
print(result)
curs.close()
```

output:
![output1](/img/post/2021/07/2021-07-23-jaydebeapi-output-01.png)

### pandas
一般我们拿到connection之后，可以直接传给pandas，让他来存储和处理数据，非常方便。
```Python
import pandas as pd
import jaydebeapi
url = 'jdbc:sqlserver://localhost:1433;databasename=BFTest'
user = 'xiche'
password = 'xxxxxx'
dirver = 'com.microsoft.sqlserver.jdbc.SQLServerDriver'
jar = './lib/mssql-jdbc-7.4.1.jre8.jar'
sql = 'select * from Test_Output'
conn = jaydebeapi.connect(dirver, url, [user, password], jar)
df = pd.read_sql_query(con=conn,sql=sql)
print(df)
```
output:
![output1](/img/post/2021/07/2021-07-23-jaydebeapi-output-02.png)

### 最后
由于Java使用的广泛性和Python本身的一些局限性，如果遇到问题时有使用Jar包的解决方案，还是比较舒服的。
