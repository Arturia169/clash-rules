# Clash / Mihomo 个人远程分流规则集 (clash-rules)

本项目用于集中维护个人 Clash / Mihomo 客户端的手动分流规则，并通过 GitHub 远程引用与自动同步。

## 3.0 主配置模板

[`configs/configV 3.0.yaml`](configs/configV%203.0.yaml) 是经过规则审计的公开脱敏版，保留 DNS、TUN、46 个策略组、规则顺序及远程规则集引用。

- 真实订阅 URL、自建节点服务器地址、UUID、密码和 Reality 参数已替换为占位值；个人资产直连规则及对应 DNS 过滤条目已移除。
- 使用前复制为 `configs/configV 3.0.local.yaml`，填写自己的订阅与自建节点参数。示例节点和订阅不能直接用于联网；填写后按需添加私人直连规则。
- `*.local.yaml` 已加入忽略规则，不应把填写了真实凭据的版本提交到公开仓库。原有本地完整配置不受此模板影响。
- 模板通过 Mihomo v1.19.32 配置语法校验。自定义规则更新后，客户端仍需更新规则集；CDN 缓存可能延迟反映 GitHub 的最新提交。

## 规则文件清单与策略映射

| 文件名 | 用途描述 | 建议绑定策略组 | CDN 加速直链 |
| :--- | :--- | :--- | :--- |
| [`rules/custom_proxy.list`](rules/custom_proxy.list) | 自用开发、搜索与 MCP 工具（Tavily, Firecrawl, Cline 等） | `🚀 默认代理` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_proxy.list` |
| [`rules/custom_ai.list`](rules/custom_ai.list) | 自用 AI 镜像与 API 分流补充 | `🤖 AI` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_ai.list` |
| [`rules/custom_crypto.list`](rules/custom_crypto.list) | 主流加密货币交易所补充（币安、OKX、Bybit、Gate 等） | `💰 加密交易所` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_crypto.list` |
| [`rules/custom_web3.list`](rules/custom_web3.list) | Web3 钱包、链上 RPC、区块链浏览器、DeFi 与跨链服务 | `🌐 Web3链上` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_web3.list` |
| [`rules/custom_speedtest.list`](rules/custom_speedtest.list) | 节点测速平台 | `🚀 节点测速` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_speedtest.list` |
| [`rules/custom_telegram.list`](rules/custom_telegram.list) | Telegram 第三方客户端（Nicegram）补充 | `📲 Telegram` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_telegram.list` |
| [`rules/custom_google.list`](rules/custom_google.list) | Google Play 与国区特殊加速域名 | `🍀 Google` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_google.list` |

## 语法规范说明

- 采用 **`classical` (传统混合) 格式**，直接支持：
  - `DOMAIN-SUFFIX,domain.com`（匹配后缀）
  - `DOMAIN,example.com`（全匹配）
  - `DOMAIN-KEYWORD,keyword`（关键字匹配）
  - `IP-CIDR,1.2.3.4/32,no-resolve`（IP 网段匹配）
  - `DST-PORT,443`（目标端口匹配）
- **注意**：规则文件中**不要**在末尾附带策略组名称，策略组是在主配置文件的 `rules:` 中的 `RULE-SET` 处绑定的。

## 客户端配置示例

```yaml
rule-anchor:
  class: &class {type: http, interval: 86400, behavior: classical, format: text}

rule-providers:
  custom_proxy: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_proxy.list", path: ./rules/custom_proxy.list }
  custom_ai: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_ai.list", path: ./rules/custom_ai.list }
  custom_crypto: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_crypto.list", path: ./rules/custom_crypto.list }
  custom_web3: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_web3.list", path: ./rules/custom_web3.list }
  custom_speedtest: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_speedtest.list", path: ./rules/custom_speedtest.list }
  custom_telegram: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_telegram.list", path: ./rules/custom_telegram.list }
  custom_google: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_google.list", path: ./rules/custom_google.list }

rules:
  - RULE-SET,custom_ai,🤖 AI
  - RULE-SET,custom_proxy,🚀 默认代理
  - RULE-SET,custom_crypto,💰 加密交易所
  - RULE-SET,custom_web3,🌐 Web3链上
  - RULE-SET,custom_telegram,📲 Telegram
  - RULE-SET,custom_google,🍀 Google
  - RULE-SET,custom_speedtest,🚀 节点测速
```

## Web3 规则维护与生效

- `custom_web3.list` 按钱包、RPC、浏览器、DeFi、跨链、行情和资讯分类维护。
- 使用 `DOMAIN-SUFFIX` 覆盖服务域名及其子域名；已覆盖的子域名不重复添加。
- 官网与 API/RPC 可能使用不同后缀，例如 `quicknode.com` 与 `quiknode.pro`、`1inch.io` 与 `1inch.com`，需要分别收录。
- 主配置中，`custom_crypto`、`custom_web3` 应优先于 Telegram 公共域名分类、公共 `category-cryptocurrency` 和 Google 等服务 IP 规则。这样 TON、DeBank、Aave、dRPC 等服务会先进入指定策略组；公共加密货币分类包含公链、钱包和资讯，并非仅有交易所。
- 补充实际服务域名：Phantom 官网与开发文档使用 `phantom.com`；WalletConnect 还使用 `walletconnect.network`，连接组件包含 `web3modal.org`；链元数据使用 `chainid.network`，DefiLlama API 使用 `llama.fi`。Ethereum、BNB Chain、Base、TON 与 TON Center 按 Web3 分类维护。
- 本地修改需同步至 GitHub 后，使用 HTTP 规则集的客户端才会获取更新；可在客户端手动更新规则集，或等待配置中的更新周期。

补充域名依据：[Phantom 官方文档](https://docs.phantom.com/introduction)、[WalletConnect 官方 SDK 页面](https://walletconnect.network/sdk)、[公共加密货币域名清单](https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/category-cryptocurrency.list)、[链元数据仓库](https://github.com/ethereum-lists/chains)、[Base 官方说明](https://www.base.org/blog/base-mainnet-is-open-for-builders)、[TON Center 官方文档](https://docs.ton.org/ecosystem/api/toncenter/v3/overview)、[DefiLlama API 文档](https://github.com/DefiLlama/api-docs/blob/main/llms-free.txt)。

## 其他自定义规则维护

- 测速：使用 `DOMAIN-KEYWORD,speedtest` 与 `DOMAIN-KEYWORD,ookla` 补充运营商自建测速主机，例如 `losangeles.ca.speedtest.frontier.com:8080`；规则匹配域名，不限制端口。关键词统一维护在本规则集，主配置通过 `custom_speedtest` 引用。无关键词的第三方测速域名仍需按实际连接补充，不使用通用 `8080` 端口规则。`measurementlab.net` 覆盖 M-Lab 网站与服务发现，`measurement-lab.org` 覆盖实际测速服务器。FAST 从自定义测速名单中移除，在 3.0 配置中由 Netflix 公共规则统一处理页面和 CDN 流量；`www.gstatic.com` 由 Google 规则接管。
- Telegram：自定义规则仅补充 Nicegram，官方域名与 IPv4/IPv6 网段统一交由主配置的公共 `telegram_domain` / `telegram_ip` 维护，不再手动重复收录。Nicegram 使用 `nicegram.app` 后缀，避免关键词匹配到无关域名；TON 区块链域名由优先级更高的 `custom_web3` 接管。
- 分流优先级：`custom_proxy` 等个人明确规则应排在对应公共大类之前，例如 `typesafe.ai` 在公共 AI 分类中，但本配置明确要求它走默认代理。游戏内容下载域名直连规则应优先于 Microsoft 等平台大类；服务 IP 规则用于专用域名规则之后的兜底。
- 交易所：同时保留 `gate.io` 和 `gate.com`，覆盖原有域名与当前官方域名。
- AI、开发工具与 Google 补充规则沿用原有配置，不重复添加公共规则已覆盖的官方 AI 域名。
