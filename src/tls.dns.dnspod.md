---
date: 2018-09-19 23:38:28 +0800
title: "tls.dns.dnspod"
sitename: "Caddy中文文档"
---

# tls.dns.dnspod

允许从DNSPod使用DNS记录获取证书，进行域名管理。

[完整文档](https://github.com/caddyserver/dnsproviders/blob/master/README.md)

## 示例

__用法__

```caddy
tls {
    dns dnspod
}
```

在你的tls指令中使用这个。