# 项目特定知识（Project Context）

## 项目名称
wisdom-modules-car（车辆管理模块）

## 核心业务范围
- 车辆基础信息管理（CarInfo）
- 车辆类型树形管理（CarType）
- 年检、保险管理
- 加油管理（CarRefuel + OilCard + OilCardRecord）
- 维修管理（CarRepair + CarRepairAccessories）
- 油卡充值/扣款及变动记录
- 电子路单（ElectronicWaybill）

## 代码生成标准规约（核心参照）

详见 **AGENTS.md** 主文件中的「项目代码生成标准规约（车辆管理模块）」章节。

关键强制要求（生成新代码时必须遵守）：
- 实体继承 `BatisPlusBase` 并定义 `FieldEnum`
- 请求VO 支持 Add/Update 分组校验 + `@JsonView`
- 业务逻辑写在 `XXXVDService` 中，遵循现有 CRUD + 分页模板
- 对象转换必须通过 `CarBeanMapperConvert`
- 添加部门数据权限过滤
- 同一天查询时处理时间范围（00:00:00 ~ 23:59:59.999）
- 多车牌/多设备绑定使用逗号分隔字符串 + 转换工具方法

## 当前模块技术决策
- 多模块 Maven 结构（dto / service / server）
- Nacos 配置中心 + 多环境 bootstrap
- MyBatis-Plus + 动态数据源 `@DS("master")`
- Sa-Token + 自定义权限工具
- MapStruct 统一转换

## 待补充知识
- 更多业务规则（后续从 requirements/ 提炼）
- 踩坑记录（例如油卡扣款事务、电子路单状态流转等）