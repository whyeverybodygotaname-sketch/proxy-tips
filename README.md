# 代理软件使用 Tips

个人对代理客户端的使用体验与配置总结。（满满ChatGPT味，但是内容基本属实，试了好几天才搞明白这些玩意）

---

# 一、客户端选择

目前主流主要分为 **Clash 类** 与 **Sing-box 类**。

对于大多数机场用户，更推荐使用 **Clash 类客户端**，生态更完善、配置资料更多。

## 1. Clash 类

### Windows

* Clash Verge Rev
* Mihomo Party

### Android

* Clash Meta for Android
* FlClash

---

## 2. Sing-box 类

### Windows

* v2rayN

### Android

* v2rayNG

---

# 二、客户端体验与 Bug

## 1. Clash 默认配置容易 DNS 泄露

Clash 类客户端分流控制非常细。

但很多机场提供的默认配置，都会存在 DNS 泄露问题。

相比之下：

* v2rayN
* v2rayNG

默认配置通常不会产生 DNS 泄露。

---

## 2. Clash Verge Rev 主界面偶发卡死

Clash Verge Rev 有时打开主界面会无响应卡死。

核心代理通常正常，只是 GUI 偶发冻结。

---

## 3. v2rayNG 与抖音搜索的玄学 Bug

v2rayNG 偶尔会导致抖音搜索显示“无网络”。

特点：

* 无法稳定复现
* 其他客户端基本不出现
* 视频播放正常，仅搜索异常

### 可能的解决办法

似乎与“绕行模式”有关：

* 开启绕行模式
* 自动下载应用列表
* v2rayNG 可能会把自己加入绕行

此时：

* 将 v2rayNG 本身移出绕行列表
* 抖音搜索大多数情况下恢复正常

但反过来：

* 不开绕行
* 不代理 v2rayNG 本身

似乎也不会导致抖音异常。

目前仍属于比较玄学的问题。

---

## 4. v2rayN 的 TUN 模式稳定性一般

v2rayN 的 TUN 模式在部分系统上可能无法开启。

而且：

* 需要管理员权限
* 建议在兼容性设置中勾选“以管理员身份运行”

否则经常出现：

* 打开后发现 TUN 被关闭
* 又要求重新以管理员身份启动

流程比较绕。

### 对比

Clash Verge Rev 的 TUN 使用体验明显更简单，通常不需要管理员权限。

---

## 5. FlClash 的 DNS 覆写问题

截止 2026 年 5 月的 `0.8.92` 版本：

FlClash 的 DNS 覆写功能会产生 DNS 泄露。

测试情况：

* 尝试多种配置均无法解决
* 相同配置在以下客户端正常：

  * Clash Meta for Android
  * Clash Verge Rev

可能是其他设置问题，也可能是版本 Bug。

后面会详细讲 DNS 覆写。

---

# 三、Clash 类 DNS 覆写功能理解

## 1. DNS 防泄露有没有意义？

不知道。

哈哈哈。

---

## 2. 一些前提

### 关闭 H3 / QUIC

部分情况下：

* HTTP/3
* QUIC

可能绕过 Clash 直接发起 DNS 查询。

当前很多代理方案无法完整处理。

因此建议：

* 禁用 H3
* 禁用 QUIC

---

### 关闭系统 DNS

这个比较容易理解。

如果系统 DNS 与 Clash 并行查询：

* Clash 查一次
* 系统再查一次

那基本一定会 DNS 泄露。

因此：

* 应尽量由 Clash 完全接管 DNS。

---

## 3. 什么是 Name Server

`name-server` 可以理解为：

> 主 DNS 服务器。

当没有命中其它 DNS 规则时：

* DNS 查询都会由它发出。

### 建议

* 使用国外 DNS
* 使用 DoH（DNS over HTTPS）

例如：

```yaml
name-server:
  - https://1.1.1.1/dns-query
  - https://8.8.8.8/dns-query
```

---

## 4. 什么是 Default Name Server

`default-name-server` 的作用：

> 用来解析其它 DNS 服务器本身。

例如：

你的 `name-server` 配置的是：

```yaml
https://dns.google/dns-query
```

但这个地址本身还是域名。

系统必须先：

* 解析 `dns.google`
* 才能连接 DoH

这时候就需要一个纯 IP DNS。

### 建议

使用国内 DNS：

* 阿里
* 腾讯

例如：

```yaml
default-name-server:
  - 223.5.5.5
  - 119.29.29.29
```

---

## 5. 什么是 Proxy Name Server

部分客户端会有：

```yaml
proxy-server-nameserver
```

作用类似 `default-name-server`：

> 专门用于解析代理服务器本身。

例如：

* 机场节点域名
* 中转服务器域名

### 建议

为了提高解析速度：

* 可直接使用纯 IP DNS

例如：

```yaml
proxy-server-nameserver:
  - 223.5.5.5
```

也可以使用：

* DoH
* DoT

---

## 6. 什么是 Name Server Policy

`nameserver-policy`：

> 根据规则决定使用哪个 DNS。

常见配置：

```yaml
nameserver-policy:
  'geosite:cn':
    - https://dns.alidns.com/dns-query
```

意思是：

* 国内域名 → 使用阿里 DNS
* 其他域名 → 使用主 DNS

### 优点

* 国内解析更快
* 国外解析更稳定
* 减少污染

---

## 7. 什么是 Fallback Server

`fallback` 可以理解为：

> 特殊情况下备用的 DNS。

根据目前的理解：

这部分更多属于历史遗留设计。

因为现在：

* `nameserver-policy`
* DNS 分流

已经足够强大。

但某些纯 IP 连接验证场景：

* 仍可能会使用 fallback。

### 建议

为了避免 DNS 泄露：

* fallback 使用国外 DoH

例如：

```yaml
fallback:
  - https://1.1.1.1/dns-query
```

### fallback-filter

通常建议：

* 设置国内区域过滤

默认逻辑更偏向黑名单制。

---

## 8. Fake-IP 与 Redir-Host

这部分放到 TUN 模式讲。

如果：

* 仅浏览器插件代理
* 不开启系统全局代理
* 不开启 TUN

那么：

* Fake-IP
* Redir-Host

意义并不大。

---

# 四、TUN 模式

## 1. 什么是 TUN

TUN 模式本质上是：

> 创建一个虚拟网卡。

数据流程：

```text
应用
 ↓
虚拟网卡（TUN）
 ↓
Clash / Sing-box 内核
 ↓
真实网卡
 ↓
互联网
```

---

## 2. 为什么需要 TUN

Windows 上：

很多程序并不走系统代理。

特别是：

* AI Agent
* CLI 工具
* 开发工具
* Terminal 程序

通过环境变量配置代理：

* 很麻烦
* 兼容性一般

而 TUN 可以直接接管流量。

---

## 3. Sing-box 类默认更省心

Sing-box 系客户端：

* 默认 DNS 处理更完整
* 默认更不容易泄露

通常只需要：

* 开启严格路由
* Mixed 模式

即可。

---

# 五、Fake-IP / Redir-Host / Normal 模式

## 1. Fake-IP 模式

Fake-IP 的原理：

> Clash 给应用返回一个“假 IP”。

例如：

```text
youtube.com
↓
198.18.x.x（假 IP）
```

然后 Clash 会记录：

```text
假 IP ↔ 域名
```

这样即使：

* CDN IP 变化
* 软件后续只连接 IP
* 不再重新查询域名

Clash 仍然知道：

> 这个连接原本属于哪个域名。

### 优点

可以实现非常精准的分流：

* CDN 场景
* UDP 游戏
* 白名单代理

效果很好。

### 缺点

内网兼容性较差。

例如：

* 银行客户端
* 游戏加速器

可能直接失效。

因为加速器需要真实 IP 判断线路。

你给它一个 Fake-IP：

它就会直接罢工。

---

## 2. Redir-Host 模式

Redir-Host 与 Fake-IP 相反：

> 返回真实 IP。

### 优点

兼容性明显更好。

例如：

* 游戏加速器
* 某些银行软件

都更容易正常工作。

实测：

AK 加速器可以与 Redir-Host 共存。

---

### 工作原理

Redir-Host 会记录：

```text
真实 IP ↔ 域名
```

但它依赖：

> “这个 IP 当前对应哪个域名”

因此：

如果多个域名共用一个 IP：

* 分流就可能失效。

---

### 但大多数人其实问题不大

如果只是：

* 国内外分流

通常问题不明显。

因为：

不太可能存在一个 CDN：

* 同时加速 Bilibili
* 又加速 YouTube

---

### 真正容易出问题的情况

例如：

你希望：

* Google Search → 香港节点
* YouTube → 日本节点

这类细粒度分流：

就可能因为 CDN 共 IP 而误判。

但这种需求相对小众。

---

## 3. Normal 模式

如果：

* Fake-IP 不开
* Redir-Host 不开

那就是默认的 Normal 模式。

工作方式：

```text
DNS 解析得到什么 IP
↓
后续直接按 IP 分流
```

后续：

* IP 变化
* CDN 切换

Clash 不再跟踪。

---

# 六、个人建议总结

## 1. 只做国内外分流

推荐：

* Sing-box 类
* 或 Clash + Redir-Host

配置简单。

---

## 2. 不需要精细分流

Fake-IP 的意义并不大。

更推荐：

* Normal
* Redir-Host

兼容性更好。

---

## 3. 需要复杂规则 / UDP / 游戏

Fake-IP 更强。

尤其：

如果机场本身线路质量很高。

甚至：

可以部分替代游戏加速器。

因为 Fake-IP 对 UDP 分流非常友好。
