# InkOS 新手入门指南（源码方式）

本文档适用于从源码运行 InkOS 的用户。

## 一分钟快速开始

```bash
# 1. 克隆项目
git clone https://github.com/Narcooo/inkos.git
cd inkos

# 2. 安装依赖
pnpm install

# 3. 构建
pnpm build

# 4. 配置 LLM（以 MiniMax 为例）
# 方法：创建 ~/.inkos/.env 全局配置
mkdir -p ~/.inkos
cat > ~/.inkos/.env << 'EOF'
INKOS_LLM_PROVIDER=anthropic
INKOS_LLM_BASE_URL=https://api.minimaxi.com/anthropic
INKOS_LLM_API_KEY=你的APIKey
INKOS_LLM_MODEL=MiniMax-M2.5
EOF

# 5. 创建项目
node packages/cli/dist/index.js init 我的小说
cd 我的小说

# 6. 创建书籍
node packages/cli/dist/index.js book create --title "我的小说" --genre xuanhuan

# 7. 写第一章
node packages/cli/dist/index.js write next 我的小说
```

---

## 源码使用方式

### 目录结构

```
inkos/
├── packages/
│   ├── core/          # 核心库（Agent、管线、状态管理）
│   └── cli/          # 命令行工具
├── scripts/          # 构建脚本
└── skills/           # Agent 技能
```

### 运行命令

```bash
# 方式一：直接运行
node packages/cli/dist/index.js <command>

# 方式二：添加软链接（推荐）
ln -s /path/to/inkos/packages/cli/dist/index.js /usr/local/bin/inkos
inkos <command>

# 方式三：使用 pnpm
pnpm --filter @actalk/inkos <command>
```

### 开发模式

```bash
# 监听模式（修改代码自动重编译）
pnpm dev

# 运行测试
pnpm test

# 类型检查
pnpm typecheck
```

---

## 完整使用流程

### 第 1 步：克隆与构建

```bash
git clone https://github.com/Narcooo/inkos.git
cd inkos
pnpm install
pnpm build
```

### 第 2 步：配置 LLM

**方式一：全局配置（推荐，所有项目共享）**

```bash
mkdir -p ~/.inkos
cat > ~/.inkos/.env << 'EOF'
# 必填
INKOS_LLM_PROVIDER=anthropic      # openai / anthropic / custom
INKOS_LLM_BASE_URL=https://api.minimaxi.com/anthropic
INKOS_LLM_API_KEY=你的APIKey
INKOS_LLM_MODEL=MiniMax-M2.5

# 可选
# INKOS_LLM_TEMPERATURE=0.7
# INKOS_LLM_MAX_TOKENS=8192
EOF
```

支持的模型配置：
| Provider | base-url 示例 | model 示例 |
|----------|---------------|------------|
| openai | `https://api.openai.com/v1` | `gpt-4o` |
| anthropic | `https://api.anthropic.com` | `claude-sonnet-4-20250514` |
| anthropic (MiniMax) | `https://api.minimaxi.com/anthropic` | `MiniMax-M2.5` |
| custom | `https://your-proxy.com/v1` | `gpt-4o` |

**方式二：项目级配置**

```bash
cd my-novel
cat > .env << 'EOF'
INKOS_LLM_PROVIDER=anthropic
INKOS_LLM_BASE_URL=https://api.minimaxi.com/anthropic
INKOS_LLM_API_KEY=你的APIKey
INKOS_LLM_MODEL=MiniMax-M2.5
EOF
```

项目级配置会覆盖全局配置。

### 第 3 步：初始化项目

```bash
node packages/cli/dist/index.js init 我的项目
cd 我的项目
```

### 第 4 步：创建书籍

```bash
node packages/cli/dist/index.js book create --title "书名" --genre xuanhuan

# 参数说明
# --genre: 题材 (xuanhuan/xianxia/urban/horror/other)
# --platform: 发布平台 (tomato/feilu/qidian/other)
# --chapter-words: 每章字数 (默认3000)
# --target-chapters: 目标章节数 (默认200)
```

### 第 5 步：开始写作

```bash
# 写下一章（完整流程：草稿→审计→修订）
node packages/cli/dist/index.js write next 书名

# 连续写多章
node packages/cli/dist/index.js write next 书名 --count 5

# 指定章节字数
node packages/cli/dist/index.js write next 书名 --words 5000
```

---

## 核心命令

### 写作

| 命令 | 说明 |
|------|------|
| `write next <id>` | 完整管线写下一章 |
| `draft <id>` | 只写草稿 |
| `audit <id> <n>` | 审计第 n 章 |
| `revise <id> <n>` | 修订第 n 章 |
| `write rewrite <id> <n>` | 重写第 n 章 |

### 查看与管理

| 命令 | 说明 |
|------|------|
| `status` | 查看项目状态 |
| `book list` | 列出所有书籍 |
| `review list <id>` | 审阅草稿 |
| `review approve-all <id>` | 批量通过 |

### 导出

```bash
# 导出 TXT
node packages/cli/dist/index.js export 书名

# 导出 EPUB
node packages/cli/dist/index.js export 书名 --format epub
```

### 其他

```bash
node packages/cli/dist/index.js genre list      # 查看支持的题材
node packages/cli/dist/index.js radar scan       # 扫描平台趋势
node packages/cli/dist/index.js detect <id> <n> # AIGC检测
node packages/cli/dist/index.js doctor          # 诊断配置问题
```

---

## 项目结构

```
my-novel/
├── inkos.json           # 项目配置
├── .env                 # API密钥（如不使用全局配置）
├── books/
│   └── 我的小说/
│       ├── book.json                # 书籍信息
│       ├── book_rules.md            # 创作规则
│       ├── story_bible.md           # 世界观设定
│       ├── state/                   # 真相文件（长期记忆）
│       │   ├── current_state.md     # 世界状态
│       │   ├── particle_ledger.md  # 资源账本
│       │   ├── pending_hooks.md     # 伏笔追踪
│       │   └── ...
│       ├── drafts/                  # 草稿
│       ├── approved/                # 已通过章节
│       └── snapshots/               # 状态快照
└── inkos.log            # 运行日志
```

---

## 真相文件（长期记忆）

InkOS 自动维护 7 个真相文件，确保情节一致性：

| 文件 | 用途 |
|------|------|
| `current_state.md` | 角色位置、关系、已知信息 |
| `particle_ledger.md` | 物品、金钱、资源追踪 |
| `pending_hooks.md` | 未闭合伏笔 |
| `chapter_summaries.md` | 各章摘要 |
| `subplot_board.md` | 支线进度 |
| `emotional_arcs.md` | 情感弧线 |
| `character_matrix.md` | 角色交互矩阵 |

---

## 守护进程模式

后台自动写作：

```bash
# 启动
node packages/cli/dist/index.js up

# 停止
node packages/cli/dist/index.js down

# 静默模式
node packages/cli/dist/index.js up -q
```

---

## 常见问题

**Q: 写出来的内容像 AI 生成的？**
A: InkOS 内置去 AI 味规则，使用 `style analyze <file>` 分析参考文本可注入文风。

**Q: 章节之间前后矛盾？**
A: 审计员会自动检查连续性，问题会标记需要修订。

**Q: 想从已有章节继续写？**
A: 使用 `import chapters <id> --from <path>` 导入。

**Q: 如何修改世界观设定？**
A: 编辑 `books/<书名>/story_bible.md` 和 `book_rules.md`。

**Q: 如何更新到最新代码？**
A: `git pull && pnpm install && pnpm build`

---

## 开发相关命令

```bash
pnpm build          # 构建所有包
pnpm dev            # 监听模式
pnpm test           # 运行测试
pnpm typecheck      # 类型检查

# 构建单个包
cd packages/core && pnpm build
cd packages/cli && pnpm build
```
