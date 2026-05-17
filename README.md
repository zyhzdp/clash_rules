# ShellCrash Clash Meta Rules

这是一套面向小米路由器 ShellCrash 的 Clash Meta 规则工程骨架，目标是长期维护：

- 国内与局域网直连
- AI 服务走独立策略组
- Google / GitHub 独立策略组
- OpenAI / ChatGPT 独立细分
- 拦截 UDP/443，禁用 QUIC，减少 Google/YouTube 走 QUIC 时的异常
- 公开仓库不保存订阅地址、节点、密码、token 等私密信息

## 目录结构

```text
.
├── config.yaml                  # ShellCrash/Clash Meta 主配置，不含私密节点/密钥
├── providers/
│   └── rules/
│       ├── ai.yaml              # AI 通用规则
│       ├── china-direct.yaml    # 国内直连补充规则
│       ├── github.yaml          # GitHub 规则
│       ├── google.yaml          # Google 规则
│       ├── lan.yaml             # 局域网/内网直连
│       ├── openai.yaml          # OpenAI/ChatGPT 规则
│       └── reject-quic.yaml     # UDP/443 QUIC 拦截规则
├── templates/
│   └── private-providers.example.yaml
├── ruleset/                     # ShellCrash 运行时下载缓存，已忽略
├── ui/                          # zashboard 运行时下载目录，已忽略
└── private/                     # 本地私密目录，已忽略
```

## 使用方法

1. 将 `templates/private-providers.example.yaml` 复制到 `private/proxy-providers.yaml`。
2. 在 `private/proxy-providers.yaml` 中填入你的机场订阅 URL 或本地节点文件。
3. 将仓库推送到 GitHub。
4. 在 ShellCrash 中加载 `config.yaml`。
5. `rule-providers` 使用 `https://raw.githubusercontent.com/zyhzdp/clash_rules/main/...` 远程地址；`path` 只是 ShellCrash 本地缓存路径。

## 私密配置

不要把以下内容提交到 public GitHub：

- 机场订阅 URL
- 代理节点
- API key / token
- Clash Dashboard secret
- 真实家庭公网 IP、DDNS、内网服务密码

本仓库用 `private/` 存这些内容，并已在 `.gitignore` 中忽略。

## 策略设计

默认链路：

1. 局域网、国内站点、Loyalsoldier direct、GeoIP CN 直连
2. UDP/443 直接拒绝，用于禁用 QUIC
3. OpenAI 命中 `OpenAI`
4. 其他 AI 命中 `AI`
5. Google 命中 `Google`
6. GitHub 命中 `GitHub`
7. Loyalsoldier gfw/proxy 命中 `Proxy`
8. 漏网流量进入 `Final`

## 合并自设备侧配置的设置

当前模板已合入透明代理常用端口、TUN、路由标记、zashboard 下载地址、IPv6 DNS、`rule-set:cn` DNS 分流、以及 Loyalsoldier `direct` / `proxy` / `gfw` 远程规则源。

为了适合 public GitHub，模板保留了两个保守调整：

- `external-controller` 默认监听 `127.0.0.1:9999`，避免无密钥暴露到局域网。
- 未加入 `fake-ip-filter: "*"`，因为这会让几乎所有域名绕过 fake-ip，削弱透明代理效果。

`OpenAI`、`AI`、`Google`、`GitHub` 都可以在运行时手动切换到 `Proxy`、`DIRECT` 或 `REJECT`。
