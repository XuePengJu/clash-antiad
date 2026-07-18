# Clash 配置说明（clash_adblock.yaml）

一个用于**屏蔽广告/追踪域名**（直接断开，不联网）的 Clash 规则配置。
所有正常流量走直连（`DIRECT`），无需代理节点即可使用。

## 功能

- **广告拦截**：通过 `rule-providers` 引入 [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules) 的 `reject.txt` 规则集，对命中的广告、隐私追踪、恶意网站域名执行 `REJECT`（直接断开连接，表现为广告位加载失败/空白）。每天自动更新一次。
- **DNS 增强**：`enhanced-mode: fake-ip` + 本地 DNS 监听，提升解析效率并配合规则匹配。
- **全部直连**：除被拦截的广告外，其余流量一律 `DIRECT`。

## 配置要点

| 配置项 | 取值 | 说明 |
| --- | --- | --- |
| `mode` | `rule` | 按规则分流 |
| `mixed-port` | `7897` | HTTP/SOCKS 混合代理端口 |
| `dns.listen` | `127.0.0.1:5353` | 本地 DNS 监听地址（**需把系统 DNS 指向它**） |
| `dns.enhanced-mode` | `fake-ip` | 假 IP 模式 |
| `rule-providers.reject` | `reject.txt` | 广告/追踪拦截规则集，缓存到 `./ruleset/reject.yaml` |
| `rules` | `RULE-SET,reject,REJECT` → `MATCH,DIRECT` | 先拦截广告，其余直连 |

> 说明：使用 `5353` 而非默认的 `53`，是为了避开 macOS 对 <1024 特权端口的权限限制，并避免与本地其它 DNS 服务（如 mDNSResponder 占用的 5353 之外的冲突）抢端口。

## 使用方法

1. 把本目录下的 `clash_adblock.yaml` 作为 Clash 核心的配置文件启动。
2. **设置系统 DNS 为 `127.0.0.1:5353`**（重要，否则本地 DNS 监听器不会被访问，广告拦截与 fake-ip 均不生效）：
   - macOS：系统设置 → 网络 → 当前连接 → DNS → 添加 `127.0.0.1`。
   - 也可在 Clash 客户端中开启「系统代理 / TUN 模式」让其自动接管 DNS。
3. 首次启动 Clash 会自动联网下载 `reject` 规则集并缓存到 `./ruleset/reject.yaml`（请确保 `ruleset/` 目录可写）。

## 目录结构

```
clash_conf/
├── clash_adblock.yaml   # 主配置文件（广告屏蔽）
├── ruleset/             # 规则集缓存目录（reject.yaml 自动生成）
└── README.md
```

## 自定义

- **补充自己的广告黑名单**：在 `rules:` 的 `RULE-SET,reject,REJECT` 之前，手动添加，例如：
  ```yaml
  rules:
    - DOMAIN-SUFFIX,ads.example.com,REJECT
    - DOMAIN-KEYWORD,tracking,REJECT
    - RULE-SET,reject,REJECT
    - MATCH,DIRECT
  ```
- **更换/精简规则集**：把 `rule-providers.reject.url` 换成你信任的其它规则集（如仅国内广告、或自建列表）即可。
- **使用代理节点**：当前只有 `direct` 节点。如需走代理，在 `proxies` 添加节点并在 `proxy-groups` / `rules` 中引用即可。
