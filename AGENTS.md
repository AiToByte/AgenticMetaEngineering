# AGENTS.md
**AI 的入职手册（最重要文件）**

> 本文件是团队所有 AI 在处理 wisdom 项目时的**最高层指引**。  
> 所有代码生成、需求分析、规范遵循必须优先参考本文件。

## 1. 项目协作模式（单仓库模式）

本仓库采用 **AgenticMetaEngineering 单仓库模式**，目的是让所有服务（车辆管理、巡检管理以及后续更多服务）共享同一套知识体系和代码规范，避免风格漂移和重复建设。

**核心原则**：
- **共性统一**：所有服务必须严格遵守 AGENTS.md 中的**通用规约**
- **服务独立**：每个服务在 `context/project/` 下拥有独立的 `* -standard.md` 文件，描述该服务的特有规则
- **知识闭环**：发现新的共性内容 → 立即补充到 AGENTS.md；服务特有内容 → 放入对应 `xxx-standard.md`
- master 分支保持干净，作为模板；实际开发在独立分支或 workspace 中进行

## 2. 通用代码生成规约（所有服务必须严格遵守）

### 2.1 技术栈与架构共性
- Spring Boot + MyBatis-Plus + Maven 多模块结构（dto / service / server）
- 配置中心：Nacos + 多环境 `bootstrap-*.yaml`
- 数据源：统一使用 `@DS("master")`
- 权限控制：Sa-Token + `DataPermissionUtils`
- 对象转换：MapStruct（`componentModel = "spring"`）
- 时间类型：优先使用 `OffsetDateTime`
- 分页请求：统一继承 `PageConditionDto`
- 异常处理：统一使用 `ServiceException + ExceptionTypeEnum`
- Lombok 风格：`@Data`、`@SuperBuilder`、`@Accessors(chain = true)`、`@EqualsAndHashCode(callSuper = false)`

### 2.2 实体类（Domain）共性
- 必须继承 `BatisPlusBase`
- 必须定义内部枚举 `FieldEnum implements FieldEnumCommonDB`
- List 类型字段必须使用 `JacksonTypeHandler`
- 所有字段添加完整 `@Schema` 描述

### 2.3 请求/响应VO共性
- 请求VO 支持 `Add`、`Update` 分组校验 + `@JsonView`
- 分页请求必须继承 `PageConditionDto`
- 时间字段使用 `@DateTimeFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss.SSSZ")`

### 2.4 VDService 层共性
- 核心业务逻辑统一写在 `XXXVDService`
- CRUD 操作遵循标准模板（add / update / delById / findById / pageByCond）
- 必须调用 `DataPermissionUtils.applyDeptDataPermission(...)`
- 同一天时间范围查询必须处理为 `00:00:00 ~ 23:59:59.999`
- 复杂操作添加 `@Transactional(rollbackFor = Exception.class)`

### 2.5 查询方案共性
- **简单查询**：优先使用 `LambdaQueryWrapper`
- **复杂查询**（统计、分组、递归、多表、JSON 操作等）：**必须使用 Mapper.xml** + `<resultMap>`
- 参考模板：`CarInfoMapper.xml` 和 `InspectionTaskMapper.xml`

### 2.6 MapStruct 转换共性
- 统一使用 `XXXBeanMapperConvert` 接口（car 使用 `CarBeanMapperConvert`，inspection 使用 `InspectionBeanMapperConver`）

## 3. 服务级核心规约（独立文件）

以下文件为各服务的**权威核心规约**，生成对应服务代码时必须同时参考 AGENTS.md + 该服务规约文件：

- **车辆管理服务**（wisdom-modules-car）：  
  [`context/project/CAR-STANDARD.MD`](context/project/CAR-STANDARD.MD)

- **巡检管理服务**（wisdom-modules-inspection）：  
  [`context/project/INSPECTION-STANDARD.MD`](context/project/INSPECTION-STANDARD.MD)

**后续新增服务**（例如：xxx-modules-xxx）：
- 在 `context/project/` 下新建 `XXX-STANDARD.MD`
- 在本文件第 3 节添加指向链接

## 4. AI 生成代码的标准 Prompt（推荐模板）

当让 AI 生成或修改任意服务代码时，请在 Prompt 最前面加入以下内容：

> “请严格按照 AGENTS.md 中的通用规约 + context/project/CAR-STANDARD.MD（或 INSPECTION-STANDARD.MD）生成代码：
> - 实体、VO、VDService 必须遵守顶层共性规范
> - 复杂查询必须使用 Mapper.xml + ResultMap
> - 保持与现有同服务代码风格完全一致”

## 5. 重要提醒

- **永远优先复用**：已有工具类、VDService 模板、Mapper.xml 模式、MapStruct 转换器
- **发现新共性**：立即补充到 AGENTS.MD
- **服务特有内容**：放入对应 `XXX-STANDARD.MD`
- **风格一致性**：所有生成的代码必须与现有 car 和 inspection 模块风格保持统一

---