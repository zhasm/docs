---
title: "清理日志"
date: 2019-07-19T20:59:00+08:00
weight: 10
description: >
  介绍如何清理各个组件的服务日志等
---


平台上各个服务的日志日积月累数据量都不小，需要定期清理。由于采用了分表保存的机制，可以将旧表drop来删除日志数据，释放空间。

但是，需要注意的是，只有在mysql开启了innodb_file_per_table=ON的选项后，drop表之后，会删除对应的数据库文件，从而释放出表空间。如果mysql未开启innodb_file_per_table选项，则mysql所有数据都保存在/var/lib/mysql/ibdata1这个文件中，即使删除了表，也不会立即释放表空间。

以下命令用来查看各个服务的日志表的情况：

```bash
climc logs-splitable --service <service_name>
```

其中，service可以是：

* compute
* image
* identity
* log
* cloudevent

以下命令清理各个服务的过期日志表（默认drop清理6个月之前的日志表）：

```bash
climc logs-purge --service <service_name>
```
