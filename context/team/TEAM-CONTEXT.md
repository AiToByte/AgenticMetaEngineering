# TEAM-CONTEXT
## 团队通用知识（Team Shared Knowledge）

### AI 协作核心原则（所有 Agent 必须严格遵守）
- 采用**单仓库模式**：整个团队共享同一份 AGENTS.md、context/ 和 .codebuddy/ 配置，确保所有AI生成的代码风格、规范、异常处理完全一致。
- **知识优先级**：任何新发现的最佳实践、规约、踩坑经验，必须及时记录到 context/ 或 AGENTS.md 中，通过 PR 共享给全团队。
- **生成代码前必须检查**：
  1. AGENTS.md 中的项目规约
  2. context/project/ 中的业务模块标准
  3. context/team/ 中的通用规范
- **禁止行为**：不得随意发明新风格、新工具类，必须最大程度复用现有共性（如 CarBeanMapperConvert、DataPermissionUtils、VDService 模板等）。

### 通用技术规范（全项目适用）
- **对象映射**：统一使用 MapStruct（componentModel = "spring"）
- **时间处理**：优先使用 `OffsetDateTime`，同一天查询时必须处理为 00:00:00 ~ 23:59:59.999
- **分页**：所有分页请求类必须继承 `PageConditionDto`
- **权限**：必须调用 `DataPermissionUtils.applyDeptDataPermission(...)`
- **异常**：统一抛 `ServiceException`，使用 `ExceptionTypeEnum`
- **Lombok**：统一使用 `@Data`、`@SuperBuilder`、`@Accessors(chain = true)`、`@EqualsAndHashCode(callSuper = false)`
- **导出导入**：统一使用 `EasyExcelUtil`

### 知识路由表
| 知识类型               | 存放位置                        | 示例 |
|------------------------|--------------------------------|------|
| 项目整体规约           | AGENTS.md                      | 车辆管理代码生成标准 |
| 车辆管理模块详细规范   | context/project/car-standard.md | Domain、VO、VDService 模板 |
| 通用技术决策与经验     | context/team/                  | 本文件 |
| 业务规则与踩坑记录     | context/project/ 或 experience/ | 油卡事务处理经验 |
| 临时工作记录           | workspace/                     | 当前需求草稿 |