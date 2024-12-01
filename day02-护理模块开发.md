# 护理模块开发

## 1 目标

- 能够掌握Mybatis自动填充的字段流程
- 能够完成护理计划的所有接口开发
- 能够完成护理等级的接口开发

## 2 Mybatis字段自动填充

当我们新增护理项目的时候，并没有单独设置创建人和创建时间和修改时间字段，但是在表中存储的数据中，这三个字段是有值的，这是为什么呢？

是因为在当前项目提供了一个Mybatis字段自动填充拦截器，代码存放在zzyl-common模块中，如下图：

![image-20240721233247161](assets/image-20240721233247161.png)

原理解析

![image-20240721233313658](assets/image-20240721233313658.png)

- 当执行mapper中的方法，如果是insert或者是update方法，则会执行自动填充拦截器
- 在拦截器中，如果是新增方法，则会给参数对象的CREATE_BY、*CREATE_TIME、UPDATE_TIME三个字段赋值*
  - CREATE_BY 根据当前登录人获取用户数据进行填充
  - *CREATE_TIME、UPDATE_TIME  赋值当前时间*
- 如果是更新方法，则会给参数对象*UPDATE_BY、UPDATE_TIME三个字段赋值*
  - *UPDATE_BY 根据当前登录人获取用户数据进行填充*
  - *UPDATE_TIME  赋值当前时间*
- 注意：赋值是给参数对象赋值，在对象中必须包含这四个字段才能赋值，不然会报错



## 3 护理计划

### 3.1 需求说明

护理计划主要作用是整合多个护理项目，一个护理计划可以包含多个护理项目，并且可以设置每个护理项目的执行频次和执行周期。

下图护理计划的列表查询，需要分页条件查询

![image-20240722191043599](assets/image-20240722191043599.png)

- 除此之外，还可以对护理计划进行新增、删除、编辑、查看、禁用和启用，这些按钮分别对应了不同的操作接口



下图是新增护理计划的弹窗，一个护理计划可以设置多个护理项目，并且一个护理计划中不能包含相关的护理项目

<img src="assets/image-20240722191118363.png" alt="image-20240722191118363" style="zoom:80%;" />



查看护理计划

<img src="assets/image-20240722191142284.png" alt="image-20240722191142284" style="zoom:80%;" />





### 3.2 表结构说明

护理计划和护理项目是多对多的关系，其中有一个中间表来表示两者之间的关系

![image-20240722191515147](assets/image-20240722191515147.png)

对应的实体类，在项目中已提供



### 3.3 条件分页查询护理计划

#### 3.3.1 接口说明

- **接口地址**:`/nursing/plan/search`

- **请求方式**:`GET`

- **请求参数**:

  | 参数名称 | 参数说明             | 数据类型       |
  | -------- | -------------------- | -------------- |
  | name     | 护理计划名称         | string         |
  | pageNum  | 页码（默认为1）      | integer(int32) |
  | pageSize | 每页大小（默认为10） | integer(int32) |
  | status   | 护理计划状态         | integer(int32) |

- **响应示例**:

  ```json
  {
      "code": 200,
      "msg": "操作成功",
      "data": {
          "total": "4",
          "pageSize": 10,
          "pages": "1",
          "page": 1,
          "records": [
              {
                  "id": "4",
                  "createTime": "2023-09-26 15:32:13",
                  "updateTime": "2023-09-26 15:50:05",
                  "createBy": "1671403256519078153",
                  "updateBy": "1671403256519078153",
                  "creator": "行政专员",
                  "planName": "三级护理计划",
                  "status": 1,
                  "count": 1
              }
          ]
      },
      "operationTime": null
  }
  ```



#### 3.3.2 控制层代码

在zzyl-web模块下创建新的NursingPlanController，并且定义分页条件方法，详细代码如下：

```java
package com.zzyl.controller;

import com.zzyl.base.PageResponse;
import com.zzyl.base.ResponseResult;
import com.zzyl.service.NursingPlanService;
import com.zzyl.vo.NursingPlanVo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/nursing")
public class NursingPlanController {

    @Autowired
    private NursingPlanService nursingPlanService;


    @GetMapping("/plan/search")
    public ResponseResult<PageResponse<NursingPlanVo>> searchNursingPlan(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) Integer status,
            @RequestParam(required = false, defaultValue = "1") Integer pageNum,
            @RequestParam(required = false, defaultValue = "10") Integer pageSize) {
        PageResponse<NursingPlanVo> planVoPageResponse = nursingPlanService.listByPage(name, status, pageNum, pageSize);
        return ResponseResult.success(planVoPageResponse);
    }
}
```



#### 3.3.3 service业务层

新增NursingPlanService

```java
package com.zzyl.service;

import com.zzyl.base.PageResponse;
import com.zzyl.vo.NursingPlanVo;

public interface NursingPlanService {

    /**
     * 分页条件查询
     * @param name
     * @param status
     * @param pageNum
     * @param pageSize
     * @return
     */
    PageResponse<NursingPlanVo> listByPage(String name, Integer status, Integer pageNum, Integer pageSize);
}
```

新增实现类，具体代码，如下：

```java
package com.zzyl.service.impl;

import com.github.pagehelper.Page;
import com.github.pagehelper.PageHelper;
import com.zzyl.base.PageResponse;
import com.zzyl.mapper.NursingPlanMapper;
import com.zzyl.service.NursingPlanService;
import com.zzyl.vo.NursingPlanVo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class NursingPlanServiceImpl implements NursingPlanService {

    @Autowired
    private NursingPlanMapper nursingPlanMapper;

    /**
     * 分页条件查询
     * @param name
     * @param status
     * @param pageNum
     * @param pageSize
     * @return
     */
    @Override
    public PageResponse<NursingPlanVo> listByPage(String name, Integer status, Integer pageNum, Integer pageSize) {

        // 使用 PageHelper 分页插件
        PageHelper.startPage(pageNum, pageSize);
        // 获取所有护理计划
        List<NursingPlanVo> list = nursingPlanMapper.listByPage(name, status);
        Page<NursingPlanVo> page = (Page<NursingPlanVo>) list;
        return PageResponse.of(page, NursingPlanVo.class);
    }
}

```



#### 3.3.4 mapper持久层

新增NursingPlanMapper

```java
package com.zzyl.mapper;

import com.zzyl.vo.NursingPlanVo;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface NursingPlanMapper {
    List<NursingPlanVo> listByPage(String name, Integer status);
}
```

NursingPlanMapper.xml映射文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zzyl.mapper.NursingPlanMapper">

    <resultMap id="NursingPlanMap" type="com.zzyl.vo.NursingPlanVo">
        <id property="id" column="id"/>
        <result property="planName" column="plan_name"/>
        <result property="sortNo" column="sort_no"/>
        <result column="status" property="status"/>
        <result column="create_by" property="createBy"/>
        <result column="update_by" property="updateBy"/>
        <result column="remark" property="remark"/>
        <result column="create_time" property="createTime"/>
        <result column="update_time" property="updateTime"/>
        <result column="creator" property="creator"/>
        <result column="lid" property="lid"/>
        <result column="count" property="count"></result>
    </resultMap>


    <select id="listByPage" resultMap="NursingPlanMap">
        SELECT np.id, np.status, np.sort_no, np.plan_name, np.create_by, np.update_by, np.remark, np.create_time,np.update_time
        , su.real_name as creator,count(nl.id) count
        FROM nursing_plan np
        LEFT JOIN sys_user su ON np.create_by = su.id
        left join nursing_level nl on np.id = nl.lplan_id
        <where>
            <if test="name != null and name != ''">
                and np.plan_name like CONCAT('%',#{name},'%')
            </if>
            <if test="status != null">
                and np.status = #{status}
            </if>
        </where>
        group by np.id
        ORDER BY np.sort_no, np.create_time DESC

    </select>
</mapper>
```

- 返回的count的作用是，判断当前护理计划是否被护理等级所引用
  - 返回0  在前端这条数据是可以正常操作的（删除、禁用、编辑）
  - 返回大于0的数据，说明该条数据被护理计划所引用，不能操作（删除、禁用、编辑）



#### 3.3.5 测试

可以直接打开页面进行测试

![image-20240722204052813](assets/image-20240722204052813.png)



### 3.4 新增

在新增护理计划的时候，需要查询所有的护理项目，所以，我们需要先开发一个新的接口：查询所有护理项目

#### 3.4.1 查询所有护理项目

##### 3.4.1.1 接口说明

- 接口地址:`/nursing_project/all`
- 请求方式:`GET`
- 请求参数:无
- 响应示例:
  - ```JavaScript
    {
        "code": 0,
        "data": [
                {
                    "id": "1",
                    "createTime": "2023-09-26 15:08:59",
                    "updateTime": "2023-10-28 01:59:27",
                    "createBy": "1671403256519078153",
                    "creator": "行政专员",
                    "name": "翻身拍背",
                    "orderNo": 1,
                    "unit": "次",
                    "price": 30.00,
                    "image": "https://yjy-slwl-oss.oss-cn-hangzhou.aliyuncs.com/424a475d-76b1-4c57-b430-60293901ef79.jpg",
                    "nursingRequirement": "帮助患者翻身，并进行背部按摩，以促进血液循环",
                    "status": 0,
                    "count": 2
                }
        ],
        "msg": "",
        "operationTime": ""
    }
    ```

##### 3.4.1.2 控制层代码

在NursingProjectController中定义查询所有护理项目的方法

```java
@GetMapping("/all")
public ResponseResult<List<NursingProjectVo>> getAll(){
    return ResponseResult.success(nursingProjectService.selectAll());
}
```



##### 3.4.1.3 service业务层

在NursingProjectService中定义查询所有护理项目的方法

```java
/**
 * 查询所有护理项目
 * @return
 */
List<NursingProjectVo> selectAll();
```

在NursingProjectServiceImpl中实现方法，查询mapper

```java
@Override
public List<NursingProjectVo> selectAll() {
    return nursingProjectMapper.selectAll();
}
```



##### 3.4.1.4 mapper持久层

在NursingProjectMapper中定义方法，如下

```java
@Select("select * from nursing_project")
List<NursingProjectVo> selectAll();
```



##### 3.4.1.5 测试

<img src="assets/image-20240722224432555.png" alt="image-20240722224432555" style="zoom:67%;" />



#### 3.4.2 新增护理计划

##### 3.4.2.1 接口说明

**接口地址**:`/nursing/plan`

**请求方式**:`POST`

**请求示例**:

```json
{
  "id": 0,
  "planName": "",//计划名称
  "projectPlans": [
    {
      "executeCycle": 0,//执行周期 0 天 1 周 2月
      "executeFrequency": 0,//执行频次
      "executeTime": "",//计划执行时间
      "planId": 0,//计划id
      "projectId": 0,//项目id
      "remark": ""//备注
    }
  ],
  "remark": "",//备注
  "sortNo": 0,//排序号
  "status": 0//状态（0：禁用，1：启用）
}
```

**响应示例**:

```json
{
    "code": 0,
    "data": {},
    "msg": "",
    "operationTime": ""
}
```



##### 3.4.2.2 控制层代码

在NursingPlanController定义新的方法，如下：

```java
@PostMapping("/plan")
public ResponseResult addNursingPlan(@RequestBody NursingPlanDto dto) {
    nursingPlanService.add(dto);
    return ResponseResult.success();
}
```

- @RequestBody  注解的作用是，接收前端传递的json格式的数据，并转换为对象

##### 3.4.2.3 service业务层

在NursingPlanService中新增方法：

```java
/**
 * 新增护理计划
 * @param dto
 */
void add(NursingPlanDto dto);
```

在NursingPlanServiceImpl实现类中新增方法，并编写逻辑

```java
@Autowired
private NursingProjectPlanMapper nursingProjectPlanMapper;

/**
 * 新增护理计划
 * @param dto
 */
@Override
public void add(NursingPlanDto dto) {
    NursingPlan plan = BeanUtil.toBean(dto, NursingPlan.class);
    plan.setStatus(1);
    nursingPlanMapper.insert(plan);
    dto.getProjectPlans().forEach(v -> v.setPlanId(plan.getId()));
    //批量新增
    //类型转换
    List<NursingProjectPlan> nursingProjectPlans = BeanUtil.copyToList(dto.getProjectPlans(), NursingProjectPlan.class);
    nursingProjectPlanMapper.insertList(nursingProjectPlans);
}
```



##### 3.4.2.4 mapper持久层

由于要两个张表，需要定义在两个mapper中进行定义

（1）在NursingPlanMapper中新增方法

```java
void insert(NursingPlan plan);
```

对应的映射文件：

```xml
 <insert id="insert" parameterType="com.zzyl.entity.NursingPlan" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO nursing_plan(status, sort_no, plan_name, create_by, update_by, remark, create_time, update_time)
        VALUES (#{status}, #{sortNo}, #{planName}, #{createBy}, #{updateBy}, #{remark}, #{createTime}, #{updateTime});
    </insert>
```

- useGeneratedKeys  主键返回，当数据插入之后，会把值回显的对应的主键中
- keyProperty  需要指定实体类中对应的主键的属性

（2）新增NursingProjectPlanMapper接口,并定义批量新增的方法

```java
package com.zzyl.mapper;

import com.zzyl.dto.NursingProjectPlanDto;
import com.zzyl.entity.NursingProjectPlan;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@Mapper
public interface NursingProjectPlanMapper {


    void insertList(@Param("list") List<NursingProjectPlan> projectPlans);
}
```

新增NursingProjectPlanMapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zzyl.mapper.NursingProjectPlanMapper">


    <insert id="insertList" parameterType="java.util.List">
        INSERT INTO nursing_project_plan (plan_id, project_id, execute_time, execute_cycle, execute_frequency,
        create_by, update_by, remark, create_time, update_time)
        VALUES
        <foreach collection="list" item="item" index="index" separator=",">
            (#{item.planId}, #{item.projectId}, #{item.executeTime}, #{item.executeCycle}, #{item.executeFrequency},
            #{item.createBy}, #{item.updateBy}, #{item.remark}, #{item.createTime}, #{item.updateTime})
        </foreach>
    </insert>

</mapper>
```

##### 3.4.2.5 测试

打开前端页面，测试是否能正常保存护理计划，能保存成功即接口没问题

<img src="assets/image-20240722210405469.png" alt="image-20240722210405469" style="zoom:67%;" />





### 3.5 根据id查询

#### 3.5.1 接口说明

**接口地址**:`/nursing/plan/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明   | 请求类型 | 数据类型       |
| -------- | ---------- | -------- | -------------- |
| id       | 护理计划ID | path     | integer(int64) |

**响应示例**：

```json
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "id": "96",
        "createTime": "2023-12-04 19:54:29",
        "updateTime": "2023-12-04 19:54:29",
        "createBy": "1",
        "planName": "123",
        "status": 1,
        "projectPlans": [
            {
                "id": "1672",
                "createTime": "2023-12-04 19:54:29",
                "createDay": null,
                "updateTime": "2023-12-04 19:54:29",
                "createBy": "1",
                "planId": "96",
                "projectId": "75",
                "executeTime": "08:00",
                "executeCycle": 1,
                "executeFrequency": 1,
                "projectName": "侦察骑士"
            }
        ]
    },
    "operationTime": null
}
```



#### 3.5.2 控制层代码

在NursingPlanController中定义方法，如下：

```java
@GetMapping("/plan/{id}")
public ResponseResult<NursingPlanVo> getNursingPlanById(@PathVariable Long id) {
    return ResponseResult.success(nursingPlanService.getById(id));
}
```



#### 3.5.3 service业务层

在NursingPlanService中定义方法，如下：

```java
/**
 * 根据id查询护理计划
 */
NursingPlanVo getById(Long id);
```

在NursingPlanServiceImpl实现类中，实现方法如下：

```java
/**
 * 根据id查询护理计划
 * @param id
 */
@Override
public NursingPlanVo getById(Long id) {

    NursingPlanVo nursingPlanVo = nursingPlanMapper.getById(id);

    //根据护理计划查询护理项目列表
    Long planId = nursingPlanVo.getId();
    List<NursingProjectPlanVo> nursingProjectPlans = nursingProjectPlanMapper.getByPlanId(planId);
    nursingPlanVo.setProjectPlans(nursingProjectPlans);
    return nursingPlanVo;
}
```



#### 3.5.4 mapper持久层

这个依然需要从两个表中去查询数据，所以需要在两个mapper中都需要查询

（1）在NursingPlanMapper中定义查询的方法

```
NursingPlanVo getById(Long id);
```

NursingPlanMapper.xml映射文件

```xml
<resultMap id="NursingPlanMap" type="com.zzyl.vo.NursingPlanVo">
    <id property="id" column="id"/>
    <result property="planName" column="plan_name"/>
    <result property="sortNo" column="sort_no"/>
    <result column="status" property="status"/>
    <result column="create_by" property="createBy"/>
    <result column="update_by" property="updateBy"/>
    <result column="remark" property="remark"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
    <result column="creator" property="creator"/>
    <result column="lid" property="lid"/>
    <result column="count" property="count"></result>
</resultMap>
<select id="getById" resultMap="NursingPlanMap">
    select * from nursing_plan where id = #{id}
</select>
```



（2）在NursingProjectPlanMapper中新增方法，如下：

```
List<NursingProjectPlanVo> getByPlanId(Long planId);
```

NursingProjectPlanMapper.xml文件中的定义，如下：

```xml
<resultMap id="BaseResultMap" type="com.zzyl.vo.NursingProjectPlanVo">
    <id property="id" column="id"/>
    <result property="planId" column="plan_id"/>
    <result property="projectId" column="project_id"/>
    <result property="executeTime" column="execute_time"/>
    <result property="executeCycle" column="execute_cycle"/>
    <result property="executeFrequency" column="execute_frequency"/>
    <result column="create_by" property="createBy"/>
    <result column="update_by" property="updateBy"/>
    <result column="remark" property="remark"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
    <result column="project_name" property="projectName"/>
</resultMap>

<select id="getByPlanId" resultMap="BaseResultMap">
    SELECT npp.*, np.name project_name from nursing_project_plan npp left join nursing_project np on npp.project_id = np.id
             where plan_id = #{planId}
</select>
```



#### 3.5.5 测试

点击前端的按钮，查看是否可以详细查看护理计划和护理项目的数据

![image-20240722214001482](assets/image-20240722214001482.png)



### 3.6 修改

#### 3.6.1 接口说明

**接口地址**:`/nursing/plan/{id}`

**请求方式**:`PUT`

**请求示例**:

```json
{
  "id": 0,
  "planName": "",//计划名称
  "projectPlans": [
    {
      "executeCycle": 0,//执行周期 0 天 1 周 2月
      "executeFrequency": 0,//执行频次
      "executeTime": "",//计划执行时间
      "id":0
      "planId": 0,//计划id
      "projectId": 0,//项目id
      "remark": ""//备注
    }
  ],
  "remark": "",//备注
  "sortNo": 0,//排序号
  "status": 0//状态（0：禁用，1：启用）
}
```

**响应示例**:

```json
{
    "code": 0,
    "data": {},
    "msg": "",
    "operationTime": ""
}
```

#### 3.6.2 控制层代码

在NursingPlanController定义修改的方法：

```java
@PutMapping("/plan")
public ResponseResult updateNursingPlan(@RequestBody NursingPlanDto dto) {
    nursingPlanService.update(dto);
    return ResponseResult.success();
}
```



#### 3.6.3 service业务层

在NursingPlanService中定义修改的方法，如下：

```java
/**
 * 修改护理计划
 * @param dto
 */
void update(NursingPlanDto dto);
```

在NursingPlanServiceImpl中定义修改的方法，编写修改的逻辑

```java
/**
 * 修改护理计划
 */
@Override
public void update(NursingPlanDto dto) {
    NursingPlan plan = BeanUtil.toBean(dto, NursingPlan.class);
    //删除之前的关系
    nursingProjectPlanMapper.deleteByPlanId(dto.getId());

    //重新批量添加
    dto.getProjectPlans().forEach(v -> v.setPlanId(plan.getId()));
    //批量新增
    //类型转换
    List<NursingProjectPlan> nursingProjectPlans = BeanUtil.copyToList(dto.getProjectPlans(), NursingProjectPlan.class);
    nursingProjectPlanMapper.insertList(nursingProjectPlans);

    //修改护理计划
    nursingPlanMapper.update(plan);
}
```



#### 3.6.4 mapper持久层

（1）在NursingProjectPlanMapper中定义根据护理计划ID删除的方法

```java
@Delete("delete from nursing_project_plan where plan_id = #{planId}")
    void deleteByPlanId(Long planId);
```



（2）在NursingPlanMapper中定义修改的方法

```java
void update(NursingPlan plan);
```

NursingPlanMapper.xml文件

```xml
<update id="update" parameterType="com.zzyl.entity.NursingPlan">
    UPDATE nursing_plan
    SET plan_name   = #{planName},
        update_time = #{updateTime},
        update_by   = #{updateBy},
        status= #{status},
        sort_no= #{sortNo}
    WHERE id = #{id}
</update>
```



#### 3.6.5 测试

测试护理计划是否能修改成功



### 3.7 删除

#### 3.7.1 接口说明

**接口地址**:`/nursing/plan/{id}`

**请求方式**:`DELETE`

**请求参数**:

| 参数名称 | 参数说明   | 请求类型 | 数据类型       |
| -------- | ---------- | -------- | -------------- |
| id       | 护理计划ID | path     | integer(int64) |

**响应示例**:

```json
{
    "code": 0,
    "data": {},
    "msg": "",
    "operationTime": ""
}
```



#### 3.7.2 控制层代码

在NursingPlanController定义删除的方法：

```java
@DeleteMapping("/plan/{id}")
public ResponseResult deleteNursingPlan(@PathVariable Long id) {
    nursingPlanService.deleteById(id);
    return ResponseResult.success();
}
```



#### 3.7.3 service业务层

在NursingPlanService中定义删除的方法，如下：

```java
/**
 * 删除护理计划
 * @param id
 */
void deleteById(Long id);
```

在NursingPlanServiceImpl中定义删除的方法，编写删除的逻辑

```java
/**
 * 删除护理计划
 * @param id
 */
@Override
public void deleteById(Long id) {
    //删除之前的关系
    nursingProjectPlanMapper.deleteByPlanId(id);
    nursingPlanMapper.deleteById(id);
}
```



#### 3.7.4 mapper持久层

在NursingPlanMapper定义删除的方法

```java
@Delete("delete from nursing_plan where id = #{id}")
void deleteById(Long id);
```



#### 3.7.5 测试

测试是否能删除成功

![image-20240722220250003](assets/image-20240722220250003.png)



### 3.8 启用禁用



#### 3.8.1 接口说明

**接口地址**:`nursing/plan/{id}/status/{status}`

**请求方式**:`PUT`

**请求参数**:

| 参数名称 | 参数说明               | 请求类型 | 数据类型       |
| -------- | ---------------------- | -------- | -------------- |
| id       | 护理等级ID             | path     | integer(int64) |
| status   | 状态，0：禁用，1：启用 | path     | integer(int32) |

**响应示例**:

```json
{
    "code": 0,
    "data": {},
    "msg": "",
    "operationTime": ""
}
```

#### 3.8.2 控制层代码

在NursingPlanController定义删除的方法：

```java
@PutMapping("/{id}/status/{status}")
public ResponseResult enableOrDisable(@PathVariable Long id,@PathVariable Integer status) {
    nursingPlanService.enableOrDisable(id, status);
    return ResponseResult.success();
}
```



#### 3.8.3 service业务层

在NursingPlanService中定义启用禁用的方法，如下：

```java
/**
 * 启用或禁用
 * @param id
 * @param status
 */
void enableOrDisable(Long id, Integer status);
```

在NursingPlanServiceImpl中定义启用禁用的方法，编写启用禁用的逻辑

```java
/**
 * 启用或禁用
 * @param id
 * @param status
 */
@Override
public void enableOrDisable(Long id, Integer status) {
    nursingPlanMapper.updateStatus(id, status);
}
```



#### 3.8.4 mapper持久层

在NursingPlanMapper定义启用修改的方法

```java
/**
 * 启用或禁用护理计划
 * @param id ID
 * @param status 状态，0：禁用，1：启用
 */
void updateStatus(@Param("id") Long id, @Param("status") Integer status);
```

在NursingPlanMapper定义修改的方法

```xml
<update id="updateStatus">
    UPDATE nursing_plan
    SET status      = #{status},
        update_time = #{updateTime},
        update_by   = #{updateBy}
    WHERE id = #{id}
</update>
```



#### 3.8.5 测试

测试启用和禁用是否可以修改状态

![image-20240722222859852](assets/image-20240722222859852.png)



### 3.9 查询所有护理计划

#### 3.9.1 接口说明

**接口地址**:`/nursing/plan`

**请求方式**:`GET`

**请求参数**:无

**响应示例**:

```JSON
{
    "code": 200,
    "msg": "操作成功",
    "data": [
        {
            "id": "4",
            "createTime": "2023-09-26 15:32:13",
            "updateTime": "2023-09-26 15:50:05",
            "createBy": "1671403256519078153",
            "updateBy": "1671403256519078153",
            "planName": "三级护理计划",
            "status": 1,
        }
    ],
    "operationTime": null
}
```



#### 3.9.2 控制层代码

在NursingPlanController定义查询所有的方法：

```java
@GetMapping("/plan")
public ResponseResult<List<NursingPlanVo>> getAllNursingPlan() {
    return ResponseResult.success(nursingPlanService.getAllNursingPlan());
}
```



#### 3.9.3 service业务层

在NursingPlanService中定义查询所有的方法，如下：

```java
/**
 * 查询所有护理计划
 * @return
 */
public List<NursingPlanVo> getAllNursingPlan();
```

在NursingPlanServiceImpl中定义查询所有的方法，编写查询所有的逻辑

```java
@Override
public List<NursingPlanVo> getAllNursingPlan() {
    return nursingPlanMapper.listAll();
}
```



#### 3.9.4 mapper持久层

在NursingPlanMapper定义查询所有的方法

```java
/**
 * 查询所有护理计划
 * @return
 */
@Select("select * from nursing_plan")
List<NursingPlanVo> listAll();
```



#### 3.9.5 测试

打开护理等级的页面，点击新增按钮，查看是否可以正常查询护理计划的数据

![image-20240722223019481](assets/image-20240722223019481.png)



## 4 护理等级

### 4.1 需求说明



![image-20240722192121316](assets/image-20240722192121316.png)



![image-20240722192108097](assets/image-20240722192108097.png)





![image-20240722192140981](assets/image-20240722192140981.png)



### 4.2 表结构说明



![image-20240722192307506](assets/image-20240722192307506.png)

### 4.3 接口说明

#### 4.3.1 查询所有护理等级

**接口地址**:`/nursingLevel/listAll`

**请求方式**:`GET`

**请求参数**:无

**响应示例**:

```JSON
{
    "code": 200,
    "msg": "操作成功",
    "data": [
            {
                "id": "3",
                "createTime": "2023-09-26 15:48:18",
                "updateTime": "2023-09-26 15:51:13",
                "createBy": "1671403256519078153",
                "updateBy": "1671403256519078153",
                "creator": "行政专员",
                "name": "二级护理等级",
                "planName": "二级护理计划",
                "planId": "3",
                "fee": 1500.00,
                "status": 1,
                "description": "二级护理等级适用于能够自理但需要一定辅助护理的老年人，该等级提供了中等程度的照护项目",
            },
            {
                "id": "2",
                "createTime": "2023-09-26 15:45:12",
                "updateTime": "2023-09-26 15:45:12",
                "createBy": "1671403256519078153",
                "creator": "行政专员",
                "name": "一级护理等级",
                "planName": "一级护理计划",
                "planId": "2",
                "fee": 2000.00,
                "status": 1,
                "description": "一级护理等级适用于需要一定程度护理和照料的老年人，该等级提供了基本的日常护理项目",
            }
        ]
}
```

#### 4.3.2 新增

**接口地址**:`/nursingLevel/insert`

**请求方式**:`POST`

**请求示例**:

```JavaScript
{
  "description": "",
  "fee": 0,
  "id": 0,
  "name": "",
  "planId": 0,
  "planName": "",
  "remark": "",
  "status": 0
}
```

**响应示例**:

```JavaScript
{
        "code": 0,
        "data": {},
        "msg": "",
        "operationTime": ""
}
```

#### 4.3.3 条件分页查询

**接口地址**:`/nursingLevel/listByPage`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明     | 数据类型       |
| -------- | ------------ | -------------- |
| pageNum  | 页码         | integer(int32) |
| pageSize | 每页大小     | integer(int32) |
| name     | 护理等级名称 | string         |
| status   | 护理等级状态 | integer(int32) |

**响应示例**:

```JavaScript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "total": "6",
        "pageSize": 10,
        "pages": "1",
        "page": 1,
        "records": [
            {
                "id": "3",
                "createTime": "2023-09-26 15:48:18",
                "updateTime": "2023-09-26 15:51:13",
                "createBy": "1671403256519078153",
                "updateBy": "1671403256519078153",
                "creator": "行政专员",
                "name": "二级护理等级",
                "planName": "二级护理计划",
                "planId": "3",
                "fee": 1500.00,
                "status": 1,
                "description": "二级护理等级适用于能够自理但需要一定辅助护理的老年人，该等级提供了中等程度的照护项目",
            },
            {
                "id": "2",
                "createTime": "2023-09-26 15:45:12",
                "updateTime": "2023-09-26 15:45:12",
                "createBy": "1671403256519078153",
                "creator": "行政专员",
                "name": "一级护理等级",
                "planName": "一级护理计划",
                "planId": "2",
                "fee": 2000.00,
                "status": 1,
                "description": "一级护理等级适用于需要一定程度护理和照料的老年人，该等级提供了基本的日常护理项目",
            }
        ]
    },
    "operationTime": null
}
```

#### 4.3.4 根据id查询

**接口地址**:`/nursingLevel/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明   | 请求类型 | 数据类型       |
| -------- | ---------- | -------- | -------------- |
| id       | 护理等级ID | path     | integer(int64) |

**响应示例**:

```JavaScript
{
        "code": 0,
        "data": {
                "id": "2",
                "createTime": "2023-09-26 15:45:12",
                "updateTime": "2023-09-26 15:45:12",
                "createBy": "1671403256519078153",
                "creator": "行政专员",
                "name": "一级护理等级",
                "planName": "一级护理计划",
                "planId": "2",
                "fee": 2000.00,
                "status": 1,
                "description": "一级护理等级适用于需要一定程度护理和照料的老年人，该等级提供了基本的日常护理项目",
            },
        "msg": "",
        "operationTime": ""
}
```

#### 4.3.5 修改

**接口地址**:`/nursingLevel/update`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "description": "",
  "fee": 0,
  "id": 0,
  "name": "",
  "planId": 0,
  "planName": "",
  "remark": "",
  "status": 0
}
```

**响应示例**:

```JavaScript
{
        "code": 0,
        "data": {},
        "msg": "",
        "operationTime": ""
}
```

#### 4.3.6 删除

**接口地址**:`/nursingLevel/delete/{id}`

**请求方式**:`DELETE`

**请求参数**:

| 参数名称 | 参数说明   | 请求类型 | 数据类型       |
| -------- | ---------- | -------- | -------------- |
| id       | 护理等级ID | path     | integer(int64) |

**响应示例**:

```JavaScript
{
        "code": 0,
        "data": {},
        "msg": "",
        "operationTime": ""
}
```

#### 4.3.7 启用禁用

**接口地址**:`/nursingLevel/{id}/status/{status}`

**请求方式**:`PUT`

**请求参数**:

| 参数名称 | 参数说明               | 请求类型 | 数据类型       |
| -------- | ---------------------- | -------- | -------------- |
| id       | 护理等级ID             | path     | integer(int64) |
| status   | 状态，0：禁用，1：启用 | path     | integer(int32) |

**响应示例**:

```JavaScript
{
        "code": 0,
        "data": {},
        "msg": "",
        "operationTime": ""
}
```



### 4.4 代码生成

```json
你是一个资深的Java开发工程师，现在需要帮我生成代码，要求如下：
1，帮我生成controller、service、mapper三层的代码
2，其中service需要有实现类
3, 使用的技术范围：Springmvc、spring、MyBatis
4，mapper不仅仅有mapper接口，sql的定义需要生成xml的映射文件
5，具体生成的功能包含，新增、根据id查询，修改、分页条件查询，、根据id删除、根据id和status修改状态
6，分页条件查询，条件为pageNum,pageSize,name(模糊查询),status
7，分页使用PageHepler插件完成
8，基础的包名为：com.zzyl
9，详细的表结构如下：
CREATE TABLE "nursing_level" (
  "id" int NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  "name" varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '等级名称',
  "lplan_id" int NOT NULL COMMENT '护理计划ID',
  "fee" decimal(10,2) NOT NULL COMMENT '护理费用',
  "status" tinyint(1) NOT NULL DEFAULT '1' COMMENT '状态（0：禁用，1：启用）',
  "description" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '等级说明',
  "create_time" datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  "create_by" bigint DEFAULT NULL COMMENT '创建人id',
  "update_by" bigint DEFAULT NULL COMMENT '更新人id',
  "remark" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '备注',
  "update_time" datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY ("id") USING BTREE,
  UNIQUE KEY "name" ("name") USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=73 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=DYNAMIC COMMENT='护理等级表';
```

