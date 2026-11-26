# Claude Code + Codex CLI 离线安装包

Windows x64 离线安装包，附安装步骤与 API Key 更换说明。

## 目录内容

| 文件 | 大小 | 说明 |
|---|---|---|
| `claude-code-2.1.332-win32-x64.tgz` | 178 MB | Anthropic Claude Code CLI 2.1.332，Windows x64 |
| `codex-0.172.0-win32-x64.tgz` | 187 MB | OpenAI Codex CLI 0.172.0，Windows x64 |
| `Claude-Codex-CLI-Setup.zip` | 366 MB | 上述两个包 + 本说明的整体压缩包（若仓库中拆成 .001~.004 分卷，见文末合并方法） |
| `说明.txt` | — | 原有简要记录 |

两个 tgz 都是 npm 官方包（`npm pack` 产物），离线使用无需联网。

---

## 1. Claude Code 安装

### 前提
- Windows 10/11 x64
- Node.js 18+（`node -v` 能输出版本即可）

### 方式 A：从 tgz 离线安装（推荐，不依赖下载）

```powershell
npm install -g "D:\path\to\claude-code-2.1.332-win32-x64.tgz"
```

全局目录通常是 `%APPDATA%\npm`，装完确认该目录在 PATH 中：

```powershell
echo $env:PATH -split ';' | Select-String 'Roaming\npm'
```

验证：

```powershell
claude --version
# 2.1.332 (Claude Code)
```

### 方式 B：有网络时直接装
```powershell
npm install -g @anthropic-ai/claude-code
```

### 常见故障
- **`claude` 能执行但报“不是有效的应用程序”**：说明 `node_modules\@anthropic-ai\claude-code\bin\claude.exe` 只有几百字节（下载中断的占位文件）。用本包重装或手工把 tgz 解出的 `package/bin/claude.exe`（约 384 MB）覆盖过去即可。
- **报 “claude.ps1 无法加载”**：执行 `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` 后重开终端。
- **想不装 Node 直接用**：解压 tgz，把 `package/bin/claude.exe` 放到任意目录，把该目录加进 PATH 即可（此 exe 自包含）。

---

## 2. Codex CLI 安装

### 方式 A：从 tgz 离线安装

```powershell
npm install -g "D:\path\to\codex-0.172.0-win32-x64.tgz"
codex --version
# codex-cli 0.172.0
```

若全局命令装到了错误位置（例如被环境变量带到别的 node 目录），显式指定前缀：

```powershell
npm install -g --prefix "$env:APPDATA\npm" "D:\path\to\codex-0.172.0-win32-x64.tgz"
```

### 方式 B：有网络时
```powershell
npm install -g @openai/codex
```

### 常见故障
- 命令找不到：`%APPDATA%\npm` 未在 PATH；或用了别的 node 版本前缀，重装时加 `--prefix`。
- 中文目录/空格路径报错：把 tgz 复制到 `C:\temp` 再装。

---

## 3. 更换 API Key

### 3.1 Claude Code

Claude Code 的接入信息存在两处，优先级：**环境变量 > `~/.claude/settings.json` > 登录态**。

**配置文件位置**：`C:\Users\<你的用户名>\.claude\settings.json`

**换用第三方中转（如 DeepSeek 的 Anthropic 兼容接口）**：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的新key",
    "ANTHROPIC_MODEL": "deepseek-chat",
    "ANTHROPIC_SMALL_FAST_MODEL": "deepseek-chat",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-chat"
  },
  "theme": "dark"
}
```

改完**关掉终端重开**（配置只在启动时读取）。验证：

```powershell
claude
# 能正常对话即成功；若报 401 说明 key 或 base_url 有误
```

**换回 Anthropic 官方**：把 `settings.json` 里 `env` 中的 `ANTHROPIC_BASE_URL` 与 `ANTHROPIC_AUTH_TOKEN` 删掉，然后执行

```powershell
claude
/login
```

按提示在浏览器完成授权，凭证写入 `~/.claude/.credentials.json`。

**临时换 key（只对当前窗口生效）**：

```powershell
$env:ANTHROPIC_AUTH_TOKEN="sk-新key"
$env:ANTHROPIC_BASE_URL="https://api.anthropic.com"
claude
```

**注意**
- `ANTHROPIC_AUTH_TOKEN` 与 `ANTHROPIC_API_KEY` 二者取其一即可；官方直连更常用 `ANTHROPIC_API_KEY`。
- 想给不同项目用不同 key：在项目目录放 `.claude/settings.local.json`（字段同上，优先级高于用户级配置）。
- 换模型：改 `ANTHROPIC_MODEL`（主模型）与 `ANTHROPIC_SMALL_FAST_MODEL`（后台小任务）。

### 3.2 Codex CLI

**方式一：登录态（推荐）**

```powershell
codex login
```

浏览器授权后凭证保存在 `C:\Users\<你的用户名>\.codex\auth.json`。换账号就 `codex logout` 后重新 `login`。

**方式二：环境变量（API Key 直连）**

```powershell
setx OPENAI_API_KEY "sk-你的新key"
```

重开终端生效，然后 `codex` 直接可用。只对当前窗口生效则用 `$env:OPENAI_API_KEY="sk-..."`。

**方式三：接第三方兼容接口**

编辑 `C:\Users\<你的用户名>\.codex\config.toml`：

```toml
model = "gpt-5-codex"
model_provider = "custom"

[model_providers.custom]
name = "custom"
base_url = "https://你的中转地址/v1"
env_key = "CUSTOM_API_KEY"
wire_api = "responses"   # 部分中转只支持 chat，改成 "chat"
```

再设 `setx CUSTOM_API_KEY "sk-你的新key"` 并重开终端。

**方式四：改配置文件里的默认模型**
```toml
model = "gpt-5"
```
`codex --model <name>` 可临时覆盖。

**验证**
```powershell
codex --version
codex "hi"
```
返回模型回复即成功；401/403 是 key 问题，404 多半是 `base_url` 少了 `/v1`。

---

## 4. 配置文件速查

| 工具 | 配置文件 | 关键字段 |
|---|---|---|
| Claude Code | `%USERPROFILE%\.claude\settings.json` | `env.ANTHROPIC_BASE_URL`、`env.ANTHROPIC_AUTH_TOKEN`、`env.ANTHROPIC_MODEL` |
| Claude Code 项目级 | `<项目>\.claude\settings.local.json` | 同上 |
| Codex | `%USERPROFILE%\.codex\config.toml` | `model`、`model_provider`、`[model_providers.*]` |
| Codex 凭证 | `%USERPROFILE%\.codex\auth.json` | 由 `codex login` 写入，不要手改 |

---

## 5. 分卷压缩包的合并

若本仓库中的 `Claude-Codex-CLI-Setup.zip` 被拆成 `.001`、`.002`… 分卷（因为单文件超过 GitHub 100 MB 限制），下载全部分卷到同一目录后：

```powershell
# 方法一：Windows 自带 copy 合并
copy /b Claude-Codex-CLI-Setup.zip.001 + Claude-Codex-CLI-Setup.zip.002 + Claude-Codex-CLI-Setup.zip.003 + Claude-Codex-CLI-Setup.zip.004 Claude-Codex-CLI-Setup.zip

# 方法二：7-Zip
7z x Claude-Codex-CLI-Setup.zip.001
```

合并后校验 SHA-256 与仓库中 `SHA256SUMS.txt` 一致，再解压。

---

## 6. 校验

```powershell
Get-FileHash .\claude-code-2.1.332-win32-x64.tgz -Algorithm SHA256
Get-FileHash .\codex-0.172.0-win32-x64.tgz -Algorithm SHA256
```

与 `SHA256SUMS.txt` 对比一致即可放心安装。

## 备注

- 两个包均为官方 npm 发布物的原样副本，未做任何修改；使用须遵守 Anthropic / OpenAI 各自的许可条款。
- 本仓库不含任何真实 API Key，示例中的 `sk-你的新key` 均为占位符。
