# 心动 httpdns 服务接口文档

本接口可以查询特定域名对应的IP地址。

## APIs

| 端口 | 协议 | 请求 | 返回 |
| ------ | ------ | ------ | ------ |
| 1053 | http | http://httpdns.xdapp.com:1053/dns?d={$domain} | 字符串，该域名的ip地址 |
| 1153 | tcp  | tcp://httpdns.xdapp.com:1153 字符串类型，$domain + "\\n" | 字符串，该域名的ip地址 |


## 通过 IP 调用

	使用 54.223.165.195 代替域名 httpdns.xdapp.com 

## 测试 http 接口

	`curl "http://httpdns.xdapp.com:1053/dns?d=tomasen.org"`

## 测试 tcp 接口

	`printf "tomasen.org\n" | nc httpdns.xdapp.com 1153`

-
_如需自行部署可以参考 [项目源代码](https://github.com/tomasen/httpdns)_
