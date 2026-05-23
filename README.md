# Smart Git — 你的 AI 智能提交助手

> 告别 "git add ." 和绞尽脑汁写 commit message 的日子。选择性暂存、AI 自动生成规范提交信息，让每一次提交都干净利落。

---

## 它是谁

Smart Git 是一套 Git 命令行增强工具，专为追求效率的开发者设计。它不会替代 Git，而是让最常用的 "暂存 → 提交" 流程变得丝般顺滑：

- 不想全部提交？用 `git lna` 交互式勾选要暂存的文件
- 写不出 commit message？用 `git agm` 让 AI 根据你的代码变更自动生成
- 一次改了很多文件？用 `git agm -g` 让 AI 帮你按功能拆分成多个提交

所有操作都走标准的 `git add` 和 `git commit`，完整兼容你的 Git Hooks（husky、lint-staged 等），绝不绕过校验。

---

## 核心功能

### 1. 选择性暂存 — "只提交我想提交的"

`git lna`（list not added）会列出工作区中所有未暂存的变更：

- 修改、新增、删除、重命名... 每种状态一目了然
- 空格键勾选/取消，方向键移动，Enter 确认
- 支持未跟踪文件（自动排除 .gitignore 的内容）
- Ctrl+C 随时退出，已选中的不会乱动

```
? 未暂存的文件（空格选中/取消，Enter 确认暂存；Ctrl+C 退出）
 ❯◯ [修改] src/ai/orchestration/race.mjs
  ◯ [修改] src/cli/interactive/aigcm.mjs
  ◉ [新增] src/ai/system-prompt-group.mjs
  ◯ [修改] package.json
```

### 2. AI 生成提交信息 — "让 AI 读代码，你来拍板"

`git agm`（或 `git aigcm`）会读取暂存区的代码变更，调用 AI 生成规范的 Conventional Commits 格式提交信息：

**生成的格式示例：**
```
✨ feat: 新增分组提交模式

- AI 自动分析 diff 并按功能拆分多个提交组
- 每组独立生成 commit message 并逐组确认
- 支持编辑、重试、跳过等灵活操作
```

**支持的 AI 后端（可同时配置多个）：**
- OpenAI 兼容接口（阿里云百炼、公司私有网关、vLLM 等）
- Anthropic Claude API
- 本地 Ollama
- Claude Code CLI（已安装 claude 命令时）

**多 Provider 竞速：** 配置多个 AI 后端时，它们会同时发起请求，哪个先返回就用哪个结果——不用干等某一家超时。

### 3. 分组提交 — "大改动也能拆得明明白白"

`git agm -g` 启用分组模式，AI 会分析你的暂存区变更，按功能逻辑拆分成多个提交组：

```
📋 AI 建议的提交分组方案：

  [组 1] ✨ feat: 新增用户登录功能
    src/auth/login.js
    src/auth/login.test.js

  [组 2] 🐛 fix: 修复分页查询返回空数组的问题
    src/api/pagination.js

共 2 组
```

确认后，工具会逐组：取消暂存 → 只 add 该组文件 → AI 生成 message → 提交。每组都是独立的 commit，历史清晰可追溯。

### 4. 快捷语法 — "一行字搞定提交"

如果你只想快速提交，不用 AI，可以用 `@` 规则手写：

| 标记 | 作用 | 示例 |
|------|------|------|
| `@d` | 描述（必填） | `@d 修复登录失败` |
| `@t` | 类型简写 | `@t bug` → 自动映射为 `fix` |
| `@z` | 禅道 ID | `@z 1234` → 展开为 `Ref: ZENTAO-1234` |
| `@r` | 备注 | `@r 产品需求` → 展开为 `备注: 产品需求` |

示例：`@d 修复登录 @t bug @z 1234` → `🐛 fix: 修复登录` + 正文 `Ref: ZENTAO-1234`

### 5. 实时链路状态 — "看清每一次 AI 调用"

确认提交信息前，终端会展示一张实时刷新的链路状态表：

```
📡 所有链路请求状态：
─────────────────────────────────────────────────────────
▏ 描述          ▏ 类型        ▏ 模型                ▏ 请求状态 ▏ 耗时   ▕
─────────────────────────────────────────────────────────
▏ 阿里云百炼    ▏ openaiCompat ▏ qwen3-coder-flash  ▏ 200    ▏ 2.35s ▕
▏ 公司平台      ▏ openaiCompat ▏ qwen3.5-397b       ▏ 检测中  ▏ 检测中 ▕
▏ 本地 Ollama   ▏ ollama      ▏ gemma-4-26B-GGUF   ▏ 502    ▏ 1.02s ▕
─────────────────────────────────────────────────────────

⚡️ 当前最快链路：阿里云百炼｜耗时：2.35s
🗂️ 当前配置文件：[Project 项目级] 文件地址：/Users/you/project/.smart-commit.jsonc
```

每条链路的 HTTP 状态、耗时、错误信息一目了然。绿色 = 成功，黄色 = 客户端错误，红色 = 服务端错误，灰色 = 还在跑。

---

## 界面一览

```
# git lna — 选择性暂存

? 未暂存的文件（空格选中/取消，Enter 确认暂存）
 ❯◉ [修改] src/ai/orchestration/race.mjs
  ◯ [新增] src/ai/system-prompt-group.mjs
  ◉ [修改] src/cli/interactive/aigcm.mjs
(Move up and down to reveal more choices)

✅ 已暂存 2 个文件

---

# git agm — AI 生成提交

 src/ai/orchestration/race.mjs      |  126 ++++++++
 src/cli/interactive/aigcm.mjs      |  274 ++++++++++++
 2 files changed, 400 insertions(+)

⏳ AI 正在生成提交信息… 已用时 3s

🤖 AI 生成提交信息：

✨ feat: 新增多 Provider 竞速与实时链路追踪

- 并发调用所有启用的 AI provider，首个成功即返回
- 支持两阶段 trace 协议，实时推送链路状态到确认弹窗
- 失败链路展示 HTTP 状态与错误摘要

📡 所有链路请求状态：
...（实时刷新的链路表格）...

? 是否使用此信息提交？(y/n/q/r/e)
```

---

## 典型使用场景

**场景一：日常开发小改动**
> 改了两个文件，用 `git lna` 勾选一个暂存，然后 `git agm` 让 AI 生成提交信息，3 秒搞定。

**场景二：大功能拆提交**
> 一次改了 15 个文件，涉及登录、分页、文档。`git agm -g` 让 AI 拆成 3 个提交组，每组一个清晰的 commit，历史干净得像艺术品。

**场景三：快速修复线上问题**
> 紧急修 bug，没空想文案。`git add` 后直接 `git agm`，AI 自动识别是修复类改动，生成 `🐛 fix: ...`，你按个 y 就提交了。

**场景四：手写快速提交**
> 不想等 AI？`git agm` 确认时按 e 打开编辑器，或者直接用 `@d 修复登录 @t bug` 一行搞定。

---

## 配置简单

一个 `.smart-commit.jsonc` 文件搞定所有配置（支持注释和尾逗号）：

```jsonc
{
  "commitlint": {
    "types": ["feat", "fix", "docs", "chore"]
  },
  "ai": {
    "providers": [
      {
        "id": "bailian",
        "desc": "阿里云百炼平台",
        "type": "openaiCompat",
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode",
        "apiKeyEnv": "DASHSCOPE_API_KEY",
        "model": "qwen3-coder-flash",
        "enabled": true
      }
    ]
  }
}
```

API Key 始终通过环境变量传入，不会写在配置文件里。项目级配置放在仓库根目录，用户级配置放在 `~/.smart-git/.smart-commit.jsonc`，团队和个人各管各的。

---

## 系统要求

- Node.js >= 18
- Git（任何近期版本）
- macOS / Linux / Windows WSL

---

## 快速开始

```bash
# 全局安装（指定 GitHub Packages registry）
pnpm install -g @wxyy-org/smart-git@latest --@wxyy-org:registry=https://npm.pkg.github.com

# 在仓库里初始化配置（可选，会生成 .smart-commit.jsonc 模板）
git agm-init

# 日常用法
git lna          # 交互式选择要暂存的文件
git agm          # AI 生成提交信息并提交
git agm -g       # 分组模式：AI 拆分多个提交
```

---

## 安全与隐私

- **API Key 不走文件**：通过环境变量传入，配置文件中只写变量名
- **操作日志本地存储**：每次调用的完整上下文写入本地日志（`~/.smart-git/logs/`），方便排查问题，不上传任何服务器
- **权限收紧**：日志文件 0600（仅本人可读），目录 0700
- **临时关闭日志**：`SMART_GIT_LOG=0 git agm`

---

*Smart Git — 让提交像呼吸一样自然。*
