# 团队通用知识（Team Context）

## AI 协作原则（所有成员及AI必须遵守）

- **单仓库模式**：所有AI使用同一份 AGENTS.md 和 context/ 知识库，确保风格、规范、踩坑经验全团队一致。
- **知识共享优先**：任何好的实践、规约、异常处理技巧，必须通过 PR 合并到 master 的 context/ 或 AGENTS.md 中。
- **上下文工程**：AI 生成代码时必须优先引用本仓库的规约，避免重复发明轮子。
- **分支隔离**：每个人/每个需求在独立分支或独立 workspace 中工作，master 保持干净作为模板。
- **Agent 行为守则**：
  - 严格遵循已建立的代码生成标准规约（见 AGENTS.md 和 context/project/）
  - 优先复用现有工具类、MapStruct 转换器、DataPermissionUtils 等共性组件
  - 遇到不确定项，先查询 context/ 而不是随意实现
  - 输出代码必须包含清晰的 @Schema、注释和异常处理

## 通用开发规范（适用于所有模块）

- 使用 Lombok + MapStruct 减少 boilerplate
- 时间类型优先 OffsetDateTime
- 分页请求统一继承 PageConditionDto
- 数据权限必须调用 DataPermissionUtils
- 异常统一使用 ServiceException + ExceptionTypeEnum
- 导入导出统一使用 EasyExcelUtil

## 团队知识路由表

- 项目特定业务规约 → `context/project/`
- 车辆管理模块标准 → `context/project/car-standard.md`（或直接在 AGENTS.md 中）
- 踩坑经验与最佳实践 → `context/experience/`（后续创建）
- 技术决策记录 → `context/tech/`