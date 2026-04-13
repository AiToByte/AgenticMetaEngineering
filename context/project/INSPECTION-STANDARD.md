# INSPECTION-STANDARD.md

## 巡检模块 (wisdom-modules-inspection) 代码标准规约

**版本**：v1.0  
**更新日期**：2026-04-13  
**核心原则**：完全复用车辆模块 (`car-standard.md`) 的所有通用规范，仅补充巡检特有部分。

### 1. 模块结构

- `wisdom-modules-inspection-dto`：Domain + ReqVo + RespVo
- `wisdom-modules-inspection-service`：Dao + Service + ServiceImpl + Mapper.xml
- `wisdom-modules-inspection-server`：Controller + VDService + 工具类 + 定时任务

### 2. 实体类补充规范

- 继续使用 `BatisPlusBase` + `FieldEnum`
- 多选字段（如 `roleId`、`nightUserId`、`inspectionTypeIdList`）使用 `JacksonTypeHandler`
- 照片相关字段（如 `inspectionPhotos`、`rectificationPhotos`）统一使用 `List<String>` + JacksonTypeHandler

### 3. VDService 补充规范

- 统计类接口（如 `cardStatistics`、`photoCardStatistics`）优先使用 Mapper.xml + ResultMap
- 文件导出/导入统一调用 `FileTransferVDService` 异步处理
- 水印添加统一使用 `ImageWatermarkUtil.addStyledWatermark(...)`

### 4. Mapper.xml 书写规范（重点）

严格遵循 `InspectionTaskMapper.xml` 和 `CarInfoMapper.xml` 风格：

- 使用 `<resultMap>` 定义返回VO映射
- 复杂统计使用 `SUM(CASE WHEN ...)` + `GROUP BY`
- 部门权限使用 `<if test="allDeptIds != null and allDeptIds.size() > 0">`
- 时间范围使用 `>=` / `<=` + `jdbcType=TIMESTAMP`
- 递归或 JSON 操作使用原生 MyBatis 语法

示例（参考 InspectionTaskMapper.xml）：
```xml
<resultMap id="CardStatisticsResultMap" type="...RespVo">
    <result column="deptId" property="deptId"/>
    ...
</resultMap>

<select id="cardStatistics" resultMap="CardStatisticsResultMap">
    SELECT ... 
    FROM inspection_task 
    WHERE del_flag = 0
    <if test="startTime != null"> AND submit_time >= #{startTime} </if>
    ...
    <choose>
        <when test="..."> GROUP BY project_id </when>
        <otherwise> GROUP BY dept_id </otherwise>
    </choose>
</select>
```


### 5. 特有工具类与定时任务

水印工具：`ImageWatermarkUtil`（支持远程底图 + 文字叠加）
定时统计：`InspectionWorkPhotoDayStatisticsScheduled`、`InspectionWorkPhotoMonthStatisticsScheduled`
`Bean` 转换：`InspectionBeanMapperConver`（命名风格与 `CarBeanMapperConvert` 保持一致）

### 6. AI 生成 Prompt 模板（巡检模块专用）
“严格按照 `context/project/car-standard.md` 和 `context/project/inspection-standard.md` 生成巡检模块代码：
实体、VO、VDService 遵循车辆模块通用规范
复杂统计必须使用 `Mapper.xml` + `ResultMap`（参考 `InspectionTaskMapper.xml`）
水印处理调用 `ImageWatermarkUtil`
文件传输走 `FileTransferVDService` 异步
保持与现有 `InspectionTask`、`InspectionWorkPhoto` 等代码风格完全一致”