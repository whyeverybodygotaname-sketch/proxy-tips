# -tip
个人对代理客户端的使用体验和设置总结。

一、关于客户端选择:
主要是clash类和singbox类。机场用户推荐选择clash类。

clash类:
windows端:clash verge rev和 mihomo party
android端:clash meta for android和flclash
singbox类:
windows端:v2rayn
android端:v2rayng

二、客户端体验或bug:
1、clash分流控制较细，但是很多机场的默认配置都会产生dns泄露。

v2rayn和v2rayng默认配置均不会产生dns泄露。

2、clash verge rev有时打开主界面会无响应卡死。

3、v2rayng有时候会导致抖音搜索功能偶尔显示无网络。无法稳定复现，但是其他客户端稳定不复现。
非常玄乎的bug解决办法是:似乎如果启用绕行模式，并选择自动下载应用列表，v2rayng会把自己搞到绕行里面，排除v2rayng本身的绕行后，抖音搜索功能似乎大部分时间正常。但是反过来不开启绕行模式，也不选择代理v2rayng本身似乎也不会导致抖音异常。

4、flclash 截止2026年5月的0.8.92版，其dns覆写功能会产生dns泄露，尝试多种配置均无法解决。可能是其他地方设置的不对。相同设置在clash meta for android和clash verge rev上均可以防止dns泄露。
后面会讲解dns覆写功能

三、clash类的dns覆写功能个人理解
1、关于dns防泄露有没有意义，我也不知道，哈哈哈。
2、一些小前提，关闭h3，quic协议可能会绕过clash发起dns，当前代理技术不能处理。关闭系统dns，这个比较好理解，控制dns查询当然首先把系统的关掉，如果系统跟clash并行查询，那肯定泄露。
3、什么是name server

name server就是你的主dns服务器，当没有触发其他任何规则的时候，dns查询都由name server发出，建议设置为国外的dns并使用doh。

4、什么是default name server

default name server是用来解析你的其它dns地址的，比如你的主dns配置了一个doh，但doh不是ip无法直接连接，需要一个ip型的dns做解析，这个建议配置为国内dns。比如阿里或者腾讯的。

5什么是proxy name server

有一些客户端有这个选项，作用和default name server类似，但这个是专门用来解析你的代理服务器的ip的，建议配置为国内dns，这个可配置为doh或者dot。个人为加快解析速度配置为纯ip。

6、什么是name server policy
就是控制什么情况下不用主dns而用另外的dns，常见的配置为:
nameserver-policy:
  'geosite:cn':
    - https://dns.alidns.com/dns-query
意思就是如果是国内域名则用这个阿里dns解析，加快解析速度，否则则用name server也就是主dns解析。

7、什么是fallback server
就是在fallback filter里规定的例外情况使用fallback server解析，按照跟ai对话的理解这个属于历史传承。现在使用name server policy已经完美分流用不到这个，但是好像纯ip连接后续验证还是会用到，为防止dns泄露应当配置为国外doh。至于fallback filter就设置为国内区域（默认是黑名单制吧）。

8、fake ip redir host放在tun模式里讲解，如果只用浏览器插件连接到代理软件，而不开启全局代理，这两个模式意义不大。
8、什么是fake ip 什么是redir host
