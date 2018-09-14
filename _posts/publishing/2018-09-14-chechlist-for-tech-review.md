---
layout: post
title:  "游戏发行技术评审Checklist"
date:   2018-09-14 19:07:00 +0800
categories: ['publishing']
---

*尚未完成*

# 技术评审Checklist

## Game Framework

- 如何保证用户数据落地，可容忍的数据丢失范围多大
- 游戏是如何 Scale Out 用户数据的
- 检查所有单点有没有无法扩展的单点数据库和服务
- 在线离线的机制，如何锁状态
- 检查冷数据的处理逻辑有没有问题，好友关系、排行榜等

## OS & Database & Network

- OS和数据盘分离。数据盘可以使用云计算提供商的高性能NAT（阿里云：`NAS`，GCP：`Filestore`）；对于需要强落地的数据，使用`lvm`来扩展本地磁盘
- 是否使用VPC网络，内网（交换机）IP地址是否足够
- 架设或使用云计算提供商的内网 `dnsmasq`、`ntpd-server`，并配置到所有服务器
- 检查OS时区设置
- 如无特殊理由，MySQL使用云计算厂商的托管服务。MySQL 版本以及 Engine 是否为 `InnoDB`（或`XtraDB`等兼容格式）

## Security

- 安全组策略使用白名单，默认`DROP`
- `ssh` 不实用密码，只使用非对称加密秘钥
- VM不设置外网IP
- 使用通用网关架构，申请高防专用域名，并开启2套Loadbalance，其中一套专门用于高防
- 账号一律使用随机生成的16位（以上）字符串，含大小写特殊字符

## Monitor

- 带宽：Loadbalance、VM、Database
- CPU：VM、Database
- 内存：Game Server、Record Server
- 硬盘：VM

## Financial

- 阿里云账号关联至心动主账号
- 阿里云账号信用额度 ≥ 20万
- 阿里云的付费类型为包年包月，并全部开通自动续费，周期1个月
- 公网带宽按量付费，峰值50~100MB
