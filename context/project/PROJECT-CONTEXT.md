# context/project/readme.md
## 项目特定知识（Project Specific Context）

### 项目名称
**wisdom-modules-car** —— 车辆管理核心模块

### 业务范围
- 车辆基础信息（CarInfo）
- 车辆类型树形结构（CarType）
- 年检管理（CarAnnualInspection）
- 保险管理（CarInsurance）
- 加油记录（CarRefuel）
- 维修管理（CarRepair + CarRepairAccessories）
- 油卡管理与充值/扣款（OilCard + OilCardRecord）
- 电子路单（ElectronicWaybill）

### 代码生成标准规约（强制参照）
详见 **AGENTS.md** 中的「项目代码生成标准规约（车辆管理模块）」章节，以及 `context/project/car-standard.md`（后续会单独拆出更详细版本）。

**关键强制点**（AI生成新代码时必须100%遵守）：
- 实体类必须继承 `BatisPlusBase` 并定义内部 `FieldEnum implements FieldEnumCommonDB`
- 业务逻辑统一写在 `XXXVDService` 类中，严格遵循现有 add/update/pageByCond/delById 模板
- 对象转换必须使用 `CarBeanMapperConvert`
- 多车牌、多责任人、多设备绑定统一使用逗号分隔字符串 + 转换工具方法
- 必须添加部门数据权限过滤
- 事务操作（充值、扣款等）必须使用 `@Transactional`

### 当前技术决策记录
- 采用 dto / service / server 三层 Maven 多模块结构
- 配置中心使用 Nacos + 多环境 bootstrap-*.yaml
- 数据源统一标注 `@DS("master")`
- 树形查询（车辆类型）使用 CTE 递归
- 油卡变动必须同步写入 OilCardRecord

### 待补充内容
- 更多业务规则提炼（从 requirements/ 中沉淀）
- 电子路单状态流转细节
- 维修配件与采购流程联动规范