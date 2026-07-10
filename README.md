# iris 翻译 CLI —— 安装与使用（供 Codex / Claude Code 执行）

> **给 AI 助手（Codex / Claude Code）的说明**：本文档帮你在非技术同事的电脑上安装并使用 `iris` 翻译命令行工具。请按顺序：① 判断平台 → ② 安装 → ③ 配置 api-key → ④ 调用翻译（可带聊天上下文）。每一步都有可直接执行的命令。

## 这是什么

`iris` 是一个**单文件命令行工具**，调用公司 iris 翻译服务（同步接口 `/api/v1/translate/sync`）完成多语种翻译，支持传入**聊天上下文**帮助模型理解语气/指代。一条命令拿译文，无需写代码。当前版本 **v0.3.0**。

---

## ① 选择对应平台的包

Release 里按「系统_架构」提供包。先判断用户环境再选：

| 用户环境 | 包文件 |
|---|---|
| Mac（Apple 芯片 M1/M2/M3…）| `iris_v0.3.0_darwin_arm64.tar.gz` |
| Mac（Intel 芯片）| `iris_v0.3.0_darwin_amd64.tar.gz` |
| Linux（x86_64）| `iris_v0.3.0_linux_amd64.tar.gz` |
| Linux（ARM64 / aarch64）| `iris_v0.3.0_linux_arm64.tar.gz` |
| Windows（64 位）| `iris_v0.3.0_windows_amd64.zip` |

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
tar xzf iris_v0.3.0_darwin_arm64.tar.gz    # 解压得到可执行文件 iris
mkdir -p ~/.local/bin && mv iris ~/.local/bin/
export PATH="$HOME/.local/bin:$PATH"       # 建议追加进 ~/.zshrc 或 ~/.bashrc
iris version                               # 验证：应输出 iris v0.3.0
```

**Mac 首次运行被拦截**（未签名 App）时任选其一：
```bash
xattr -d com.apple.quarantine ~/.local/bin/iris
```
或：系统设置 → 隐私与安全性 → 「仍要打开」。

### Windows
解压 `iris_v0.3.0_windows_amd64.zip` 得到 `iris.exe`，放入某目录并加进 PATH；PowerShell 执行 `iris.exe version` 验证。

---

## ③ 首次配置（必做一次）

```bash
iris config set api-key <你的APIKey>       # 必填：向翻译服务鉴权
iris config show                           # 查看配置（api-key 脱敏显示）
```

- 配置保存在 `~/.config/iris-cli/config.json`。
- 设置服务地址：`iris config set base-url <URL>`。

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
| `--template <名>` | 按**提示词模板**翻译（与 `--template-id` 互斥）；指定即锁定模板绑定的模型 |
| `--template-id <ID>` | 按模板 ID 翻译（与 `--template` 互斥）|
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

## ⑥ 提示词模板（template 命令）

模板 = 保存在服务端的提示词，**绑定一个模型**；翻译带 `--template <名>` 即按该模板渲染并**锁定其模型**。管理命令（可缩写 `iris tpl`）：

```bash
iris template list                     # 摘要表格：NAME/MODEL/DEFAULT/ENABLED/TEMPLATE_ID
iris template get <模板名或ID>         # 详情（含 content）；--json 输出原始 JSON
iris template delete <模板名或ID>      # 逻辑删除（数据保留在服务端库，同名可重建）
```

**创建**（内容多行，推荐 heredoc 从 stdin 传）：
```bash
iris template create --name chat-casual --model <模型名，从 list 的 MODEL 列取> --content-file - <<'EOF'
Reference the following translations and preservation rules:
{{PRESERVATION_RULES}}

{{BACKGROUND_BLOCK}}[Source Text]
{{SOURCE_TEXT}}

[Translation Tasks]
1. Translate the [Source Text] into {{TARGET_LANG}}.
2. Treat the source language as {{SOURCE_LANG}} and the target language as {{TARGET_LANG}}.
3. You must ONLY output the translated result without any additional explanation.
EOF
```
- 内容**必须包含全部 5 个占位符**：`{{SOURCE_TEXT}}` `{{SOURCE_LANG}}` `{{TARGET_LANG}}` `{{PRESERVATION_RULES}}` `{{BACKGROUND_BLOCK}}`（缺失或出现未知 `{{...}}` 服务端拒绝）。
- 也可 `--content-file tpl.txt`（文件）或 `--content "<串>"`（短内容）；可选 `--remark 备注`、`--default`（设为该模型默认）、`--disabled`。

**更新**（只改传入的字段）：
```bash
iris template update chat-casual --remark "闲聊/口语"
iris template update chat-casual --disable          # 或 --enable
iris template update chat-casual --default          # 设为其模型的默认（原默认自动退位）
```

**规则速记**：每个模型有且仅有一个**默认模板**（`DEFAULT=yes`），默认模板**只读**——修改/停用/删除都会被服务端 409 拒绝；要调整默认行为：先创建新模板并对它 `--default`，旧默认退位后即可修改/删除。

翻译时使用：`iris translate -f en -t zh --template chat-casual "文本"`（或 `--template-id <ID>`，二者互斥）。

---

## ⑦ 完整示例

```bash
iris config set api-key sk-xxxxx
iris translate -f en -t zh "he free tonight?"                    # → 今晚有空吗？
iris translate -f zh -t en "今晚有空吗？"
echo "Bonjour" | iris translate -f fr -t zh
iris translate -f zh -t en --ctx user:"上一句聊天内容" "在吗"     # 带上下文
iris translate -f en -t zh --json "hello"                        # 看完整 JSON
iris translate -f en -t zh --template chat-casual "hello"        # 按提示词模板翻译
iris template list                                               # 查看现有模板
```

---

## ⑧ 给 AI 助手的操作要点

- 用户说「帮我把 X 翻译成 Y 语言」时：
  1. 先确认已安装（`iris version`）且已配 api-key（`iris config show`），否则先引导安装/配置。
  2. 执行 `iris translate -f <源> -t <目标> "X"`，把 stdout 的译文回给用户。
  3. 源语言不确定时，按文本自动推断填 `-f`，或直接问用户。
- **聊天翻译带上下文**：若用户提供了对话的上文，用 `--ctx user:"上一句"`（可多条）或 `--context '<JSON>'` 传入，翻译更贴合语气/指代。上下文不会被翻译。
- **提示词模板**：用户要求"用 XX 模板翻译"时加 `--template <模板名>`；不确定模板名先 `iris template list`。要新建/调整模板用 `iris template create/update`（见⑥，内容须含全部 5 个占位符）。
- **错误处理**：
  - `错误: 未配置 API Key` → 引导执行 `iris config set api-key <KEY>`。
  - `错误: --ctx 需 role:content 格式` → 用户 `--ctx` 少了冒号或 role，改成 `--ctx user:内容`。
  - `错误: --template 与 --template-id 互斥` → 只传其中一个。
  - `错误: 模板不存在: xxx` / `服务返回 400`（模板相关）→ 模板名写错、已停用或占位符不合规；先 `iris template list` 核对。
  - `服务返回 409`（模板管理）→ 模板名已存在（换个名字），或试图修改/删除**默认模板**（只读；先对另一模板 `iris template update <另一模板> --default` 让它退位）。
  - `服务返回 503` → 模板绑定的模型当前不可用，换模板或联系服务维护者。
  - `服务返回 401/403` → api-key 无效或错误，检查 key。
  - `服务返回 4xx`（如未知语种）→ 检查 `-f/-t` 语言代码是否受支持。
  - `请求失败`（网络）→ 检查 base-url 可达性（`iris config show` 看当前地址）。
- **校验包完整性**（可选）：把包和 `SHA256SUMS` 放同一目录，`shasum -a 256 -c SHA256SUMS`（Linux 用 `sha256sum -c`），对应行应为 `OK`。

---

**版本**：v0.3.0 ｜ **仅分发二进制，不含源码** ｜ 问题反馈联系服务维护者。
