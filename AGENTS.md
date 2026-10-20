# AGENTS.md

个人 Clash / Mihomo 远程分流规则集。客户端通过 `rule-providers` 远程引用 `rules/*.list`。

> **这是纯数据仓库，不是代码项目。** 没有构建、没有测试、没有依赖、没有 CI。
> 改动的全部意义就是：让客户端的 `RULE-SET` 匹配到正确的域名/IP。

## 仓库结构

```
rules/custom_*.list   # 规则数据，每个文件对应一个策略组
README.md             # 文件清单表 + 客户端配置示例（改规则文件时必须同步）
```

现有 7 个文件及其绑定策略组：

| 文件 | 策略组 |
| :--- | :--- |
| `custom_proxy.list` | 🚀 默认代理 |
| `custom_chatgpt.list` | 🤖 ChatGPT |
| `custom_telegram.list` | 📲 Telegram |
| `custom_google.list` | 🍀 Google |
| `custom_speedtest.list` | 🚀 节点测速 |
| `custom_crypto.list` | 💰 加密交易所 |
| `custom_web3.list` | 🌐 Web3链上 |

## 硬性规则（动手前必读）

1. **每行只能是规则本身，绝不能在行尾写策略组名。**
   错误：`DOMAIN-SUFFIX,binance.com,💰 加密交易所`
   正确：`DOMAIN-SUFFIX,binance.com`
   策略组是在客户端主配置的 `RULE-SET` 处绑定的，写进 `.list` 里会导致解析失败。

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

## 新增一个规则文件

必须同时改四处，缺一不可：

1. 新建 `rules/custom_<name>.list`，带两行注释头
2. README 的「规则文件清单与策略映射」表格加一行（含 jsdelivr 直链）
3. README 的 `rule-providers` 示例加一条（沿用 `<<: *class` 锚点）
4. README 的 `rules:` 列表加一行 `RULE-SET,<name>,<策略组>`

**顺序即优先级**：`rules:` 里越靠前越先匹配。放新文件时想清楚它该在哪个位置，不要默认塞到末尾。

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

四条都无输出才算干净。

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
chore: 更新 README 说明
```
