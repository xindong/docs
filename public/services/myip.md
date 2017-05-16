### 心动 myip 服务接口文档

本接口可以返回请求者的公网IP地址。提供 http 和 tcp 两种调用形式。

#### APIs

| 端口 | 协议 | 请求 | 返回 |
| ------ | ------ | ------ | ------ |
| 443 | https | https://ip.xindong.com/myip | 字符串类型，请求者的IP |
| 443 | https | https://ip.xindong.com/myloc | JSON格式，请求者的地区信息 |

#### 测试 http 接口
```bash
#curl https://myip.xindong.com/myip
```

