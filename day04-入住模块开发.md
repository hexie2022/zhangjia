# 入住模块开发

## 1 目标

- 完成入住和入住配置的基础代码快速生成
- 完成入住列表接口开发
- 完成楼层房间床位的接口开发
- 完成申请入住接口的开发
- 完成入住详情接口的开发

## 2 需求分析

### 2.1 原型需求说明

在住管理模块包含入住管理、退住管理两个子模块；

1. 入住管理展示的正在入住的老人信息，包含老人的基本信息、家属信息、入住配置、签约办理；
2. 退住管理是指当老人因个人原因或者违反养老院规定时，需办理退住手续。办理退住时需填写老人的基本信息、上传解除协议、完成费用清算等操作；

- 列表查询

![image-20240723234440138](assets/image-20240723234440138.png)

- 发起入住申请

![发起入住申请表单](assets/发起入住申请表单.png)

- 查看详情

![入住详情](assets/入住详情.png)



### 2.2 表结构说明

老人入住的表较多，直接关系或者关系的，共有12张表，关系如下图：

![image-20240723232836697](assets/image-20240723232836697.png)

- 当老人入住成功后，需要保存老人信息、固定老人床位、创建合同、入住及入住配置
- 当老人选择床位的时候，不同的房间价格是不同的，选房间也需要知道在哪个楼层
- 入住选择护理等级的时候，需要知道护理计划，护理计划中包含了哪些护理项目



## 3 代码快速生成

在申请入住中，涉及到的表很多，也就是说需要操作的表就比较多，不过，在目前的代码中，已经提供了很大一部分

- 目前项目中存在的代码：楼层、房间、房间类型、护理相关（项目、计划、等级）、合同、老人
- 目前项目中不存在的代码：入住（check_in）和入住配置表（check_in_config）



由于这些相对来说都比较复杂，我们不可能把所有的代码都生成，不过，我们可以生成的是基础的代码，比如基础的mapper和service层代码，后期可以根据需求来进行改造，这样就能加快开发的速度

注意：controller的代码全部自己编写



使用大模型提示词来生成入住（check_in）和入住配置表（check_in_config）这两个表的基础代码

1）Prompt提示词-生成check_in基础代码

```json
你是一个资深的Java开发工程师，现在需要帮我生成代码，要求如下：
1，帮我生成service、mapper三层的代码
2，其中service需要有实现类
3, 使用的技术范围：Springmvc、spring、MyBatis
4，mapper不仅仅有mapper接口，sql的定义需要生成xml的映射文件
5，具体生成的功能包含，新增、根据id查询，修改、分页条件查询，根据id删除
6，分页条件查询，条件为pageNum,pageSize,elderName,idCardNo
7，分页使用PageHepler插件完成
8，基础的包名为：com.zzyl
9，详细的表结构如下：
CREATE TABLE "check_in" (
  "id" bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  "check_in_code" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '编号',
  "title" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '标题',
  "elder_id" bigint NOT NULL COMMENT '老人id',
  "check_in_time" timestamp NULL DEFAULT NULL COMMENT '入住时间',
  "remark" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '备注',
  "applicat" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '申请人',
  "dept_no" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '部门编号',
  "applicat_id" bigint NOT NULL COMMENT '申请人id',
  "create_time" timestamp NOT NULL COMMENT '创建时间',
  "status" int NOT NULL COMMENT '入住状态，0：入住中，1：已退住',
  "other_apply_info" text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci COMMENT '其他申请信息',
  "update_time" timestamp NULL DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY ("id") USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=328 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC;

```

2）Prompt提示词-生成check_in_config基础代码

```json
你是一个资深的Java开发工程师，现在需要帮我生成代码，要求如下：
1，帮我生成controller、service、mapper三层的代码
2，其中service需要有实现类
3, 使用的技术范围：Springmvc、spring、MyBatis
4，mapper不仅仅有mapper接口，sql的定义需要生成xml的映射文件
5，具体生成的功能包含，新增、根据id查询，修改、分页条件查询，根据id删除
6，分页条件查询，条件为pageNum,pageSize,elderName,idCardNo
7，分页使用PageHepler插件完成
8，基础的包名为：com.zzyl
9，详细的表结构如下：
CREATE TABLE "check_in_config" (
  "id" bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
  "elder_id" bigint NOT NULL COMMENT '老人ID',
  "check_in_code" varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '入住编码',
  "check_in_start_time" datetime NOT NULL COMMENT '入住开始时间',
  "check_in_end_time" datetime NOT NULL COMMENT '入住结束时间',
  "nursing_level_id" bigint NOT NULL COMMENT '护理等级ID',
  "nursing_level_name" varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '护理等级名称',
  "bed_number" varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '床位号',
  "cost_start_time" datetime NOT NULL COMMENT '费用开始时间',
  "cost_end_time" datetime NOT NULL COMMENT '费用结束时间',
  "deposit_amount" decimal(10,2) NOT NULL COMMENT '押金（元）',
  "nursing_cost" decimal(10,2) NOT NULL COMMENT '护理费用（元/月）',
  "bed_cost" decimal(10,2) NOT NULL COMMENT '床位费用（元/月）',
  "other_cost" decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '其他费用（元/月）',
  "medical_insurance_payment" decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '医保支付（元/月）',
  "government_subsidy" decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '政府补贴（元/月）',
  "create_time" datetime NOT NULL COMMENT '创建时间',
  "update_time" datetime DEFAULT NULL COMMENT '更新时间',
  "create_by" bigint DEFAULT NULL COMMENT '创建人id',
  "update_by" bigint DEFAULT NULL COMMENT '更新人id',
  "remark" varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '备注',
  PRIMARY KEY ("id") USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=314 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=DYNAMIC COMMENT='入住配置表';

```



大家可以从当天的资料中获取生成后的类，方便快速开发，也可以使用AI自主生成

<img src="assets/image-20240724211113082.png" alt="image-20240724211113082" style="zoom:70%;" />

<img src="assets/image-20240724211151662.png" alt="image-20240724211151662" style="zoom:67%;" />





## 4 功能开发

### 4.1 入住列表开发

#### 4.1.1 接口说明

**接口地址**:`/check-in/pageQuery`

**请求方式**:`GET`

**请求参数**:


| 参数名称  | 参数说明           | 是否必须 | 数据类型       |
| --------- | ------------------ | -------- | -------------- |
| pageNum   | 页码               | true     | integer(int32) |
| pageSize  | 页面大小           | true     | integer(int32) |
| elderName | 老人姓名，模糊查询 | false    | string         |
| idCardNo  | 身份证号，精确查询 | false    | string         |

**响应示例**:

```javascript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "total": "2",
        "pageSize": 10,
        "pages": "1",
        "page": 1,
        "records": [
            {
                "id": "2",
                "elderName": "刘拥军",
                "elderIdCardNo": "132123195612191234",
                "bedNumber": "202-2",
                "nursingLevelName": "三级护理等级",
                "checkInStartTime": "2024-07-21 00:00:00",
                "checkInEndTime": "2024-08-31 23:59:59",
                "applicat": "超级管理员",
                "createTime": "2024-07-21 21:55:57"	
            },
            {
                "id": "1",
                "elderName": "李爱国",
                "elderIdCardNo": "132123194512121234",
                "bedNumber": "101-1",
                "nursingLevelName": "二级护理等级",
                "checkInStartTime": "2024-07-21 00:00:00",
                "checkInEndTime": "2024-08-31 23:59:59",
                "applicat": "超级管理员",
                "createTime": "2024-07-21 21:54:02"
            }
        ]
    },
    "operationTime": null
}
```

#### 4.1.2 思路说明

通过接口文档的响应结果来看，这些字段在check_in表中不能查询到全部，我们需要从三张表中来查询



![image-20240724223518458](assets/image-20240724223518458.png)



#### 4.1.3 控制层代码

在CheckInController中定义条件分页查询的方法，如下：

```java
package com.zzyl.controller;

import com.zzyl.base.PageResponse;
import com.zzyl.base.ResponseResult;
import com.zzyl.service.CheckInService;
import com.zzyl.vo.CheckInPageQueryVo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/check-in")
public class CheckInController {

    @Autowired
    private CheckInService checkInService;


    @GetMapping("/pageQuery")
    public ResponseResult<PageResponse<CheckInPageQueryVo>> pageQuery(
            @RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
            @RequestParam(value = "pageSize", defaultValue = "10") Integer pageSize,
            @RequestParam(value = "elderName", required = false) String elderName,
            @RequestParam(value = "idCardNo", required = false) String idCardNo
    ) {
        PageResponse<CheckInPageQueryVo> response = checkInService.pageQuery(pageNum, pageSize, elderName, idCardNo);
        return ResponseResult.success(response);
    }
}
```



#### 4.1.4 service业务层

在CheckInService中定义分页条件查询的方法，如下：

```java
/**
 * 分页条件查询
 * @param pageNum
 * @param pageSize
 * @param elderName
 * @param idCardNo
 * @return
 */
PageResponse<CheckInPageQueryVo> pageQuery(Integer pageNum, Integer pageSize, String elderName, String idCardNo);
```

在CheckInServiceImpl中实现接口中的方法

```java
/**
 * 分页条件查询
 * @param pageNum
 * @param pageSize
 * @param elderName
 * @param idCardNo
 * @return
 */
@Override
public PageResponse<CheckInPageQueryVo> pageQuery(Integer pageNum, Integer pageSize, String elderName, String idCardNo) {
    PageHelper.startPage(pageNum, pageSize);
    List<CheckInPageQueryVo> list =  checkInMapper.selectByPage(elderName, idCardNo);
    Page<CheckInPageQueryVo> page = (Page<CheckInPageQueryVo>) list;
    return PageResponse.of(page, CheckInPageQueryVo.class);
}
```



#### 4.1.5 mapper持久层

在CheckInMapper中定义条件查询的方法

```java
List<CheckInPageQueryVo> selectByPage(String elderName, String idCardNo);
```

在CheckInMapper.xml文件中中定义条件查询的标签，封装数据返回

```xml
<select id="selectByPage" resultType="com.zzyl.vo.CheckInPageQueryVo">
    select ci.id,e.name elder_name,e.id_card_no elder_id_card_no,
    e.bed_number,cic.nursing_level_name,cic.check_in_start_time,cic.check_in_end_time,ci.applicat,ci.create_time
    from check_in ci left join elder e on ci.elder_id = e.id
    left join check_in_config cic on e.id = cic.elder_id
    <where>
        <if test="elderName != null and elderName != ''">
            and e.name like concat('%', #{elderName}, '%')
        </if>
        <if test="idCardNo != null">
            and e.id_card_no = #{idCardNo}
        </if>
    </where>
    order by ci.create_time desc
</select>
```





#### 4.1.6 测试

打开页面，查看测试效果

![image-20240724221609179](assets/image-20240724221609179.png)



### 4.2 查询楼层房间床位

#### 4.2.1 接口说明


**接口地址**:`/floor/getRoomAndBedByBedStatus/{status}`

**请求方式**:`GET`

**请求参数**:


| 参数名称 | 参数说明                       | 是否必须 | 数据类型       |
| -------- | ------------------------------ | -------- | -------------- |
| status   | 床位状态，0：未入住，1：已入住 | true     | integer(int32) |

**响应示例**:

```javascript
{
    "code": 200,
    "msg": "操作成功",
    "data": [
        {
            "id": "403",
            "createTime": "2023-12-28 16:11:35",
            "updateTime": "2023-12-28 16:12:22",
            "createBy": "1671403256519078239",
            "updateBy": "1671403256519078239",
            "name": "1楼",
            "roomVoList": [
                {
                    "id": "80",
                    "createTime": "2023-12-28 16:11:35",
                    "updateTime": "2023-12-28 16:12:22",
                    "createBy": "1671403256519078239",
                    "updateBy": "1671403256519078239",
                    "code": "102",
                    "sort": 2,
                    "floorId": "403",
                    "bedVoList": [
                        {
                            "id": "191",
                            "bedNumber": "102-1",
                            "bedStatus": 0,
                            "roomId": "80",
                            "status": 0,
                        }
                    ],
                    "status": 0,
                    "price": 3500.00
                },
                {
                    "id": "81",
                    "createTime": "2023-12-28 16:13:35",
                    "updateTime": "2023-12-28 16:13:35",
                    "createBy": "1671403256519078239",
                    "code": "103",
                    "sort": 3,
                    "floorId": "403",
                    "bedVoList": [
                        {
                            "id": "193",
                            "bedNumber": "103-1",
                            "bedStatus": 0,
                            "roomId": "81",
                            "status": 0,
                        }
                    ],
                    "status": 0,
                    "price": 2000.00
                }
            ]
        }
    ],
    "operationTime": null
}
```

#### 4.2.2 思路说明

在接口的返回数据中，需要从四张表中查询数据，详细表关系如下：

![image-20240724231929146](assets/image-20240724231929146.png)



#### 4.2.3 控制层代码

在FloorController中新增方法，来查询楼层、房间、床位及房间价格

```java
@GetMapping("/getRoomAndBedByBedStatus/{status}")
public ResponseResult<List<FloorVo>> getRoomAndBedByBedStatus(@PathVariable Integer status){
    return success(floorService.getRoomAndBedByBedStatus(status));
}
```



#### 4.2.4 service业务层

在FloorService中新增方法

```java
/**
 * 查询楼层，房间，床位
 * @param status
 * @return
 */
List<FloorVo> getRoomAndBedByBedStatus(Integer status);
```

在FloorServiceImpl中新增实现方法

```java
/**
 * 查询楼层，房间，床位
 *
 * @param status
 * @return
 */
@Override
public List<FloorVo> getRoomAndBedByBedStatus(Integer status) {
    return floorMapper.getRoomAndBedByBedStatus(status);
}
```





#### 4.2.5 mapper持久层

在FloorMapper中定义查询的方法

```java
List<FloorVo> getRoomAndBedByBedStatus(Integer status);
```

在FloorMapper.xml文件中定义查询的方法

```java
<resultMap id="floorResultMapWithRoomAndBed" type="com.zzyl.vo.FloorVo">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="create_by" property="createBy"/>
    <result column="update_by" property="updateBy"/>
    <result column="remark" property="remark"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
    <collection property="roomVoList" ofType="com.zzyl.vo.RoomVo">
    <id property="id" column="rid"/>
    <result property="code" column="code"/>
    <result property="sort" column="sort"/>
    <result property="sort" column="sort"/>
    <result property="floorId" column="floor_id"/>
    <result column="create_by" property="createBy"/>
    <result column="update_by" property="updateBy"/>
    <result column="remark" property="remark"/>
    <result column="create_time" property="createTime"/>
    <result column="update_time" property="updateTime"/>
    <result column="price" property="price"/>
    <collection property="bedVoList" ofType="com.zzyl.vo.BedVo">
    <id column="bid" property="id"/>
    <result column="bed_number" property="bedNumber"/>
    <result column="bed_status" property="bedStatus"/>
    <result column="room_id" property="roomId"/>
    <result column="lname" property="lname"/>
    <result column="ename" property="name"/>
    <result column="eid" property="elderId"/>
    </collection>
    </collection>
    </resultMap>

<select id="getRoomAndBedByBedStatus" resultMap="floorResultMapWithRoomAndBed" parameterType="java.lang.Integer">
    select f.name,
           f.id,
           r.id as rid,
           r.code,
           r.sort,
           r.type_name,
           r.floor_id,
           b.id as bid,
           b.bed_number,
           b.sort,
           b.bed_status,
           b.room_id,
           b.create_by,
           b.update_by,
           b.remark,
           rt.price,
           b.create_time,
           b.update_time
    from floor f
             left join room r on r.floor_id = f.id
             left join bed b on b.room_id = r.id
             left join room_type rt on rt.name = r.type_name
    where b.bed_status = #{status}
    order by f.id,r.id,b.id
</select>
```



#### 4.2.6 测试

打开申请入住，查看入住床位，当选择床位后是否可以自动给**床位价格**赋值

![image-20240724213218027](assets/image-20240724213218027.png)



### 4.3 申请入住

#### 4.3.1 接口说明


**接口地址**:`/check-in/apply`

**请求方式**:`POST`


**请求示例**:


```json
{
  "checkInElderDto": {//基本信息
    "address": "北京市昌平区回龙观新龙城",//家庭住址
    "age": "68",//年龄
    "birthday": "1956-01-02",//出生日期
    "idCardNationalEmblemImg": "https://itheim.oss-cn-beijing.aliyuncs.com/b6e1fb36-d752-4831-93c3-139621ec60e1.jpg",
    "idCardNo": "132143195601021234",//身份证号
    "idCardPortraitImg": "https://itheim.oss-cn-beijing.aliyuncs.com/72e25c86-c335-462c-85d2-3f1d8c056409.jpg",
    "name": "李山",//姓名
    "oneInchPhoto": "https://itheim.oss-cn-beijing.aliyuncs.com/81c10ed6-12c1-4e94-ad2e-6404938e091a.png",
    "phone": "13567890987",//手机号
    "sex": 1//性别  男1   女0
  },
  "elderFamilyDtoList": [//家属信息
    {
      "kinship": "0",//亲属关系
      "name": "李佳佳",//姓名
      "phone": "13212345432"//联系方式
    }
  ],
  "checkInConfigDto": {//入住配置
    "bedCost": 6000,//床位费用
    "bedId": "62",//床位Id
    "checkInEndTime": "2024-05-31 00:00:00",//入住结束时间
    "checkInStartTime": "2024-04-20 00:00:00",//入住开始时间
    "code": "606",//房间编号
    "costEndTime": "2024-04-30 00:00:00",//费用结束时间
    "costStartTime": "2024-04-20 00:00:00",//费用开始时间
    "depositAmount": 3000,//押金金额
    "floorId": 6,//楼层id
    "floorName": "6楼",//楼层名称
    "governmentSubsidy": 0,//政府补贴
    "medicalInsurancePayment": 0,//医保支付
    "nursingCost": 2000,//护理费用
    "nursingLevelId": 2,//护理等级ID
    "nursingLevelName": "一级护理等级",//护理等级名称
    "otherCost": 0,//其他费用
    "roomId": 42//房间ID
  },
  "checkInContractDto": {//签约办理
    "memberName": "李佳佳",//丙方名称
    "memberPhone": "13212345432",//丙方手机号
    "name": "李山的入住合同",//合同名称
    "pdfUrl": "https://itheim.oss-cn-beijing.aliyuncs.com/cdb9c12f-2ba0-4d83-a57d-c7f028b87ab9.pdf",
    "signDate": "2024-04-19 16:31:07"//签约时间
  }
}
```


**响应示例**:
```javascript
{
	"code": 0,
	"msg": "",
	"operationTime": ""
}
```

#### 4.3.2 思路说明

申请入住的流程较长，涉及到多表中的操作，流程如下：

<img src="assets/申请入住流程.png" alt="申请入住流程" style="zoom:80%;" />

#### 4.3.3 控制层代码

在CheckInController中定义申请入住的方法

```java
@PostMapping("/apply")
public ResponseResult apply(@RequestBody CheckInApplyDto dto){
    checkInService.apply(dto);
    return ResponseResult.success();
}
```



#### 4.3.4 service业务层

在CheckInService中定义申请入住的方法

```java
/**
 * 申请入住
 * @param dto
 */
void apply(CheckInApplyDto dto);
```

在CheckInServiceImpl中定义实现的方法

```java
@Autowired
private ElderMapper elderMapper;

@Autowired
private BedMapper bedMapper;

@Autowired
private RedisTemplate<String,String> redisTemplate;

/**
 * 申请入住
 * @param dto
 */
@Override
public void apply(CheckInApplyDto dto) {

    //1.检查老人是否已经入住
    Elder elderDb = elderMapper.selectByIdCardAndStatus(dto.getCheckInElderDto().getIdCardNo(),1);
    if(elderDb != null){
        throw new BaseException(BasicEnum.ELDER_ALREADY_CHECKIN);
    }

    //2.更新床位状态
    bedMapper.updateBedStatusById(dto.getCheckInConfigDto().getBedId(),1);

    //3.新增或更新老人信息
    Bed bed = bedMapper.getBedById(dto.getCheckInConfigDto().getBedId());
    Elder elder = insertOrUpdateElder(dto, bed);

    //生成入住编码
    String checkInCode = CodeUtil.generateCode("RZ", redisTemplate, 5);
    //生成合同编码
    String contractNo = CodeUtil.generateCode("HT", redisTemplate, 5);

    //4.新增合同
    insertContract(dto,elder,checkInCode,contractNo);

    //5.新增入住信息
    insertCheckIn(dto,elder,checkInCode,contractNo);

    //6.新增入住配置
    inserCheckInConfig(dto,bed.getBedNumber(),elder.getId(),checkInCode);

}

@Autowired
private CheckInConfigMapper checkInConfigMapper;

/**
 * 保存入住配置
 * @param dto
 * @param bedNumber
 * @param elderId
 * @param checkInCode
 */
private void inserCheckInConfig(CheckInApplyDto dto, String bedNumber, Long elderId, String checkInCode) {
    CheckInConfig checkInConfig = BeanUtil.toBean(dto.getCheckInConfigDto(), CheckInConfig.class);
    checkInConfig.setElderId(elderId);
    checkInConfig.setCheckInCode(checkInCode);
    checkInConfig.setBedNumber(bedNumber);
    //保存在remark中的一个规范----> 楼层id:房间ID:床位ID:楼层名称:房间编号
    CheckInConfigDto checkInConfigDto = dto.getCheckInConfigDto();
    String remark = checkInConfigDto.getFloorId()+":"+checkInConfigDto.getRoomId()+":"+checkInConfigDto.getBedId()+":"+checkInConfigDto.getFloorName()+":"+checkInConfigDto.getCode();
    checkInConfig.setRemark(remark);
    checkInConfigMapper.insert(checkInConfig);

}

/**
 * 新增入住信息
 * @param dto
 * @param elder
 * @param checkInCode
 * @param contractNo
 */
private void insertCheckIn(CheckInApplyDto dto, Elder elder, String checkInCode, String contractNo) {

    //获取当前登录人信息
    String subject = UserThreadLocal.getSubject();
    User user = JSONUtil.toBean(subject, User.class);

    //没办法进行拷贝
    CheckIn checkIn = new CheckIn();
    checkIn.setCheckInCode(checkInCode);
    checkIn.setElderId(elder.getId());
    checkIn.setTitle(elder.getName()+"的入住申请");
    checkIn.setCheckInTime(LocalDateTime.now());
    checkIn.setRemark(contractNo);
    checkIn.setStatus(0);
    checkIn.setOtherApplyInfo(JSONUtil.toJsonStr(dto.getElderFamilyDtoList()));
    checkIn.setApplicat(user.getRealName());
    checkIn.setApplicatId(user.getId());
    checkIn.setDeptNo(user.getDeptNo());
    checkInMapper.insert(checkIn);

}

@Autowired
private ContractMapper contractMapper;

/**
 * 新增入住合同
 * @param dto
 * @param elder
 * @param checkInCode
 * @param contractNo
 */
private void insertContract(CheckInApplyDto dto, Elder elder, String checkInCode, String contractNo) {

    Contract contract = BeanUtil.toBean(dto.getCheckInContractDto(), Contract.class);
    contract.setContractNo(contractNo);
    contract.setCheckInNo(checkInCode);
    contract.setElderId(elder.getId());
    contract.setElderName(elder.getName());
    contract.setStartTime(dto.getCheckInConfigDto().getCheckInStartTime());
    contract.setEndTime(dto.getCheckInConfigDto().getCheckInEndTime());
    contract.setStatus(1);
    contractMapper.insert(contract);
}

/**
 * 新增或修改老人
 * @param dto
 * @param bed
 */
private Elder insertOrUpdateElder(CheckInApplyDto dto, Bed bed) {

    Elder elder = BeanUtil.toBean(dto.getCheckInElderDto(), Elder.class);
    elder.setImage(dto.getCheckInElderDto().getOneInchPhoto());
    elder.setStatus(1);
    elder.setBedId(bed.getId());
    elder.setBedNumber(bed.getBedNumber());
    elder.setPhone(dto.getCheckInElderDto().getPhone());

    //根据身份证号查询退住状态的老人
    Elder elderDb = elderMapper.selectByIdCardAndStatus(elder.getIdCardNo(), 0);
    if(elderDb != null){
        elder.setId(elderDb.getId());
        elderMapper.updateByPrimaryKeySelective(elder);
    }else{
        elderMapper.insert(elder);
    }

    return elder;

}
```



#### 4.3.5 mapper持久层

大部分的mapper我们都已经定义好了，不过还需要两个mapper查询

（1）在ElderMapper中定义根据身份证号和状态查询的方法

```java
@Select("select * from elder where id_card_no = #{idCardNo} and status = #{status}")
Elder selectByIdCardAndStatus(String idCardNo, int status);
```

（2）在BedMapper中定义根据id修改状态的方法

```java
@Update("UPDATE bed SET bed_status = #{status} WHERE id = #{bedId}")
void updateBedStatusById(Long bedId, int status);
```



#### 4.3.6 测试

当我们输入了入住数据之后，查看是否能正常保存数据



### 4.4 入住详情

#### 4.4.1 接口说明


**接口地址**:`/check-in/detail/{id}`

**请求方式**:`GET`


**请求参数**:


| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型       |
| -------- | -------- | -------- | -------- | -------------- |
| id       | 入住id   | path     | true     | integer(int64) |

**响应示例**:

```javascript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "checkInElderVo": {
            "id": "2",
            "name": "刘拥军",
            "idCardNo": "132123195612191234",
            "birthday": "1956-12-19",
            "sex": "0",
            "phone": "15533220099",
            "address": "北京市西城区",
            "oneInchPhoto": "https://itheim.oss-cn-beijing.aliyuncs.com/d6bb9f5f-0fd2-432a-9b6c-7e9879fbd39a.png",
            "idCardNationalEmblemImg": "https://itheim.oss-cn-beijing.aliyuncs.com/4b7398a3-61f1-4a37-b410-e09a20d6d778.jpg",
            "idCardPortraitImg": "https://itheim.oss-cn-beijing.aliyuncs.com/0b4fd72b-1e09-4669-b596-8358fb833d35.jpg"
        },
        "elderFamilyVoList": [
            {
                "name": "刘小军",
                "phone": "13212340987",
                "kinship": "子女"
            }
        ],
        "checkInConfigVo": {
            "id": "2",
            "createTime": "2024-07-21 21:55:57",
            "updateTime": "2024-07-21 21:55:57",
            "createBy": "1671403256519078138",
            "remark": "404:87:202:2楼:202",
            "elderId": "2",
            "checkInStartTime": "2024-07-21 00:00:00",
            "checkInEndTime": "2024-08-31 23:59:59",
            "nursingLevelId": "4",
            "nursingLevelName": "三级护理等级",
            "bedNumber": "202-2",
            "costStartTime": "2024-07-21 00:00:00",
            "costEndTime": "2024-07-31 23:59:59",
            "depositAmount": 3000.00,
            "nursingCost": 1000.00,
            "bedCost": 3500.00,
            "otherCost": 0.00,
            "medicalInsurancePayment": 0.00,
            "governmentSubsidy": 0.00,
        },
        "checkInContractVo": {
            "id": "2",
            "name": "刘拥军的入住合同",
            "signDate": "2024-07-21 21:54:09",
            "memberName": "刘小军",
            "memberPhone": "13212340987",
            "pdfUrl": "https://itheim.oss-cn-beijing.aliyuncs.com/7057106f-86a5-4498-8ae6-1035b522ffbe.png"
        }
    },
    "operationTime": null
}
```

#### 4.4.2 思路说明

查看详情数据较多，需要从多个表中整合数据，整体实现思路如下：

<img src="assets/image-20240724233832858.png" alt="image-20240724233832858" style="zoom:80%;" />



#### 4.4.3 控制层代码

在CheckInController中定义新的方法，如下：

```java
@GetMapping("/detail/{id}")
public ResponseResult<CheckInDetailVo> detail(@PathVariable Long id) {
    return ResponseResult.success(checkInService.detail(id));
}
```



#### 4.4.4 service业务层

在CheckInService中定义查询详情的方法：

```java
/**
 * 入住详情
 *
 * @param id 入住id
 * @return 入住详情
 */
CheckInDetailVo detail(Long id);
```

在CheckInServiceImpl中定义实现的方法，逻辑如下：

```java
/**
 * 入住详情
 * @param id 入住id
 * @return 入住详情
 */
@Override
public CheckInDetailVo detail(Long id) {

    CheckInDetailVo vo = new CheckInDetailVo();

    //查询入住信息
    CheckIn checkIn = checkInMapper.selectById(id);

    //查询老人
    Elder elder = elderMapper.selectByPrimaryKey(checkIn.getElderId());
    CheckInElderVo checkInElderVo = BeanUtil.toBean(elder, CheckInElderVo.class);
    checkInElderVo.setOneInchPhoto(elder.getImage());
    vo.setCheckInElderVo(checkInElderVo);

    //查询入住配置
    CheckInConfigVo checkInConfigVo = checkInConfigMapper.selectByElderId(checkIn.getElderId());
    vo.setCheckInConfigVo(checkInConfigVo);

    //合同信息
    CheckInContractVo checkInContractVo = contractMapper.selectByElder(checkIn.getElderId());
    vo.setCheckInContractVo(checkInContractVo);
    //家属信息
    vo.setElderFamilyVoList(JSONUtil.toList(checkIn.getOtherApplyInfo(), ElderFamilyVo.class));
    return vo;
}
```



#### 4.4.5 mapper持久层

我们需要定义两个mapper接口，查询方法需要定义两个

（1）在CheckInConfigMapper中定义查询的方法

```java
@Select("select * from check_in_config where elder_id = #{elderId}")
CheckInConfigVo selectByElderId(Long elderId);
```

（2）在ContractMapper中定义查询的方法

```
@Select("select * from contract where elder_id = #{elderId}")
CheckInContractVo selectByElder(Long elderId);
```



#### 4.4.6 测试

点击**查看**按钮，是否可以入住的详细数据

![](assets/入住详情.png)







