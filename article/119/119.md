# 解决 Steam、Xbox、Ubisoft 代理下的使用问题

本文主要基于 Windows 系统，其他系统请自行确认。<br>
**本文尚不完整，但我不一定什么时候能写完。先发出来，能帮到一些人。**<br>
同时也是整理我自己的思路。如果你有更好的方案，请不吝赐教。

## 玩游戏的代理方案

要解决代理引起的问题，首先要明白代理的工作原理、数据包会被怎样处理。

### 玩游戏需要什么代理

**支持 UDP 转发**：UDP 是网络游戏通信的主要手段。

**网络质量**：全程要求丢包少、低延迟。

### 代理的服务端

**主要有这几种情况**：

**自建代理或翻墙机场的普通节点**：线路质量一般都不足以满足游戏要求的丢包少、低延迟。机场节点可能不支持 UDP 转发。而自建代理也需要一定的网络知识才能配置正确。<br>

**机场的游戏节点**：没用过，待补充。

**游戏加速器**：原理上也是境内服务器中转流量到境外，专门对延迟、丢包做了优化。<br>
（《游戏加速器的工作原理》还在研究中。）

**已经买了翻墙服务，有必要再买游戏加速器吗？**<br>
游戏加速器跟中转机场的工作原理是类似的，运营方式也是类似的（境内服务器、境外服务器、运维）。<br>
游戏加速器是合法生意，机场是非法生意。<br>
综上，同价格下，机场要承担更大风险，无法提供更好的服务。<br>
上面这一段只是理论分析，我没怎么用过机场，欢迎读者让我长见识。

### 代理的客户端

**游戏一般都不支持配置代理？**<br>
我主要玩单机游戏。在我玩过的网络游戏中，我确实想不到哪个游戏平台客户端、游戏是支持配置代理的。<br>
欢迎读者列举支持配置代理的游戏，我会收集在这里。

**本地用哪种方式进行代理？**<br>
系统代理、TUN / TAP，还是透明代理？<br>
不同的代理方式会带来不同的问题。<br>
如果使用游戏加速器的话，是否与正在使用的代理冲突？

#### 劫持软件网络调用（LSP、WFP）

待补充<br>
Layered Service Provider<br>
例如：Proxifier

#### 系统代理

使用时简单明了，灵活、侵入性小。但可能也是导致问题最多的。<br>
软件一般默认，甚至只支持 HTTP 代理而非 SOCKS5，而 HTTP 代理不支持转发 UDP。<br>
而且，由于 HTTP 代理是应用层代理（七层代理），代理软件可能会修改 HTTP 报文，导致应用故障。

#### TUN / TAP（本机）

待补充<br>
我目前需求还不强烈，没有仔细研究。

#### 透明代理

待补充<br>
游戏加速器的路由器插件，属于这种模式（我自己分析的，不一定正确）。添加路由表、iptables 劫持流量，TUN 响应 fake IP。只代理游戏流量，其他直连。代理规则不透明，且不支持配置。

#### VPN

待补充<br>

### 分流规则（路由）

除了游戏加速器，其他的代理方案都需要配置分流规则。<br>

**游戏域名的代理需求复杂**<br>
- 下载游戏要直连
- 游戏社区（steamcommunity.com）要代理
- 游戏通信：到服务端的要代理，P2P 的境内直连、境外代理
- 游戏商店：有时要直连，有时要代理（国区、跨区购买）。

这是游戏玩家使用代理时的一个主要的麻烦来源。<br>
如果玩一些不够热门的游戏，那一般也找不到资料总结它的分流规则。于是就需要自己抓包研究。且不说所需的网络知识，自己研究的这个过程就挺麻烦的，而且浪费时间。<br>

**有网友维护规则，但域名都不够完善**：

- [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community/blob/master/data/ubisoft)
- Loyalsoldier：[v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) （设置类别 `geosite:category-games@cn`为直连）、[clash-rules](https://github.com/Loyalsoldier/clash-rules)
- 某个禁止宣传的 GitHub 库
- [Sukka Ruleset](https://github.com/SukkaW/Surge/)

**UsbEAm Hosts Editor（UHE）**<br>
这是网友 [Dogfight360](https://www.dogfight360.com/blog/) 在持续维护软件。我简单分析一下（不一定正确）：<br>
工作原理是通过改 Hosts 选择合适的 CDN 服务器来解决下载、使用问题。它不是代理，它只是帮你优选 IP。对于没有翻墙代理，也没有游戏加速器的玩家，这个软件非常棒。但它有这些局限性：

- 不管怎么优选 IP 也还是直连，无法解决：自己宽带差、国际线路丢包等问题
- 域名不全
- Hosts 文件不支持泛解析。遇到多个子域名的服务，维护困难。

## Steam

### Steam 客户端下载与访问分流 / 下载使用境内服务器

**本段内容于 2024-12 月验证**<br>

Steam 客户端支持系统代理，不支持手动配置代理。Steam 客户端会自动判断所在区域，并分配最近的下载服务器；可以在 `设置 - 下载 - 下载地区` 确认。或者`运行(Win+R)` `steam://open/console`，打开 Steam 控制台，输入 `user_info` ，回车，查看 IPCountry。<br>
使用「绕过大陆」路由时，会发现下载地区也变到了代理所在地区，这时下载游戏会消耗代理节点的流量。<br>
v2ray、Xray 配置以下域名直连，其他代理，即可实现 Steam 下载与访问分流：
```
domain:steamserver.net,
geosite:steam@cn,
```

[steam@cn](https://github.com/v2fly/domain-list-community/blob/master/data/steam) 包括哪些域名，可点击链接查看。

**配置逻辑**（转载自 [zhanbao2000](https://github.com/2dust/v2rayN/issues/1361#issuecomment-1856192253) ）：
> 1. steam **客户端通过 steamserver.net 最终判断下载位置**，该域名在 [geosite:steam](https://github.com/v2fly/domain-list-community/blob/master/data/steam) 中，且没有 @cn 标记。
> 2. 若 steamserver.net 经过代理，则 steam 会从 steamcontent.com 域名下载游戏（例如 cache1-hkg1.steamcontent.com）。
> 3. 若 steamserver.net 不经过代理，则 steam 会从 xz.pphimalayanrt.com 域名（阿里云）下载游戏，该域名在 geosite:steam@cn 中。
> 4. 该操作不影响 steamcommunity.com 等没有 @cn 标记的域名，这些域名依旧会被代理。

## Xbox

### Windows 无法打开 Xbox 游戏（XGP / PGP）

#### 如果是开启系统代理后无法打开

**本段内容于 2024-12 月验证**<br>

游玩 XGP 游戏时，需要验证帐户身份（「是否允许此应用访问你的信息?」）才能打开游戏。而开启系统代理后，验证身份的那个页面必然报错：

> 发生了错误<br>
> 请稍后再试。<br>
> 0X80190001<br>
> 发送反馈

Windows 上的 Xbox 应用是 **UWP 应用，网络环境默认是隔离的，需要解除回环限制能才访问系统代理**。<br>
可以用 Fiddler（v2rayN、Clash 等）附带的 **EnableLoopback.exe**（解除 Win10 UWP 应用回环代理限制）。<br>
打开软件，在需要解除限制的应用前打勾，最后 **Save Changes**，就可以了。<br>
但是，具体要给哪些应用取消限制，我在网络上还没发现能解决我的问题的方法。这个问题持续了 2 年多，最近被我偶然解决了。<br>

**对于 Xbox 应用**：
- 只勾选 Xbox 这一个就行，其他的 Xbox 开头的应用，没发现有代理的必要。勾选后可以加快打开 Xbox 应用的速度，及应用内的页面加载速度。
- 而**负责帐户验证的应用是「你的帐户」**，Package 名：`Microsoft.Windows.CloudExperienceHost_cw5n1h2txyewy`。勾选解除限制后，就能开着系统代理，正常打开游戏了。（我在互联网上还没看到有人提过这个）
- 可以顺便把 Microsoft Store、OneDrive、邮件和日历，也勾选

#### 如果不开代理也打不开

微软认证服务的域名是 `licensing.mp.microsoft.com`。<br>
2023-02 月开始，微软的认证服务器 IP（20.197.103.57）被墙（ [例1](https://www.v2ex.com/t/915450)、[例2](https://ngabbs.com/read.php?tid=30667368) ）。2024-12-13 测试，这个 IP 443 端口 TCP 也还是不通。<br>
这个 IP 被墙超过 1 个月之后（具体时间不详），微软才配置 DNS 解析去掉这个 IP。在此期间，只能把域名加入代理，或者使用加速器，才能玩 Xbox 游戏。

## Ubisoft 育碧

### 分流规则

**本段内容于 2024-12 月验证**<br>

育碧的服务器目前是在亚马逊云美国机房，并且会根据源 IP 自动切换商店语言。相关域名没有被 GFW 屏蔽。<br>
以下列出**下载域名**，使用的是 Akamai 的服务，「绕过大陆」下会通过代理下载。<br>
测试用的游戏是《疯狂兔子：编程学院》（免费游戏）。只是简单测试，不一定完整，也不一定适合你的情况。
```
e11196.dscd.akamaiedge.net（主要）
e11196.g.akamaiedge.net
e7923.d.akamaiedge.net
a1961.g2.akamai.net
a109.d.akamai.net
```
这几个域名都没被上文提到的几个分流规则集收录，IP 位于境外。添加直连的时候，最好精确匹配域名，避免影响其他服务。

### Ubisoft Connect 侦测到无法复原的错误且必须关闭

**本段内容于 2024-12 月验证**<br>

> Ubisoft Connect 侦测到无法复原的错误且必须关闭。<br>
> 损毁转储已保存于"C:/ProgramFiles (x86)/Ubisoft/Ubisoft Game Launcher/crashes"

这个转储文件一直都是空的，也没法分析。

系统代理为 HTTP 代理的情况下，打开 Ubisoft Connect 可能是正常的，但下载游戏时必然会出现这个错误。<br>
如果系统代理为 SOCKS5 代理，那 Ubisoft Connect 直接就打不开，报错如下：<br>
> 登录 Ubisoft Connect 时发生错误。请重启客户端后再试。(Error dolphin-028)


如果你使用了代理才出现这个报错，那么 [育碧官方支持](https://www.ubisoft.com/zh-cn/help/connectivity-and-performance/article/error-message-ubisoft-connect-has-detected-an-unrecoverable-error-and-must-shutdown-on-ubisoft-connect-pc/000064974) 的方法是没用的。<br>
这个问题的原因是**育碧客户端对 HTTP 代理的适配不好**（转载自 [shunf4](https://github.com/2dust/v2rayN/discussions/2158#discussioncomment-9312266) ）：<br>
> 这个是 HTTP 响应头部字段的顺序问题，当 Ubisoft Connect 通过 HTTP 代理请求 http://uplaypc-s-ubisoft.cdn.ubi.com/uplaypc/downloads/... 后取得的响应，其头部字段必须按照原响应的头部字段顺序进行排列，否则当某些头字段顺序不符合时就会报这个错。<br>
> 原响应的头部字段顺序排列如下，供参考：

如果你用的是 TUN 模式或透明代理，注意观察下载游戏时有没有走代理。

**我的做法：**<br>
我主要使用系统代理，目前就只有育碧这一个平台需要调整配置，而且也不经常用，所以我不太想花时间研究它的报错。<br>
使用时，关闭系统代理，打开 Ubisoft Connect。之后看需要，因为我的宽带直连也基本能玩，有需要的时候，再开加速器。注意登录平台时，保持 IP 属地固定（直连、代理都行，但是要固定），这样可以减少登录平台时输入账号、密码、验证码的次数（那个「记住本设备」从来没有管用过）。

## Nintendo 任天堂

以前写过一篇以 Switch 为例的网络优化的教程：<br>

《[为什么开了加速器还掉线？《斯普拉遁3》](https://telegra.ph/communications-error-splatoon3-04-10)》

## 更新日志

2024-12-12 初版。本文计划把用代理玩游戏时的常见问题都收录。我目前只知道这么多，先发出来，可以帮助其他玩家。未来学习、研究后会继续完善。