---
title: 将 Cloudflare Zero Trust 提供的 WARP+ 用于代理客户端
date: 2026-04-06 21:56:16
updated: 2026-04-06 21:56:16
tags: cloudflare 
---
## 准备
一个 Cloudflare 账号
## 步骤
1. 安装依赖：`jq` , `curl` , `wireguard-tools`
2. 拉取项目[rany2/warp.sh](https://github.com/rany2/warp.sh)的储存库
```bash
git clone https://github.com/rany2/warp.sh.git
cd warp.sh
```
3. 在浏览器中打开 `https://<你的组织名>.cloudflareaccess.com/warp` 登录账号，在浏览器开发者工具中获取页面中按钮 (\<bottom\>) 里的 token 值
4. 运行
```bash
./warp.sh -T <token>
```
3, 4 两步要快，否则你会得到 401[^1]

5. 从输出中提取我们需要的内容
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

## 参考资料
https://github.com/rany2/warp.sh
https://wiki.metacubex.one/config/proxies/wg/#wireguard_1
[^1]: https://github.com/rany2/warp.sh/issues/14