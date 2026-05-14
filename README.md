```markdown
# 代理软件使用 Tips

个人对代理客户端的使用体验和设置总结。

## 一、客户端选择

主要分为 **Clash 类** 和 **Sing-box 类**。机场用户推荐选择 Clash 类。

### Clash 类
- **Windows端**：Clash Verge Rev、Mihomo Party
- **Android端**：Clash Meta for Android、FlClash

### Sing-box 类
- **Windows端**：v2rayN
- **Android端**：v2rayNG

## 二、客户端体验与 Bug

### 1. DNS 泄露问题
- Clash 分流控制较细，但很多机场的默认配置都会产生 DNS 泄露。
- v2rayN 和 v2rayNG 的默认配置均不会产生 DNS 泄露。

### 2. Clash Verge Rev
有时打开主界面会无响应卡死。

### 3. v2rayNG
偶尔导致抖音搜索功能显示无网络，无法稳定复现。  
**解决办法**：启用绕行模式并选择自动下载应用列表时，v2rayNG 可能会把自己也加入绕行列表。排除 v2rayNG 本身的绕行后，抖音搜索功能似乎大部分时间正常。若不开启绕行模式且不代理 v2rayNG 本身，也不会出现异常。

### 4. v2rayN 的 TUN 模式
- 不太稳定，部分系统上无法开启。
- 需要管理员权限。建议在程序兼容性页面勾选“以管理员身份启动”，否则每次打开都会提示 TUN 模式关闭并要求重启。
- Clash Verge 的 TUN 模式不需要管理员权限。

### 5. FlClash（截止 2026 年 5 月的 0.8.92 版）
DNS 覆写功能会产生 DNS 泄露，尝试多种配置均无法解决。相同设置在 Clash Meta for Android 和 Clash Verge Rev 上可正常防泄露。

## 三、Clash 类的 DNS 覆写功能个人理解

### 1. DNS 防泄露是否有意义？
（我也不知道，哈哈哈。）

### 2. 前提
- 关闭 H3、QUIC 协议（这些协议可能会绕过 Clash 发起 DNS 查询，当前代理技术无法处理）。
- 关闭系统 DNS（让 Clash 独占 DNS 查询，避免并行查询导致泄露）。

### 3. Name Server
主 DNS 服务器。当未触发其他规则时，所有 DNS 查询都由 Name Server 发出。  
**建议**：设置为国外的 DNS，并启用 DoH。

### 4. Default Name Server
用于解析其他 DNS 地址（例如主 DNS 配置为 DoH 时，DoH 本身不是 IP 地址，需要一个 IP 型的 DNS 来解析它）。  
**建议**：配置为国内 DNS，如阿里或腾讯的 DNS。

### 5. Proxy Name Server
（部分客户端有此选项）作用类似 Default Name Server，但专门用来解析代理服务器的 IP。  
**建议**：配置为国内 DNS，可使用 DoH/DoT，为加快解析速度也可配置为纯 IP。

### 6. Name Server Policy
控制什么情况下不使用主 DNS，而改用其他 DNS。  
**常见配置示例**：
```yaml
nameserver-policy:
  'geosite:cn':
    - https://dns.alidns.com/dns-query
```
含义：国内域名用阿里 DNS 解析（加快速度），其余域名用主 DNS 解析。

### 7. Fallback Server
属于历史传承功能。现在使用 Name Server Policy 已经能完美分流，但纯 IP 连接的后续验证可能仍会用到。  
**建议**：为防止 DNS 泄露，配置为国外 DoH。  
Fallback Filter 可设置为国内区域（默认黑名单制）。

### 8. Fake IP 与 Redir Host
放在 TUN 模式中讲解。如果只使用浏览器插件连接代理软件而不开启全局代理，这两个模式意义不大。

## 四、TUN 模式

TUN 模式会创建一张虚拟网卡，所有流量进入物理网卡前先经过虚拟网卡，再由 Clash 或 Sing-box 核心处理。  
**为什么需要 TUN 模式？** Windows 平台上系统代理不能解决所有软件的代理问题，尤其是 AI Agent、CLI 版 Agent 以及很多开发工具（只在终端运行）不走系统代理，通过环境变量设置又很麻烦。

> 注：Sing-box 的两个客户端默认基本不会泄露，开启“严格路由”和“mixed”即可。

### 1. Fake IP 模式
Clash 返回一个虚假 IP 给软件，让软件立即连接，同时 Clash 记录虚假 IP 与域名的映射关系。  
- **优点**：即使域名指向的 IP 发生变化（如 CDN），或软件后续直接连接 IP 而不经过域名查询，也能按原本的域名进行分流；能完美解决游戏 UDP 分流问题。  
- **缺点**：内网兼容性不佳，某些银行应用、游戏加速器可能无法工作（加速器需根据真实 IP 判断线路）。

### 2. Redir Host 模式
返回真实 IP，并记录真实 IP 与域名的关系。  
- **优点**：兼容性好（例如 Ak 加速器可正常工作）。  
- **缺点**：如果同一个 IP 对应多个域名，分流会失效。对于简单的国内外分流，问题不大（不太可能一个 CDN 同时加速 Bilibili 和 YouTube）。但如果需要精细分流（如 Google 搜索走香港、YouTube 走日本），就可能出错。

### 3. Normal 模式（两者都不开）
DNS 解析出什么 IP 就按规则分流，后续 IP 变化也不管。

### 总结
- 仅做国内外分流：可使用 Sing-box 类，很简单。  
- 无特殊需求：使用 Normal 或 Redir Host 模式。  
- 机场很给力，想充当游戏加速器：开启 Fake IP 模式，可完美解决游戏 UDP 分流问题。
```
