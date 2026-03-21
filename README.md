# AgenticMetaEngineering

## 团队共享：单仓库模式


> 为什么用单仓库？



既然大模型是无状态函数，那么团队其实不需要"50 个 Agent 实例"，只需要"**1 个 Agent 工程 + 50 个独立的上下文空间**"。



这里说的"单仓库"有两层含义：


### 第一层：团队共享同一套 AI 配置（知识层面）



传统做法是每个人各自维护自己的 AI 配置——你的 CLAUDE.md 和我的 CLAUDE.md 完全不同。问题是：

* 你踩过的坑，我不知道
* 好的提示词技巧无法共享
* 团队知识变成孤岛

**单仓库的解法**：把 CLAUDE.md、context/、.codebuddy/ 等 AI 配置文件**统一放在一个 Git 仓库里，团队成员通过 clone 这个仓库获得完全一致的 AI "记忆"**。



### 第二层：并行工作时的隔离（执行层面）

Claude Code 创作者的实践验证了这一点：

> “我在终端中并行运行 5 个 Claude。我使用同一仓库的 5 个独立 git checkout。”

关键在于：每个 checkout 都包含**相同的 CLAUDE.md 和团队知识**（因为都是从同一个仓库 clone 的），但各自在**独立的分支**上工作，互不干扰。


```md
团队共享仓库（master）
├── CLAUDE.md          ← 所有人共用的 AI 记忆
├── context/           ← 所有人共用的知识库
└── .codebuddy/        ← 所有人共用的工具

开发者 A 的 checkout     开发者 B 的 checkout     开发者 C 的 checkout
├── (继承 master)        ├── (继承 master)        ├── (继承 master)
├── feature/auth         ├── feature/payment      ├── bugfix/login
└── 独立的工作空间        └── 独立的工作空间        └── 独立的工作空间
```

**这样实现了**：知识共享（大家的 AI 都知道同样的事情）+ 工作隔离（各自的代码改动不会冲突）。



**总结——传统模式 vs 单仓库模式：**
|维度|传统模式|单仓库模式
|---|---------|---------|
|AI配置| 每人各自维护->知识孤岛| 统一工程, 所有人共用一套AGENTS.md|
|踩坑经验| 你踩过的坑, 同事不知道 -> 重复踩坑| 记录到`context/`, 通过PR共享给所有人|
|好的实践| 只能口头相传| Git版本管理, 合并到master瞬间分发|
|并行工作| 容易冲突| 独立分支隔离，互不干扰|



#### **目录结构**
```md
AgenticMetaEngineering/
├── AGENTS.md              # AI 的"入职手册"（最重要）
├── context/                  # 团队知识库
│   ├── team/                 # 团队通用知识
│   └── project/              # 项目特定知识
├── requirements/             # 需求记录（Git 管理）
│   └── {requirement-id}/
├── workspace/                # [废弃] 占位目录（实际工作区迁移至 ../workspace/）
│   └── {requirement-id}/
└── .codebuddy/
    └── commands/             # 自定义命令
```

#### **分支策略**
```md
master（模板，保持干净）
├── AGENTS.md      # 团队共享的提示词
├── context/          # 团队共享的知识库
└── .codebuddy/       # 团队共享的工具

feature/your-work（你的工作分支）
├── 继承 master 全部内容
├── requirements/     # 你的需求记录
└── ../workspace/     # 你的代码（与本仓库同级）
```

**核心原则：**
* master 是模板，不直接在上面工作
* 创建分支开始需求开发
* 好的实践通过 PR 合并回 master，所有人受益



#### **上下文检索：为什么不需要 RAG**

很多人第一反应是"要不要用向量数据库做 RAG？"——答案是：**不需要**。
Claude Code 团队最初也尝试过向量 embeddings，但发现了问题：
* **维护成本高**：需要持续重建索引，本地文件修改后难以实时同步
* **安全风险**：独立的向量数据库增加了攻击表面积
* **检索准确率**：对于代码库这种结构化数据，简单的语义搜索往往不如 Agent 主动探索精准

**我们的方案**：直接给 AI 赋予 `grep`、`find`、`ls` 能力。它能像资深工程师一样，通过文件结构和关键词自己找到答案。



**组织原则：**
按领域分文件夹：`tech/`、`business/`、`experience/`
文件名要清晰：让 `ls` 出来的列表就有语义
内容是 Markdown：最自然的文本格式



#### **知识路由表**
| 信息类型| 放哪里 | 示例 |
| --------- | ------ | ---- |
| 新人必读的项目背景| `AGENTS.md` | "本项目使用DDD架构"|
| 踩坑经验 | `context/experience/` | "我踩过一个坑，在`context/tech/`里记录了" |
| 业务逻辑规则 | `context/business/` | "VIP用户提现额度计算规则" |
| 整体计划 | `plan.md` | "designing完成后进入developing" |
| 当前任务进度 | `process.txt` | "API完成, 下一步写测试" |
| 已确认关键发现 | `notes.md` | "JWT exp 使用UTC时区, 需要注意转换" |
| 未确认临时记录 | `process.txt` | "发现User表有个字段没用, 待确认"|


**决策口诀：**
1. 是整体计划？→ `plan.md`
2. 是当前进度？→ `process.txt`
3. 是长期知识？
    * 所有人都得知道？→ `AGENTS.md`
    * 特定领域？→ `context/` 目录

