# iris 翻译 CLI —— 下载与使用

> 本仓库**只承载 iris 翻译 CLI 的各平台二进制**（源码不在此）。这是一个单文件命令行工具，调用公司 iris 翻译服务完成多语种翻译，支持传入聊天上下文帮助模型理解语气/指代。一条命令拿译文，无需写代码。
>
> 本文档也供 **Codex / Claude Code 等 AI 助手**阅读并代为操作：按 ① 判断平台 → ② 安装 → ③ 配置 api-key → ④ 调用翻译 的顺序执行。

## 下载

前往 **[Releases](../../releases/latest)** 下载最新版对应平台的包，或直接看下表。当前版本 **v0.2.0**。

---

## ① 选择对应平台的包

| 用户环境 | 包文件 |
|---|---|
| Mac（Apple 芯片 M1/M2/M3…）| `iris_v0.2.0_darwin_arm64.tar.gz` |
| Mac（Intel 芯片）| `iris_v0.2.0_darwin_amd64.tar.gz` |
| Linux（x86_64）| `iris_v0.2.0_linux_amd64.tar.gz` |
| Linux（ARM64 / aarch64）| `iris_v0.2.0_linux_arm64.tar.gz` |
| Windows（64 位）| `iris_v0.2.0_windows_amd64.zip` |

**判断命令**（Mac/Linux）：`uname -sm`
- `Darwin arm64` → Apple 芯片 Mac（darwin_arm64）
- `Darwin x86_64` → Intel Mac（darwin_amd64）
- `Linux x86_64` → linux_amd64；`Linux aarch64` → linux_arm64

Windows 一般选 `windows_amd64`。

---

## ② 安装

### Mac / Linux
```bash
# 示例：Apple 芯片 Mac（其他平台把包名换成上表对应项）
cd <包所在目录>                       # 例如 ~/Downloads
tar xzf iris_v0.2.0_darwin_arm64.tar.gz    # 解压得到可执行文件 iris
mkdir -p ~/.local/bin && mv iris ~/.local/bin/
export PATH="$HOME/.local/bin:$PATH"       # 建议追加进 ~/.zshrc 或 ~/.bashrc
iris version                               # 验证：应输出 iris v0.2.0
```

**Mac 首次运行被拦截**（未签名 App）时任选其一：
```bash
xattr -d com.apple.quarantine ~/.local/bin/iris
```
或：系统设置 → 隐私与安全性 → 「仍要打开」。

### Windows
解压 `iris_v0.2.0_windows_amd64.zip` 得到 `iris.exe`，放入某目录并加进 PATH；PowerShell 执行 `iris.exe version` 验证。

---

## ③ 首次配置（必做一次）

```bash
iris config set api-key <你的APIKey>       # 必填：向翻译服务鉴权
iris config show                           # 查看配置（api-key 脱敏显示）
```

- 配置保存在 `~/.config/iris-cli/config.json`。
- 服务地址**默认测试环境** `https://translate-api-test.crushonfeature.com`。
  切换服务地址：`iris config set base-url <URL>`。

---

## ④ 翻译

```bash
iris translate -f <源语言代码> -t <目标语言代码> "要翻译的文本"
echo "要翻译的文本" | iris translate -f en -t zh      # 也可从管道读取
```

**参数**：
| 参数 | 说明 |
|---|---|
| `-f, --from` | 源语言代码（必填）|
| `-t, --to` | 目标语言代码（必填）|
| `--ctx <role>:<content>` | 追加一条**上下文**（可多次）；role ∈ `system`\|`user`\|`anchor`（或 `0`\|`1`\|`2`）|
| `--context <JSON>` | 直接传上下文 JSON 数组，如 `'[{"role":1,"content":"上一句话"}]'` |
| `--json` | 输出完整 JSON 响应（默认只打印译文）|
| `--api-key` | 覆盖本次 API Key |
| `--base-url` | 覆盖本次服务地址 |

**配置优先级**：命令行参数 > 环境变量 `IRIS_API_KEY` / `IRIS_BASE_URL` > 配置文件。

**常用语言代码**（用代码，不用中文名）：
`en` 英语 · `zh` 简体中文 · `zh-Hant` 繁体中文 · `fr` 法语 · `ja` 日语 · `ko` 韩语 · `es` 西班牙语 · `de` 德语 · `ru` 俄语 · `pt` 葡萄牙语 · `vi` 越南语 · `th` 泰语 · `id` 印尼语。

---

## ⑤ 上下文（context）—— 让翻译更准

聊天/对话场景下，把**上文**传给模型能改善语气、指代、省略主语等的翻译质量。上下文本身**不会被翻译**，只作背景参考。

**role 含义**：`system`(0) 系统/场景设定 · `user`(1) 对话中的用户消息 · `anchor`(2) 关键锚点（如对方昵称、称谓）。

两种传法（可混用，按出现顺序累加）：
```bash
# 方式一：--ctx role:content（可重复，适合人/AI 逐条给）
iris translate -f zh -t en \
  --ctx user:"你昨天说的那个方案" \
  --ctx anchor:"小王" \
  "同意吗"

# 方式二：--context 直接传 JSON 数组（适合 AI 一次性构造）
iris translate -f zh -t en \
  --context '[{"role":1,"content":"你昨天说的那个方案"},{"role":2,"content":"小王"}]' \
  "同意吗"
```

---

## ⑥ 完整示例

```bash
iris config set api-key sk-xxxxx
iris translate -f en -t zh "he free tonight?"                    # → 今晚有空吗？
iris translate -f zh -t en "今晚有空吗？"
echo "Bonjour" | iris translate -f fr -t zh
iris translate -f zh -t en --ctx user:"上一句聊天内容" "在吗"     # 带上下文
iris translate -f en -t zh --json "hello"                        # 看完整 JSON
```

---

## ⑦ 给 AI 助手的操作要点

- 用户说「帮我把 X 翻译成 Y 语言」时：
  1. 先确认已安装（`iris version`）且已配 api-key（`iris config show`），否则先引导安装/配置。
  2. 执行 `iris translate -f <源> -t <目标> "X"`，把 stdout 的译文回给用户。
  3. 源语言不确定时，按文本自动推断填 `-f`，或直接问用户。
- **聊天翻译带上下文**：若用户提供了对话的上文，用 `--ctx user:"上一句"`（可多条）或 `--context '<JSON>'` 传入，翻译更贴合语气/指代。上下文不会被翻译。
- **错误处理**：
  - `错误: 未配置 API Key` → 引导执行 `iris config set api-key <KEY>`。
  - `错误: --ctx 需 role:content 格式` → 用户 `--ctx` 少了冒号或 role，改成 `--ctx user:内容`。
  - `服务返回 401/403` → api-key 无效或错误，检查 key。
  - `服务返回 4xx`（如未知语种）→ 检查 `-f/-t` 语言代码是否受支持。
  - `请求失败`（网络）→ 检查 base-url 可达性（`iris config show` 看当前地址）。
- **校验包完整性**（可选）：把包和 `SHA256SUMS` 放同一目录，`shasum -a 256 -c SHA256SUMS`（Linux 用 `sha256sum -c`），对应行应为 `OK`。

---

**仅分发二进制，不含源码** ｜ 问题反馈联系服务维护者。
