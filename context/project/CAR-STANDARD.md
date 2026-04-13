# CAR-STANDARAD

## 车辆管理模块 (wisdom-modules-car) 标准规约

**版本**：v1.0  
**更新日期**：2026-04-13  
**适用范围**：所有后续车辆相关模块的新功能开发、AI代码生成

---

### 1. 模块架构与分层规范（必须严格遵守）

项目采用 **Maven 多模块结构**：

- `wisdom-modules-car-dto`：实体 Domain + 请求VO + 响应VO
- `wisdom-modules-car-service`：DAO 接口、Service 接口、ServiceImpl、Mapper.xml
- `wisdom-modules-car-server`：Controller、VDService（核心业务服务）、工具类、配置、启动类

**包命名规范**：
- Domain：`com.wisdom.modules.cardb.domain`
- 请求VO：`com.wisdom.modules.cardb.vo.request.{子模块}`
- 响应VO：`com.wisdom.modules.cardb.vo.response.{子模块}`
- VDService：`com.wisdom.modules.car.service.{业务}VDService`
- 工具类：`com.wisdom.modules.car.util`

---

### 2. 实体类 (Domain) 编写规范

所有实体必须遵循以下模板：

```java
@Data
@SuperBuilder
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@NoArgsConstructor
@AllArgsConstructor
@TableName(value = "table_name", autoResultMap = true)
public class CarXXX extends BatisPlusBase {

    // 字段枚举（动态查询、权限等场景必须使用）
    public enum FieldEnum implements FieldEnumCommonDB {
        company("`company`"),
        deptId("`dept_id`"),
        // ... 所有字段都要定义
        ;
        private final String value;
        FieldEnum(String value) { this.value = value; }
        @Override
        public String getValue() { return value; }
    }

    @Schema(description = "所属公司", example = "")
    @TableField("`company`")
    private String company;

    // List 类型字段必须使用 JacksonTypeHandler
    @TableField(value = "`attachment`", typeHandler = JacksonTypeHandler.class)
    private List<String> attachment;
}
```

**强制要求：**

* 必须继承 `BatisPlusBase`
* 必须定义 `FieldEnum` 枚举
* 所有字段添加完整 `@Schema`
* 时间类型优先使用 `OffsetDateTime`

### 3. 请求VO (ReqVo) 编写规范
```java
@Data
@SuperBuilder
public class CarXXXReqVo {
    public interface Add {}
    public interface Update {}
    public interface UpdatePurchaseInfo {}   // 特殊场景可用

    @Schema(description = "数据库id", example = "xxx")
    @JsonView({Update.class})
    @NotBlank(groups = {Update.class})
    private String id;

    // 其他业务字段...
}
```

分页请求必须继承 `PageConditionDto`
支持分组校验（`Add / Update`）
时间字段无需添加其他注解, 项目中有全局格式化转换器


### 4. VDService 层编写规范（最核心共性）
所有业务逻辑统一放在 XXXVDService 中，CRUD 模板高度一致：
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class CarXXXVDService {

    private final XXXService xxxService;           // MyBatis-Plus Service
    private final CarBeanMapperConvert beanMapper;

    // 标准方法模板
    public XXX add(String menuId, XXXReqVo vo) { ... }
    public XXX update(String menuId, XXXReqVo vo) { ... }
    public void delById(String id) { ... }
    public PageResultDto<XXXRespVo> pageByCond(XXXPageReqVo vo) { ... }
}
```

**强制处理项：**

数据权限：例如:`DataPermissionUtils.applyDeptDataPermission(wrapper, StpCommonUtils.getDeptIdsFromSession(), vo.getDeptId(), OilCard::getDeptId, deptVDService::getAllHavingDeptId);`
具体权限工具方法: 
```java
public static <T> void applyDeptDataPermission(
            LambdaQueryWrapper<T> queryWrapper,
            List<String> userDeptIds,
            String frontendDeptId,
            SFunction<T, ?> deptField,
            Function<String, List<String>> getSubDeptIdsFunc) {

        if (CollectionUtils.isEmpty(userDeptIds)) {
            // 用户无任何部门权限，返回空结果
            queryWrapper.apply("1 = 0");
            return;
        }

        queryWrapper.and(wrapper -> {
            if (StringUtils.isNotBlank(frontendDeptId)) {
                // 前端指定了部门，需获取其所有子部门
                List<String> allowedDepts = getSubDeptIdsFunc.apply(frontendDeptId);
                if (CollectionUtils.isEmpty(allowedDepts)) {
                    wrapper.apply("1 = 0"); // 无子部门，返回空
                    return;
                }

                // 取交集：前端部门子集 ∩ 用户权限部门
                allowedDepts.retainAll(userDeptIds);
                if (allowedDepts.isEmpty()) {
                    wrapper.apply("1 = 0"); // 无交集，无权限
                } else {
                    wrapper.in(deptField, allowedDepts);
                }
            } else {
                // 未指定部门，直接按用户权限过滤
                wrapper.in(deptField, userDeptIds);
            }
        });
    }
```
同一天时间范围处理（`beginTime / endTime`）
复杂操作添加 `@Transactional(rollbackFor = Exception.class)`
异常统一使用 `ServiceException`
大量复用 `CarBeanMapperConvert`


### 5. 查询方案规范（新增重点章节）

#### 5.1 简单查询优先使用 LambdaQueryWrapper

- 在 `XXXVDService` 的 `pageByCond`、`findById` 等方法中，优先使用 `LambdaQueryWrapper` 进行动态条件拼接。
- 必须处理以下共性逻辑：
  - `del_flag = 0`
  - 部门数据权限：`DataPermissionUtils.applyDeptDataPermission(wrapper, ... , OilCard::getDeptId, deptVDService::getAllHavingDeptId)`
  - 同一天时间范围处理（startTime / endTime）：
    ```java
    if (startTime != null && endTime != null && startTime.toLocalDate().isEqual(endTime.toLocalDate())) {
        startTime = startTime.toLocalDate().atStartOfDay().atOffset(startTime.getOffset());
        endTime = startTime.toLocalDate().atTime(LocalTime.MAX).atOffset(endTime.getOffset());
    }
    ```

### 5.2 复杂查询必须使用 Mapper.xml + 自定义 SQL
何时必须使用 `Mapper.xml`：

需要统计（COUNT、SUM、GROUP BY）
涉及递归查询（树形结构，如车辆类型）
多表关联或多项复杂条件（IN、FIND_IN_SET、CTE 等）
需要自定义 ResultMap
查询性能敏感或SQL较长时

标准查询链路模板（以 `CarInfo` 的 `cardStatistics` 为例，已提取并模块化）：

**完整链路：**
1. Controller → 调用 `CarInfoVDService.cardStatistics(...)`
2. VDService（CarInfoVDService）：
* 组装权限（deptIds）
* 调用 carInfoService.cardStatistics(menuId, allDeptIds, carTypeId)

3. Service 接口（`CarInfoService`）声明方法
4. ServiceImpl 直接委托给 Dao（或直接调用 Dao 方法）
5. Dao 接口（`CarInfoDao`）定义方法：`List<CarInfoCardStatisticsRespVo> cardStatistics(...)`
6. Mapper.xml（`CarInfoMapper.xml`）书写具体 SQL + ResultMap

推荐 `Mapper.xml` 书写规则（严格遵循 `CarInfoMapper.xml` 风格）：
```XML
<mapper namespace="com.wisdom.modules.cardb.dao.CarInfoDao">

    <!-- 1. 定义 ResultMap（强烈推荐，用于复杂统计返回VO） -->
    <resultMap id="CardStatisticsResultMap" type="com.wisdom.modules.cardb.vo.response.carinfo.CarInfoCardStatisticsRespVo">
        <result column="deptId" property="deptId"/>
        <result column="projectId" property="projectId"/>
        <result column="sumCount" property="sumCount"/>
        <!-- ... 其他映射 -->
    </resultMap>

    <!-- 2. SQL 使用 <select> + 动态 <if> / <choose> / <foreach> -->
    <select id="cardStatistics" resultMap="CardStatisticsResultMap">
        SELECT
            dept_id AS deptId,
            project_id AS projectId,
            COUNT(1) AS sumCount,
            SUM(CASE WHEN status = '1' THEN 1 ELSE 0 END) AS normalUseCount,
            SUM(CASE WHEN status = '0' THEN 1 ELSE 0 END) AS underRepairCount
        FROM car_info
        WHERE del_flag = 0
        <if test="menuId != null and menuId != ''">
            AND menu_id = #{menuId}
        </if>
        <if test="carTypeId != null and carTypeId != ''">
            AND find_in_set(#{carTypeId}, car_type_bak)
        </if>
        <!-- 部门权限 -->
        <if test="allDeptIds != null and allDeptIds.size() > 0">
            AND dept_id IN
            <foreach collection="allDeptIds" item="deptId" open="(" separator="," close=")">
                #{deptId}
            </foreach>
        </if>
        <!-- 分组逻辑 -->
        <choose>
            <when test="allDeptIds != null and allDeptIds.size() > 0">
                GROUP BY project_id
            </when>
            <otherwise>
                GROUP BY dept_id
            </otherwise>
        </choose>
        ORDER BY deptId ASC, projectId ASC
    </select>

</mapper>
```

**复杂查询最佳实践：**

* 使用 <resultMap> 而不是默认 resultType（尤其返回自定义VO时）
* 递归查询（如车辆类型树）使用 WITH RECURSIVE
* 大量 IN 查询时使用 <foreach>
* 保持 SQL 可读性，合理换行和缩进
* XML 文件名与 Dao 接口一致（如 CarInfoMapper.xml）


### 6. MapStruct 转换器规范
统一使用 CarBeanMapperConvert：

```java
@Mapper(componentModel = "spring")
public interface CarBeanMapperConvert {

    CarXXX toCarXXX(XXXReqVo vo);

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateCarXXX(XXXReqVo vo, @MappingTarget CarXXX entity);

    XXXRespVo toXXXRespVo(CarXXX entity);
}
```


### 7. 其他共性规范

多值绑定（车牌、责任人、设备）：使用逗号分隔字符串，统一通过工具方法转换
油卡操作：充值/扣款必须同时写入 `OilCardRecord`，状态常量使用 `OilCardRecordConst`
树形结构（车辆类型）：使用 CTE 递归查询
导出导入：统一使用 `EasyExcelUtil`
常量类：放在 `com.wisdom.modules.car.util` 下，如 `OilCardRecordConst.java`
配置：多环境使用 `bootstrap-*.yaml`，统一通过 `Nacos`