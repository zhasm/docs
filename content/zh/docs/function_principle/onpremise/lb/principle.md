---
title: "原理介绍"
date: 2021-12-22T15:47:12+08:00
weight: 1
description: >
    介绍负载均衡相关概念
---

## 概念

在使用{{<oem_name>}}提供的负载均衡功能时，会用到以下术语

负载均衡实例（Load Balancer），有时也简称为“实例”。实例与子网关联，创建实例时需要选择子网，实例的IP地址即从该子网中分配。

一个实例下可以有多个负载均衡监听（Listener）。监听的类型通常有HTTP、HTTPS、TCP、UDP，每个监听还需指定一个端口。实例和监听联合起来即表达了服务面向外部的地址、协议和端口。

后端服务器组（Backend Group）、后端服务器（Backend）。每个后端服务器指定了服务后端的IP和端口，同一个服务器组中的多个后端服务器提供相同服务。

监听可指定后端服务器组，当用户访问监听所在地址时，负载均衡的转发节点会将请求转发给服务器组中的具体后端，从而实现负载均衡的效果。

负载均衡集群是一组转发节点LBAgent的集合，一般情况下一个集群下配置两个LBAgent节点互为主备即可，同一集群下的LBAgent节点的VRRP路由ID必须相同。

负载均衡转发节点用于监听负载均衡实例，并将客户端的请求根据监听和转发规则转发给后端服务器。同一个集群下可以有多个转发节点，但同一时刻只有一个节点作为Master节点提供转发服务。属于一个集群的节点的VRRP路由ID必须相同。

节点生命周期管理：

1. 新建节点：设置节点的配置参数。
2. 部署节点：将节点的配置文件下发到指定机器，提供转发功能的机器可称为转发实例，当节点状态为Master时，表示集群中的节点可正常提供负载均衡转发服务。
3. 下线节点：当不需要节点提供服务时，可下线节点，将配置文件从转发实例上删除。
4. 删除节点：删除节点的配置文件信息等。

### HTTP/HTTPS类型监听

对于HTTP/HTTPS类型的监听，我们还可以创建“转发策略（Listener Rule）”，利用HTTP协议的特点，来进一步复用监听的地址和端口。转发策略包含路径、域名属性，用来匹配来自用户的HTTP请求。每个转发策略又可指定自己的后端服务器组，当请求中的路径、域名匹配时，负载均衡会将其转发到此后端组中的服务器。

	          ___ Host: src0.example.com -> srv0 backendGroup
	        /
	HTTP/80
	        \
	          --- Host: srv1.example.com -> srv1 backendGroup

HTTPS类型的监听需要绑定TLS证书。{{<oem_name>}}平台支持RSA、EC2证书

### 访问控制

{{<oem_name>}}支持创建访问控制列表，列表中指定网络CIDR。监听可与访问控制列表关联，并指定将该列表应用为白名单、黑名单。作为白名单时，仅来源IP为列表中地址的请求被接受，其余请求将被拒绝。黑名单时，列表中的网络将被拒绝服务，列表外的地址可访问。
