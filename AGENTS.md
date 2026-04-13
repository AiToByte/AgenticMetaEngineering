# AGENTS.md  
**AI入职手册 & 项目代码生成标准规约（车辆管理模块）**

## 1. 项目背景与目标

本项目为 **wisdom-modules-car** 车辆管理模块，采用 **Spring Boot + MyBatis-Plus + Maven 多模块** 架构，服务于企业内部车辆全生命周期管理（基础信息、类型、年检、保险、加油、维修、油卡、电子路单等）。

**核心目标**：  
通过抽取共性，形成**统一、严格、可复用的代码生成规约**，确保后续AI生成的所有车辆相关模块代码在风格、结构、命名、异常处理、权限控制等方面保持高度一致，避免风格漂移。

## 2. 技术栈与架构规范（必须严格遵守）

- **框架**：Spring Boot 3.x + MyBatis-Plus + MapStruct
- **多模块结构**：
  - `wisdom-modules-car-dto`：存放 Domain 实体、ReqVo、RespVo
  - `wisdom-modules-car-service`：DAO、Service 接口、ServiceImpl、Mapper.xml
  - `wisdom-modules-car-server`：Controller、VDService（视图服务）、工具类、配置、启动类
- **配置中心**：Nacos（多环境 bootstrap 配置）
- **权限**：Sa-Token + 自定义 `DataPermissionUtils`
- **对象转换**：MapStruct（统一使用 `CarBeanMapperConvert` 风格）
- **数据源**：`@DS("master")`
- **时间类型**：优先使用 `OffsetDateTime`，少数场景使用 `LocalDateTime`
- **分页基类**：所有分页请求继承 `PageConditionDto`

## 3. 包结构与命名规范

**dto 模块**：
```
com.wisdom.modules.cardb.domain.*          // 实体类（继承 BatisPlusBase）
com.wisdom.modules.cardb.vo.request.*      // ReqVo（按业务子模块分包）
com.wisdom.modules.cardb.vo.response.*     // RespVo
```

**service 模块**：
```
com.wisdom.modules.cardb.dao.*             // xxxDao extends BaseMapper
com.wisdom.modules.cardb.service.*         // 接口
com.wisdom.modules.cardb.service.impl.*    // 实现类（带 @DS("master")）
```

**server 模块**：
```
com.wisdom.modules.car.controller.*        // Controller
com.wisdom.modules.car.service.*VDService  // 视图服务（核心业务逻辑）
com.wisdom.modules.car.util.*              // 工具类、常量、转换器
```

**命名规则**：
- 实体：`CarXXX`、`OilCardXXX`
- 请求VO：`XXXReqVo`（支持 Add/Update 分组校验）
- 响应VO：`XXXRespVo`、`XXXExportVO`
- 服务：`XXXVDService`（视图+数据传输）
- Mapper：`CarXXXMapper.xml`

## 4. 实体类（Domain）编写规范

所有实体必须遵守以下模板：

```java
@Data
@SuperBuilder
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@NoArgsConstructor
@AllArgsConstructor
@TableName(value = "table_name", autoResultMap = true)
public class CarXXX extends BatisPlusBase {

    public enum FieldEnum implements FieldEnumCommonDB {
        // 所有字段枚举（用于动态查询）
        fieldName("`column_name`"),
        ...
    }

    @Schema(description = "...", example = "...")
    @TableField("`column_name`")
    private Type fieldName;

    // List 类型使用 JacksonTypeHandler
    @TableField(value = "`attachment`", typeHandler = JacksonTypeHandler.class)
    private List<String> attachment;
}
```

**必备点**：
- 必须继承 `BatisPlusBase`
- 必须定义 `FieldEnum` 枚举
- 使用 `@Schema` 完整描述
- 字段名与数据库列名一致（使用反引号）

## 5. 请求VO（ReqVo）编写规范

```java
@Data
@SuperBuilder   // 或 @Builder
public class XXXReqVo {
    public interface Add {}
    public interface Update {}
    public interface XXXExtra {}

    @Schema(...)
    @JsonView({Update.class})
    @NotBlank(groups = {Update.class})
    private String id;

    // 其他字段...
}
```

- 分页请求继承 `PageConditionDto`
- 必须支持分组校验（Add / Update）
- 使用 `@JsonView` 控制序列化
- 时间字段使用 `@DateTimeFormat` + `OffsetDateTime`

## 6. VDService 层编写规范（最核心共性）

所有业务逻辑放在 `XXXVDService` 中，CRUD 模板高度统一：

```java
@Service
@RequiredArgsConstructor
public class CarXXXVDService {

    private final XXXService xxxService;           // mybatis-plus service
    private final CarBeanMapperConvert beanMapper;

    // 标准方法：add / update / delById / findById / pageByCond / remove(List)

    public XXX add(String menuId, XXXReqVo vo) { ... }
    public XXX update(String menuId, XXXReqVo vo) { ... }
    public PageResultDto<XXXRespVo> pageByCond(XXXPageReqVo vo) { ... }
}
```

**必做事项**：
- 使用 `DataPermissionUtils.applyDeptDataPermission(...)` 处理数据权限
- 同一天时间范围处理：`beginTime` → 当天00:00:00，`endTime` → 当天23:59:59.999
- 复杂逻辑使用事务 `@Transactional`
- 异常统一抛 `ServiceException(ExceptionTypeEnum.xxx)`

## 7. MapStruct 转换规范

统一使用 `CarBeanMapperConvert` 接口：

```java
@Mapper(componentModel = "spring")
public interface CarBeanMapperConvert {

    CarXXX toCarXXX(XXXReqVo vo);
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateCarXXX(XXXReqVo vo, @MappingTarget CarXXX entity);
    XXXRespVo toXXXRespVo(CarXXX entity);
}
```

## 8. 其他共性规范

- **常量**：放在 `util` 包下的 `XXXConst.java`
- **导出/导入**：统一使用 `EasyExcelUtil`
- **列表转字符串**（多车牌、多设备绑定）：使用逗号分隔 + `convertListToString` 方法
- **油卡相关**：充值/扣款必须同时写 `OilCardRecord`，状态常量使用 `OilCardRecordConst`
- **树形结构**（车辆类型）：使用 CTE 递归查询

## 9. 代码生成 Prompt 参照模板（后续AI生成时直接使用）

当你让AI生成新业务时，请在Prompt中加入以下关键指令：

> “严格按照 wisdom-modules-car 模块的代码生成标准规约进行开发：
> - 实体必须继承 BatisPlusBase 并定义 FieldEnum
> - VO 使用 Add/Update 分组校验 + @JsonView
> - 业务逻辑写在 XXXVDService 中，遵循现有 CRUD 模板
> - 使用 CarBeanMapperConvert 进行转换
> - 添加数据权限过滤和同一天时间范围处理
> - 保持与现有 CarRepair、OilCard 等模块完全一致的风格”