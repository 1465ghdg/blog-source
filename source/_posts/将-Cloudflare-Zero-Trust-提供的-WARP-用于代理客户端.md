---
title: 将 Cloudflare Zero Trust 提供的 WARP+ 用于代理客户端
date: 2026-04-06 21:56:16
updated: 2026-05-05 21:54:55
tags: cloudflare
---
## 前言
我们知道，cloudflare 提供免费的匿名代理 WARP，但它在国内无法使用

如果你用过 ZeroTrust 就会注意到它可以正常连接  
// 它用的甚至是 WARP+

但不管是哪个，为它们配置分流都是个大问题

如果将其以 Wireguard 节点的方式用在 mihomo/singbox 之类的东西中，就可以完美解决这个问题  
// 当然通过代理链的方式用 WARP 来落地也行...咳咳扯远了，让我们回到正题
## 开始前的准备
显然，我们需要一个 Cloudflare 账号（
## 步骤
首先，安装依赖：`jq` ，`curl` ，`wireguard-tools`

然后拉取项目[rany2/warp.sh](https://github.com/rany2/warp.sh)的储存库
```bash
git clone https://github.com/rany2/warp.sh.git
cd warp.sh
```
在浏览器中打开 `https://<你的组织名>.cloudflareaccess.com/warp` 登录账号，在浏览器开发者工具中获取页面中按钮 (\<bottom\>) 里的 token 值

接着运行
```bash
./warp.sh -T <token>
# 如果你懒得去 shell 历史记录里删可以把 token 存进文件，然后把命令里放 token 的地方换成文件路径就行
```
token 有效期只有 1min 左右，所以要快，否则你会得到 401[^1]

从输出中提取我们需要的内容
```conf
[Interface]
Address = <本机组网IP>
PrivateKey = <本机私钥>
MTU = <预设MTU>

[Peer]
AllowedIPs = <转发IP段>
Endpoint = <远端地址>:<远端端口>
PublicKey = <远端公钥>
```
```yaml
- name: "wg"
  type: wireguard
  ip: <本机组网 IPv4>
  ipv6: <本机组网 IPv6>
  private-key: <本机私钥>
  server: <远端地址>
  port: <远端端口>
  public-key: <远端公钥>
  udp: true
  mtu: <预设 MTU>
```
大功告成，现在享受国际超级无敌大厂的节点吧（

对了...记得每隔大约 1 天用
```bash
./warp.sh -R <refresh_token>
# refresh_token 在前面输出的第 9 行
```
刷新，配置不会变，但就是要刷新（

// 虽然但是，这玩意儿延迟真高啊（
## MASQUE
如果想使用 `MASQUE` 连接 WARP, 可以使用 [usque](https://github.com/Diniboy1123/usque/) 生成配置
```bash
usque register --jwt <token>
```
然后从工作目录下其生成的 `config.json` 中提取内容
```json
{
  "private_key": "<private_key>",
  "endpoint_v4": "<endpoint_v4>",
  "endpoint_v6": "<endpoint_v6>",
  "endpoint_h2_v4": "<endpoint_h2_v4>",
  "endpoint_h2_v6": "",
  "endpoint_pub_key": "-----BEGIN PUBLIC KEY-----\nMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEIaU7MToJm9NKp8YfGxR6r+/h4mcG\n7SxI8tsW8OR1A5tv/zCzVbCRRh2t87/kxnP6lAy0lkr7qYwu+ox+k3dr6w==\n-----END PUBLIC KEY-----\n",
  "license": "<license>",
  "id": "<id>",
  "access_token": "<access_token>",
  "ipv4": "<ipv4>",
  "ipv6": "<ipv6>"
}
```
```yaml
- name: "masque"
  type: masque
  server: <endpoint>
  # network: h2 # 如果你需要使用 h2 连接的话
  port: 443
  public-key: MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEIaU7MToJm9NKp8YfGxR6r+/h4mcG7SxI8tsW8OR1A5tv/zCzVbCRRh2t87/kxnP6lAy0lkr7qYwu+ox+k3dr6w==
  private-key: <private_key>
  ip: <ipv4>
  ipv6: <ipv6>
  # udp: true # 如果你需要允许 udp 通过代理的话
  # congestion-controller: bbr # 如果你需要改用 bbr 拥塞控制的话
```
## 更改 ip 属地
通过将 endpoint 改为 `162.159.198.2` 或 `162.159.199.2`, 似乎可以让 WARP 为你分配位于美国的出口 ip

据说使用了 NAT64 , 可能涉及滥用服务[^2]...?

等等...上文提到的 endpoint 似乎就是 cloudflare 的...好吧我也不知道 WARP 为什么会这么做但...用就完了（
## 参考资料
https://github.com/rany2/warp.sh
https://github.com/Diniboy1123/usque
https://wiki.metacubex.one/config/proxies/wg/
https://wiki.metacubex.one/config/proxies/masque/
https://github.com/Diniboy1123/usque/issues/72

[^1]: https://github.com/rany2/warp.sh/issues/14
[^2]: https://github.com/Diniboy1123/usque/discussions/74#discussioncomment-16024482