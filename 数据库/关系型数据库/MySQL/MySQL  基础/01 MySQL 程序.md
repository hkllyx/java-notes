- [概述](#概述)
- [服务器程序](#服务器程序)
  - [mysqld](#mysqld)
  - [mysqld_safe](#mysqld_safe)
  - [mysql.server](#mysqlserver)
  - [mysqld_multi](#mysqld_multi)
- [安装或升级过程中执行设置操作的程序](#安装或升级过程中执行设置操作的程序)
  - [comp_err](#comp_err)
  - [mysql_secure_installation](#mysql_secure_installation)
  - [mysql_ssl_rsa_setup](#mysql_ssl_rsa_setup)
  - [mysql_tzinfo_to_sql](#mysql_tzinfo_to_sql)
  - [mysql_upgrade](#mysql_upgrade)
- [连接到服务器的客户端程序](#连接到服务器的客户端程序)
  - [mysql](#mysql)
  - [mysqladmin](#mysqladmin)
  - [mysqlcheck](#mysqlcheck)
  - [mysqldump](#mysqldump)
  - [mysqlimport](#mysqlimport)
  - [mysqlpump](#mysqlpump)
  - [mysqlsh](#mysqlsh)
  - [mysqlshow](#mysqlshow)
  - [mysqlslap](#mysqlslap)
- [管理和实用程序](#管理和实用程序)
  - [innochecksum](#innochecksum)
  - [myisam_ftdump](#myisam_ftdump)
  - [myisamchk](#myisamchk)
  - [myisamlog](#myisamlog)
  - [myisampack](#myisampack)
  - [mysql_config_editor](#mysql_config_editor)
  - [mysqlbinlog](#mysqlbinlog)
  - [mysqldumpslow](#mysqldumpslow)
- [开发实用程序](#开发实用程序)
  - [mysql_config](#mysql_config)
  - [my_print_defaults](#my_print_defaults)
- [杂项实用程序](#杂项实用程序)
  - [lz4_decompress](#lz4_decompress)
  - [perror](#perror)
  - [zlib_decompress](#zlib_decompress)

# 概述

- 每个MySQL程序都有许多不同的选项。大多数程序都提供了一个--help选项，您可以使用该选项来描述程序的不同选项。例如，mysql --help。
- 可以通过在命令行或选项文件中指定选项来覆盖MySQL程序的默认选项值。有关调用程序和指定程序选项的一般信息。

# 服务器程序

MySQL服务器mysqld是完成MySQL安装中大部分工作的主程序。服务器随附有几个相关的脚本，可帮助您启动和停止服务器：

## mysqld

SQL守护程序（即MySQL服务器）。要使用客户端程序，必须运行mysqld，因为客户端可以通过连接到服务器来访问数据库。

## mysqld_safe

服务器启动脚本。mysqld_safe尝试启动mysqld。

## mysql.server

服务器启动脚本。该脚本在使用System V-style运行目录的系统上使用，该目录包含启动特定运行级别的系统服务的脚本。它调用mysqld_safe启动MySQL服务器。

## mysqld_multi

服务器启动脚本，可以启动或停止系统上安装的多个服务器。

# 安装或升级过程中执行设置操作的程序

## comp_err

在MySQL构建 / 安装过程中使用该程序。它从错误源文件编译错误消息文件。

## mysql_secure_installation

该程序使您可以提高MySQL安装的安全性。

## mysql_ssl_rsa_setup

如果缺少安全文件，此程序将创建支持安全连接所需的SSL证书和密钥文件以及RSA密钥对文件。mysql_ssl_rsa_setup创建的文件 可用于使用SSL或RSA的安全连接。

## mysql_tzinfo_to_sql

该程序mysql使用主机系统zoneinfo数据库（描述时区的文件集）的内容将时区表加载到 数据库中。

## mysql_upgrade

在MySQL升级操作之后使用此程序。它使用在新版本的MySQL中所做的任何更改来更新授权表，并检查表中的不兼容性并在必要时进行修复。

# 连接到服务器的客户端程序

## mysql

用于以交互方式交互式输入SQL语句或从文件中以批处理模式执行它们的命令行工具。

## mysqladmin

客户端，它执行管理操作，例如创建或删除数据库，重新加载授权表，将表刷新到磁盘以及重新打开日志文件。mysqladmin还可以用于从服务器检索版本，进程和状态信息。

## mysqlcheck

一个表维护客户端，用于检查，修复，分析和优化表。

## mysqldump

一种将MySQL数据库以 .sql，.txt或 .xml格式转储到文件中的客户端。

## mysqlimport

客户端使用LOAD DATA将文本文件导入到各自的表中。

## mysqlpump

客户端将MySQL数据库作为SQL转储到文件中。

## mysqlsh

MySQL Shell是MySQL Server的高级客户端和代码编辑器。除了提供的类似于mysql的SQL功能外，MySQL Shell还提供JavaScript和Python的脚本功能，并包括与MySQL一起使用的API。X DevAPI使您能够使用关系数据和文档数据，将MySQL用作文档存储。AdminAPI使您可以使用InnoDB集群。

## mysqlshow

显示有关数据库，表，列和索引的信息的客户端。

## mysqlslap

旨在模拟MySQL服务器的客户端负载并报告每个阶段的时间的客户端。它就像多个客户端正在访问服务器一样工作。

# 管理和实用程序

## innochecksum

脱机InnoDB脱机文件校验和实用程序。

## myisam_ftdump

一个实用程序，用于显示有关MyISAM表中全文索引的信息。

## myisamchk

描述，检查，优化和修复MyISAM表的实用程序。

## myisamlog

处理MyISAM日志文件内容的实用程序。

## myisampack

压缩MyISAM表以生成较小的只读表的实用程序。

## mysql_config_editor

一种实用程序，使您可以将身份验证凭据存储在名为的安全，加密的登录路径文件中 .mylogin.cnf。

## mysqlbinlog

从二进制日志读取语句的实用程序。二进制日志文件中包含的已执行语句的日志可用于帮助从崩溃中恢复。

## mysqldumpslow

用于读取和总结慢查询日志内容的实用程序。

# 开发实用程序

## mysql_config

一个shell脚本，产生编译MySQL程序时所需的选项值。

## my_print_defaults

一个实用程序，用于显示选项文件的选项组中存在哪些选项。

# 杂项实用程序

## lz4_decompress

一个实用程序，用于解压缩使用LZ4压缩创建的mysqlpump输出。

## perror

该实用程序显示系统或MySQL错误代码的含义。

## zlib_decompress

一个实用程序，用于解压缩使用ZLIB压缩创建的mysqlpump输出。