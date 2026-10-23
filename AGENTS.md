# AGENTS.md

个人 Clash / Mihomo 远程分流规则集，外加一份公开脱敏的主配置模板。客户端通过 `rule-providers` 远程引用 `rules/*.list`。

> **这是数据仓库，不是代码项目。** 没有构建、没有测试、没有依赖、没有 CI。
> 改动只有两个意义：让客户端的 `RULE-SET` 匹配到正确的域名/IP；让 `configs/` 里的模板仍能被 mihomo 直接加载。

## 仓库结构

```
rules/custom_*.list        # 规则数据，每个文件对应一个策略组
configs/configV 3.0.yaml   # 主配置公开脱敏模板（DNS / TUN / 策略组 / 规则顺序 / 远程规则集引用）
README.md                  # 清单表 + 配置示例 + 各类规则维护说明（改规则文件时必须同步）
.gitignore                 # 忽略 configs/*.local.yaml
```

现有 7 个规则文件：

| 文件 | 管什么 | 策略组 |
| :--- | :--- | :--- |
| `custom_proxy.list` | 自用开发、搜索、MCP 工具链（Tavily / Firecrawl / Context7 等） | 🚀 默认代理 |
| `custom_ai.list` | 自用 AI 镜像与 API 补充 | 🤖 AI |
| `custom_telegram.list` | **仅**第三方客户端补充（Nicegram） | 📲 Telegram |
| `custom_google.list` | Google Play 与国区特殊加速域名 | 🍀 Google |
| `custom_speedtest.list` | 节点测速平台 + 运营商自建测速主机关键词 | 🚀 节点测速 |
| `custom_crypto.list` | 主流交易所补充（币安 / OKX / Bybit / Gate 等） | 💰 加密交易所 |
| `custom_web3.list` | Web3 钱包、链上 RPC、浏览器、DeFi、跨链 | 🌐 Web3链上 |

## ⛔ 安全红线：`configs/` 是公开仓库

`configs/configV 3.0.yaml` 是**脱敏模板**，全世界都能看到它。

1. **绝不**把真实订阅 URL、节点服务器地址、UUID、密码、Reality `public-key` / `short-id`、个人资产直连规则写进这个文件。
2. 占位值必须保持原样：IP 用 `192.0.2.x`（RFC 5737 文档保留段）、UUID 用 `00000000-0000-4000-8000-000000000001`、`public-key` 用全 `A`。
3. 真实配置放 `configs/*.local.yaml`，已被 `.gitignore` 忽略。**永远不要** `git add -f` 它。
4. 改完模板必须自查。第一条列出全部凭据行 —— 只允许全 0 的 uuid / 全 `A` 的 public-key / 全 0 的 short-id / 含 `REPLACE` 的 password；第二条**必须零输出**：

```bash
# A. 凭据行：逐行确认是占位值
grep -nE '^\s*(uuid|password|public-key|short-id):' "configs/configV 3.0.yaml"

# B. 订阅与节点地址：只允许 example.com 或 192.0.2.x（应无输出）
grep -nE '^\s*(url|server):' "configs/configV 3.0.yaml" \
  | grep -vE '192\.0\.2\.|example\.com|gstatic\.com|generate_204'
```

> 不要用 `grep -i reality` 这类宽匹配查脱敏 —— 策略组名字里就带 `Reality` / `HY2`，会刷出几十行噪音，把真正的凭据行淹掉。

## 硬性规则（动手前必读）

1. **每行只能是规则本身，绝不能在行尾写策略组名。**
   错误：`DOMAIN-SUFFIX,binance.com,💰 加密交易所`
   正确：`DOMAIN-SUFFIX,binance.com`
   策略组是在主配置的 `RULE-SET` 处绑定的，写进 `.list` 里会导致解析失败。

2. **文件头两行注释固定格式，修改内容时保留，不要删改：**
   ```
   # <这个文件管什么>
   # 匹配策略组: <组名>
   ```
   第二行的组名必须与 README 表格一致。

3. **只用以下规则类型**（`behavior: classical`）：
   `DOMAIN` / `DOMAIN-SUFFIX` / `DOMAIN-KEYWORD` / `IP-CIDR` / `IP-CIDR6` / `DST-PORT`
   要引入 `PROCESS-NAME`、`GEOSITE`、`RULE-SET` 之类的新类型前先确认目标 mihomo 版本支持，并同步更新 README 的语法说明。

4. **域名写法**：只写裸域名，不带协议、不带路径、不带 `*.` 通配符。
   错误：`DOMAIN-SUFFIX,https://binance.com/api` / `DOMAIN-SUFFIX,*.binance.com`
   正确：`DOMAIN-SUFFIX,binance.com`

5. **`IP-CIDR` 一律加 `,no-resolve`**，除非确实需要对域名做 DNS 解析后再匹配（会拖慢请求）。IPv6 用 `IP-CIDR6`。

6. **在文件末尾追加，不要重排既有行。** 现有文件并非严格字母序，重排会产生整文件 diff 噪音，让后续 review 和回滚变难。

7. **一个域名只出现在一个文件里。** 重复会导致匹配顺序依赖，难以排查。新增前先 `grep -r` 全仓确认。

8. **不写注释掉的历史规则**。删就删干净，需要留痕用 git history。

9. **不提交 `configs/*.local.yaml`**，哪怕只是临时调试。真实凭据一旦进 git history，删文件也抹不掉。

## 现行分工（别把已经交给公共规则的域名加回来）

这些是踩过的重复，动手前先确认：

- **Telegram**：本仓库只补第三方客户端（`nicegram.app`）。官方域名与 IPv4/IPv6 网段由主配置的公共 `telegram_domain` / `telegram_ip` 维护，**不要手动加回**。TON 等链上域名由优先级更高的 `custom_web3` 接管。
- **测速**：用 `DOMAIN-KEYWORD,speedtest` 和 `DOMAIN-KEYWORD,ookla` 覆盖运营商自建主机（规则匹配域名，不限制端口）。`fast.com` 已移除，交给 Netflix 公共规则；`www.gstatic.com` 交给 Google 规则。
- **AI / 开发工具**：不重复添加公共 AI 规则已覆盖的官方域名。
- **交易所**：`gate.io` 和 `gate.com` 都要留，覆盖新旧官方域名。
- **Web3**：官网和 API/RPC 常是不同后缀（`quicknode.com` / `quiknode.pro`、`1inch.io` / `1inch.com`），需要分别收录；已覆盖的子域名不重复加。

**优先级**：`rules:` 里越靠前越先匹配。`custom_crypto` / `custom_web3` 必须排在 Telegram 公共分类、`category-cryptocurrency`、Google 等服务 IP 规则**之前**；个人明确规则（如 `typesafe.ai` 走默认代理）排在对应公共大类之前。

## 新增一个规则文件

必须同时改四处，缺一不可：

1. 新建 `rules/custom_<name>.list`，带两行注释头
2. README 的「规则文件清单与策略映射」表格加一行（含 jsdelivr 直链）
3. README 的 `rule-providers` 示例加一条（沿用 `<<: *class` 锚点）
4. README 的 `rules:` 列表加一行 `RULE-SET,<name>,<策略组>`

如果新文件要在主配置模板里生效，还要在 `configs/configV 3.0.yaml` 的 `rule-providers` 和 `rules:` 里同步加，位置按上面的优先级规矩定。

## 提交前自检

```bash
# 1a. 字段数：普通规则必须恰好 2 段；IP-CIDR 允许第 3 段且必须是 no-resolve
awk -F, '!/^#/ && NF>0 {
  if ($1=="IP-CIDR" || $1=="IP-CIDR6") {
    if (NF!=2 && !(NF==3 && $3=="no-resolve")) print FILENAME":"FNR": "$0
  } else if ($1=="DOMAIN" || $1=="DOMAIN-SUFFIX" || $1=="DOMAIN-KEYWORD" || $1=="DST-PORT") {
    if (NF!=2) print FILENAME":"FNR": "$0
  } else print FILENAME":"FNR": [未知规则类型] "$0
}' rules/*.list

# 1b. 域名不能带通配符 / 协议 / 路径
awk -F, '/^(DOMAIN|DOMAIN-SUFFIX|DOMAIN-KEYWORD),/ && ($2 ~ /[*\/:]/) {print FILENAME":"FNR": "$0}' rules/*.list

# 2. 查重复域名
grep -rhE '^(DOMAIN|DOMAIN-SUFFIX|DOMAIN-KEYWORD),' rules/ | awk -F, '{print $2}' | sort | uniq -d

# 3. IP-CIDR 必须带 no-resolve（用 awk，grep 正则在这里会误报）
awk -F, '/^IP-CIDR6?,/ && $3 != "no-resolve" {print FILENAME":"FNR": "$0}' rules/*.list
```

四条都无输出才算干净。**不要用 `grep -v '^前缀'` 做这件事** —— 它只看行首，`DOMAIN-SUFFIX,evil.com,💰 加密交易所` 这种最典型的错误会直接漏过去。

## 发布与生效

- 引用走 jsdelivr：`https://testingcf.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/<file>.list`
- **推到 `main` 即生效，没有预览和灰度。** 不确定的规则先本地测。
- jsdelivr 有边缘缓存，推完立即拉可能还是旧内容。需要立刻生效时清缓存：
  `https://purge.jsdelivr.net/gh/Arturia169/clash-rules@main/rules/<file>.list`
- 备用直链（绕过 jsdelivr）：`https://raw.githubusercontent.com/Arturia169/clash-rules/main/rules/<file>.list`
- 客户端 `interval: 86400`，改完要在路由器上重载 OpenClash 配置才会重新拉取。

## 提交信息

沿用现有风格（Conventional Commits）：

```
feat: 新增 <域名> 到 custom_<组>
fix: 修正 custom_<组> 中错误的匹配类型
docs: 更新 README / AGENTS 说明
```
