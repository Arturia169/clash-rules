# Clash / Mihomo 个人远程分流规则集 (clash-rules)

本项目用于集中维护个人 Clash / Mihomo 客户端的手动分流规则，并通过 GitHub 远程引用与自动同步。

## 规则文件清单与策略映射

| 文件名 | 用途描述 | 建议绑定策略组 | CDN 加速直链 |
| :--- | :--- | :--- | :--- |
| [`rules/custom_proxy.list`](rules/custom_proxy.list) | 自用开发、搜索与 MCP 工具（Tavily, Firecrawl, Cline 等） | `🚀 默认代理` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_proxy.list` |
| [`rules/custom_chatgpt.list`](rules/custom_chatgpt.list) | 自用 AI 镜像与 API 分流补充 | `🤖 ChatGPT` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_chatgpt.list` |
| [`rules/custom_crypto.list`](rules/custom_crypto.list) | 主流加密货币交易所补充（币安、OKX、Bybit、Gate 等） | `💰 加密交易所` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_crypto.list` |
| [`rules/custom_web3.list`](rules/custom_web3.list) | Web3 钱包、链上 RPC、区块链浏览器、DeFi | `🌐 Web3链上` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_web3.list` |
| [`rules/custom_speedtest.list`](rules/custom_speedtest.list) | 节点测速平台 | `🚀 节点测速` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_speedtest.list` |
| [`rules/custom_telegram.list`](rules/custom_telegram.list) | Telegram 常用域名与官方 IP 网段 | `📲 Telegram` | `https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_telegram.list` |
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
  custom_chatgpt: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_chatgpt.list", path: ./rules/custom_chatgpt.list }
  custom_crypto: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_crypto.list", path: ./rules/custom_crypto.list }
  custom_web3: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_web3.list", path: ./rules/custom_web3.list }
  custom_speedtest: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_speedtest.list", path: ./rules/custom_speedtest.list }
  custom_telegram: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_telegram.list", path: ./rules/custom_telegram.list }
  custom_google: { <<: *class, url: "https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/custom_google.list", path: ./rules/custom_google.list }

rules:
  - RULE-SET,custom_chatgpt,🤖 ChatGPT
  - RULE-SET,custom_proxy,🚀 默认代理
  - RULE-SET,custom_telegram,📲 Telegram
  - RULE-SET,custom_google,🍀 Google
  - RULE-SET,custom_speedtest,🚀 节点测速
  - RULE-SET,custom_crypto,💰 加密交易所
  - RULE-SET,custom_web3,🌐 Web3链上
```
