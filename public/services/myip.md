### 心动 myip 服务接口文档

本接口可以返回请求者的公网IP地址。提供 http 和 tcp 两种调用形式。

#### APIs

| 端口 | 协议 | 请求 | 返回 |
| ------ | ------ | ------ | ------ |
| 1053 | http | http://myip.xdapp.com:1053/myip | 字符串，请求者的IP |
| 1154 | tcp  | tcp://myip.xdapp.com:1154 | 字符串类型，请求者的IP + '\\n'，并关闭连接 |

#### 测试 http 接口
```bash
#curl http://myip.xdapp.com:1053/myip
```

#### 测试 tcp 接口
```bash
#nc myip.xdapp.com 1154
```

_如需自行部署可以参考 [项目源代码](https://github.com/tomasen/httpdns)_
