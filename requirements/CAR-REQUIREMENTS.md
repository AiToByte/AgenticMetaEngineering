# requirements/readme.md
## 需求记录（Requirements - Git 管理）

本目录用于结构化记录和管理车辆管理模块的所有需求变更。

### 使用规则
- 每个需求创建一个独立子文件夹：`requirements/REQ-XXXX-描述/`
- 子文件夹内必须包含：
  - `readme.md`（需求描述、目标、验收标准）
  - `process.md`（进度记录、决策、问题）
  - 可选：`notes.md`、`draft/`（临时草稿）
- 需求完成后：
  - 将成熟规约/经验沉淀到 `context/project/` 或 AGENTS.md
  - 代码通过 PR 合并到对应分支
  - 可将该需求文件夹归档或保留作为历史上下文

### 当前/历史需求列表

**REQ-001**：车辆管理模块共性抽取与标准规约建立（已完成）
- 状态：完成
- 成果：已填充 AGENTS.md + context/team/ + context/project/
- 关联文件：context/project/car-standard.md（规划中）

**REQ-002**：电子路单模块功能完善与规范对齐（待启动）
- 状态：规划中

**REQ-003**：油卡与维修业务联动优化（待启动）

### AI 处理需求时的标准流程
1. 阅读 AGENTS.md 中的整体规约
2. 阅读 context/project/ 中的车辆模块知识
3. 在 workspace/ 对应目录下进行草稿开发
4. 完成后更新 context/ 知识，并提交 PR

更新日期：2026-04-13