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
1、
