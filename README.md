# WorkBuddy 每日积分签到（Windows 兼容独立版）

自动领取 WorkBuddy 每日积分（100 积分/天，连续第 7 天 1000 积分）。全流程在本机完成：读取本地登录态 → 调用腾讯官方签到接口，无后端服务。

> 本仓库是 [`cat-xierluo/legal-skills`](https://github.com/cat-xierluo/legal-skills) 中 `skills/workbuddy-checkin`（v1.0.2，MIT）的**独立维护分支**，额外合入 3 个 Windows 兼容性修复，使其在 Windows + PowerShell 5.1 上开箱可用。

## 相对上游的改动（本分支特有）

| # | 文件 | 修复内容 |
|---|------|----------|
| 1 | `scripts/decrypt-token.js` | 追加 `%LOCALAPPDATA%` 登录态探测路径（Windows 新版桌面端实际存储位置；上游仅探测 `%APPDATA%` 导致检索失败） |
| 2 | `scripts/checkin.ps1` | 文件加 UTF-8 BOM，修复 PowerShell 5.1 无 BOM 解析报错 |
| 3 | `scripts/checkin.ps1` | 改用 `curl -o` 落盘 + UTF-8 读取，修复 PS 5.1 把含中文响应的 UTF-8 字节按 ANSI(GBK) 解码，导致 `ConvertFrom-Json` 失败的 `PARSE_ERR` |

> 上游 `legal-skills` 的 Windows 路径与 `checkin.ps1` 均标注「仅推导未实测」，以上为本机真实踩坑验证修复。**macOS / Linux 用户建议直接用上游原版**（无需这些补丁）。

## 安装

将本仓库内容放置到 WorkBuddy 用户级技能目录：

```bash
# 克隆（把 qwgaan 换成实际 GitHub 用户名）
git clone https://github.com/qwgaan/workbuddy-checkin.git

# 复制到技能目录（仓库名恰为 workbuddy-checkin，直接复制文件夹即可）
cp -r workbuddy-checkin "$HOME/.workbuddy/skills/workbuddy-checkin"
```

Windows（PowerShell）：

```powershell
git clone https://github.com/qwgaan/workbuddy-checkin.git
Copy-Item -Recurse workbuddy-checkin "$env:USERPROFILE\.workbuddy\skills\workbuddy-checkin"
```

复制完成后**重启 WorkBuddy**，用自然语言「WorkBuddy 签到 / 每日积分 / check-in」即可唤起。

## 手动签到（验证）

```powershell
# Windows
powershell -ExecutionPolicy Bypass -File scripts\checkin.ps1

# macOS / Linux
bash scripts/checkin.sh
```

## 配置每日自动签到

详见 `SKILL.md` 的「在 WorkBuddy 内（Agent 自动化）」一节：用 `automation_update`（recurring 类型）创建定时任务即可。

本机实测配置参考：每天 09:00、工作目录指向一个专门文件夹、运行 `checkin.ps1`、结果追加到单一 Markdown 文件（不在每次运行新建任务列表）。例如：

```jsonc
{
  "name": "Buddy加油站每日自动签到",
  "scheduleType": "recurring",
  "rrule": "FREQ=DAILY;BYHOUR=9;BYMINUTE=0",
  "cwds": ["F:\\workbuddy\\定时任务\\workbuddy签到"],
  "status": "ACTIVE",
  "prompt": "运行 scripts/checkin.ps1 完成签到，读取 logs/checkin.log 末行判定结果，把当天一行追加进签到历史.md，可见回复只输出一句话。"
}
```

> 注意：`update` 已有任务时必须显式传 `rrule`，否则可能被重置。

## 安全

- 仅读取**本机当前登录用户自己**的 WorkBuddy 登录态；每个人读自己的 token，分享本技能不会泄露任何人的凭据。
- 网络访问仅发往腾讯官方接口 `copilot.tencent.com/billing/meter/*`，不上传任何第三方。
- 令牌仅在内存中经管道被签到请求消费，**不落盘、不回显、不提交到仓库**。详见 `SKILL.md`「安全说明」。

## 许可

MIT。基于 [`cat-xierluo/legal-skills`](https://github.com/cat-xierluo/legal-skills) 的 `workbuddy-checkin` v1.0.2，原作者保留版权；本分支改动以相同许可发布。
