# Shadowrocket 个人分流配置

配置文件：[`shadowrocket.conf`](shadowrocket.conf)。使用 Shadowrocket 的配置分流模式。
节点由设备上的订阅或已有节点提供；MITM 证书仍由各设备本地管理。

## 分流顺序

1. 个人直连、服务器 IP、EMBY 和个人强制代理例外。
2. 国内 AI 的网页/API 直连。
3. Apple 商店入口、下载及证书服务默认直连；其他 AppleProxy 服务仍走 Apple 代理组。
4. Apple、Spotify 的专用规则。
5. 通用 Proxy 集合。
6. 通用 China 集合。
7. `GEOIP,CN,DIRECT`，最后由默认代理兜底。

通用 Proxy 优先于通用 China：两者会有重叠，例如 `google.cn` 和
`login.microsoftonline.com`。需要覆盖默认行为时，应在前部添加精确例外。
`RULE-SET` 与 `DOMAIN-SET` 是互补文件，Apple、Proxy、China 均应配套使用。

国内 AI 直连包括 DeepSeek、Kimi、Moonshot 国内域名、豆包，以及
`discovery-api.intern-ai.org.cn`。海外域名及中转接口使用各自现有规则；
`always-real-ip` 仅决定 DNS 返回真实 IP，不决定是否走代理。

## DNS 与节点选择

- 默认和直连 DNS 均使用阿里、腾讯 DoH；`no-h3` 只限制 DNS 的 HTTP/3。
- 保留普通 DNS 和系统 DNS 回退以保证可用性；回退不保证加密解析。
- 直连域名解析失败时，不自动改走代理。
- EMBY 节点筛选支持新加坡、台湾的名称、旗帜及有边界的地区缩写。
  HTTPS 204 探测仅判断网络可达性，不代表 Emby 能解锁或流畅播放。
  无匹配节点或播放失败时，在 EMBY 组手动选择默认代理或直连。
- Apple 基础服务组默认直连，仍可手动切换到默认代理。

## 设备验证

Linux 上的格式与规则匹配检查不能代替 Shadowrocket 实机验证。
导入本地配置后，确认远程规则均加载成功，使用配置分流模式，并检查是否有模块或
设备上已保存的策略选择覆盖配置默认行为。重新连接后，在 Wi-Fi、蜂窝网络分别验证：

| 检查对象 | 默认预期 |
| --- | --- |
| 微信、支付宝、淘宝、京东的主站和资源请求 | 直连；打开登录/支付页面即可，无需交易 |
| B站、抖音的视频及图片请求 | 直连，播放和拖动正常 |
| 国内 AI 网页、API | 直连，流式响应正常 |
| Google、海外 AI、linux.do、NodeSeek | 对应代理策略 |
| App Store 入口、下载、Apple OCSP、DigiCert 的 OCSP/CRL/证书下载端点 | Apple 基础组，默认直连 |
| Apple 开发者、TestFlight、Apple TV | Apple 代理组 |
| EMBY | 仅筛选预期地区节点，并验证实际播放、拖动及手动切换 |
| g.cn、google.cn 的 HTTP/HTTPS 请求 | 重定向到 Google；HTTPS 需本地 MITM 证书受信任 |

通过请求日志核对命中的规则和实际出口；若某个辅助域名走错，补充精确例外。
不要把宽泛的 CDN、云厂商域名或整个 `.ai` 后缀统一指定为直连。
导入时检查 DNS 参数是否被识别，以及直连域名是否实际使用指定的 DoH。

配置自动更新地址跟随 GitHub `main`。工作区内的修改合并到 `main` 后，设备才能通过
该地址获取；此前可手动导入本地文件验证。出现回归时可从 Git 恢复旧配置并重新导入。

## 规则与配置参考

- [blackmatrix7 China](https://github.com/blackmatrix7/ios_rule_script/tree/master/rule/Shadowrocket/China)
- [ACL4SSR 分流顺序](https://github.com/ACL4SSR/ACL4SSR/blob/master/Clash/config/ACL4SSR_Online_Full.ini)
- [Loyalsoldier 白名单模式](https://github.com/Loyalsoldier/surge-rules#白名单模式推荐)
- [Johnshall Shadowrocket 配置参考](https://github.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever/blob/release/lazy_group.conf)
- [国内 DNS 与 CDN 选择](https://github.com/felixonmars/dnsmasq-china-list)
