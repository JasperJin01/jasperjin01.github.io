# 订单模块

# 1 结算

## 1.1 需求说明

入口：购物车点击去结算按钮 ，进入结算页面(订单确认页面)，如图所示：

![trade](assets/trade.gif) 

分析页面需要的数据：

1、 用户地址信息列表，结算页选中默认地址

2、 购物车中选择的商品列表，及商品的总金额



**查看接口文档：**

获取用户地址信息列表接口地址及返回结果：

```json
get /api/user/userAddress/auth/findUserAddressList
返回结果：
{
    "code": 200,
    "message": "操作成功",
    "data": [
        {
            "id": 1,
            "createTime": "2023-05-12 17:50:41",
            "updateTime": "2023-06-02 19:15:17",
            "isDeleted": 0,
            "userId": 1,
            "name": "晴天",
            "phone": "15014526352",
            "tagName": "家",
            "provinceCode": "110000",
            "cityCode": "110100",
            "districtCode": "110114",
            "address": "天通苑大街1号",
            "fullAddress": "北京市北京市昌平区天通苑大街1号",
            "isDefault": 0
        },
        ...
    ]
}
```

结算接口地址及返回结果：

```json
get api/order/orderInfo/auth/trade
返回结果：
{
    "code": 200,
    "message": "操作成功",
    "data": {
        "totalAmount": 6997.00,
        "orderItemList": [
            {
                "skuId": 1,
                "skuName": "小米 红米Note10 5G手机 颜色:白色 内存:8G",
                "thumbImg": "http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1 (1).jpg",
                "skuPrice": 1999.00,
                "skuNum": 2
            },
            ...
        ]
    }
} 
```

获取用户地址列表调用流程如下所示：

![image-20230709114029869](assets/image-20230709114029869.png) 



获取购物车中选中的商品列表调用流程如下所示：

![image-20230709114143558](assets/image-20230709114143558.png) 



## 1.2 获取用户地址

在service-user模块中开发一个获取地址列表接口，供service-order微服务进行调用。

### 1.2.1 UserAddress

针对当前要操作的数据定义一个与之对应的实体类：

```java
//com.atguigu.spzx.model.entity.user;
@Data
@Schema(description = "用户地址实体类")
public class UserAddress extends BaseEntity {

   private static final long serialVersionUID = 1L;

   @Schema(description = "用户ID")
   private Long userId;

   @Schema(description = "name")
   private String name;

   @Schema(description = "电话")
   private String phone;

   @Schema(description = "标签名称")
   private String tagName;

   @Schema(description = "provinceCode")
   private String provinceCode;

   @Schema(description = "cityCode")
   private String cityCode;

   @Schema(description = "districtCode")
   private String districtCode;

   @Schema(description = "详细地址")
   private String address;

   @Schema(description = "完整地址")
   private String fullAddress;

   @Schema(description = "是否默认地址（0：否 1：是）")
   private Integer isDefault;

}
```

### 1.2.2 UserAddressController

表现层代码：

```java
// com.atguigu.spzx.user.controller;
@Tag(name = "用户地址接口")
@RestController
@RequestMapping(value="/api/user/userAddress")
@SuppressWarnings({"unchecked", "rawtypes"})
public class UserAddressController {
   
   @Autowired
   private UserAddressService userAddressService;

   @Operation(summary = "获取用户地址列表")
   @GetMapping("auth/findUserAddressList")
   public Result<List<UserAddress>> findUserAddressList() {
      List<UserAddress> list = userAddressService.findUserAddressList();
      return Result.build(list , ResultCodeEnum.SUCCESS) ;
   }
}
```

### 1.2.3 UserAddressService

业务层代码实现：

```java
//业务接口
public interface UserAddressService {

    List<UserAddress> findUserAddressList();
}

//业务接口实现
@Service
@SuppressWarnings({"unchecked", "rawtypes"})
public class UserAddressServiceImpl implements UserAddressService {

   @Autowired
   private UserAddressMapper userAddressMapper;

   @Override
   public List<UserAddress> findUserAddressList() {
      Long userId = AuthContextUtil.getUserInfo().getId();
      return userAddressMapper.findByUserId(userId);
   }
}
```

### 1.2.4 UserAddressMapper

持久层代码实现：

```java
@Mapper
public interface UserAddressMapper {

    List<UserAddress> findByUserId(Long userId);
}
```

### 1.2.5 UserAddressMapper.xml

在映射文件中定义对应的sql语句：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">


<mapper namespace="com.atguigu.spzx.user.mapper.UserAddressMapper">

   <resultMap id="userAddressMap" type="com.atguigu.spzx.model.entity.user.UserAddress" autoMapping="true">
   </resultMap>
   
   <!-- 用于select查询公用抽取的列 -->
   <sql id="columns">
      id,user_id,name,phone,tag_name,province_code,city_code,district_code,address,full_address,is_default,create_time,update_time,is_deleted
   </sql>

   <select id="findByUserId" resultMap="userAddressMap">
      select <include refid="columns" />
      from user_address
      where
      user_id = #{userId}
   </select>
</mapper>
```

启动服务进行测试。

## 1.3 获取购物项数据

在service-cart微服务中开发一个接口，获取当前登录用户选中的购物项列表数据，如下所示：

### 1.3.1 CartController

表现层接口：

```java
@Operation(summary="选中的购物车")
@GetMapping(value = "/auth/getAllCkecked")
public Result<List<CartInfo>> getAllCkecked() {
    List<CartInfo> cartInfoList = cartService.getAllCkecked() ;
    return Result.build(cartInfoList , ResultCodeEnum.SUCCESS) ;
}
```

### 1.3.2 CartService

业务层接口：

```java
// 业务接口
List<CartInfo> getAllCkecked();

// 业务接口实现类
@Override
public List<CartInfo> getAllCkecked() {
    Long userId = AuthContextUtil.getUserInfo().getId();
    String cartKey = getCartKey(userId);
    List<Object> objectList = redisTemplate.opsForHash().values(cartKey);       // 获取所有的购物项数据
    if(!CollectionUtils.isEmpty(objectList)) {
        List<CartInfo> cartInfoList = objectList.stream().map(cartInfoJSON -> JSON.parseObject(cartInfoJSON.toString(), CartInfo.class))
            .filter(cartInfo -> cartInfo.getIsChecked() == 1)
            .collect(Collectors.toList());
        return cartInfoList ;
    }
    return new ArrayList<>() ;
}
```

### 1.3.3 openFeign接口定义

步骤：

1、在spzx-service-client模块下创建一个service-cart-client的子模块

2、在service-cart-client模块下定义远程openFeign接口

```java
// com.atguigu.spzx.feign.cart;
@FeignClient(value = "service-cart" , fallback = CartFeignClientFallback.class)
public interface CartFeignClient {

    @GetMapping(value = "/api/order/cart/auth/getAllCkecked")
    public abstract Result<List<CartInfo>> getAllCkecked() ;

}


// com.atguigu.spzx.feign.cart.fallback;
@Slf4j
public class CartFeignClientFallback implements CartFeignClient {

    @Override
    public Result<List<CartInfo>> getAllCkecked() {
        log.info("CartFeignClientFallback...getAllCkecked的方法执行了");
        return Result.build(null , ResultCodeEnum.SUCCESS) ;
    }
}

```

3、降级类自动化配置

在resources目录下创建一个MATE-INF/spring文件夹，在该文件夹下创建一个

org.springframework.boot.autoconfigure.AutoConfiguration.imports文件，文件的中的内容如下所示：

```java
com.atguigu.spzx.feign.cart.fallback.CartFeignClientFallback
```

## 1.4 环境搭建(service-order)

步骤：

1、在spzx-service模块下创建一个service-order微服务,并加入如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2、准备application.yml、application-dev.yml、logback-spring.xml、mybatis-config.xml文件。文件内容如下所示：

```yml
server:
  port: 8514

spring:
  application:
    name: service-order
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.136.142:8848
    sentinel:
      transport:
        dashboard: localhost:8080
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.136.142:3306/db_spzx?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=true
    username: root
    password: 1234
  data:
    redis:
      host: 192.168.136.142
      port: 6379
      password: 1234

mybatis:
  config-location: classpath:mybatis-config.xml
  mapper-locations: classpath:mapper/*/*.xml
feign:
  sentinel:
    enabled: true
```

logback-spring.xml修改输出路径：

```xml
<property name="log.path" value="D://work//service-order//logs" />
```

mybatis-config.xml：从之前的模块中进行复制

3、创建启动类

```java
//  com.atguigu.spzx.order;
@SpringBootApplication
@EnableFeignClients(basePackages = {
        "com.atguigu.spzx.feign.cart"
})
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class , args) ;
    }

}
```

## 1.5 后端业务接口开发

操作模块：service-order

### 1.5.1 TradeVo

定义一个实体类用来封装结算的参数(根据接口文档进行定义)，具体定义如下所示

```java
// com.atguigu.spzx.model.vo.h5
@Data
@Schema(description = "结算实体类")
public class TradeVo {

    @Schema(description = "结算总金额")
    private BigDecimal totalAmount;

    @Schema(description = "结算商品列表")
    private List<OrderItem> orderItemList;

}
```

### 1.5.2 OrderItem

定义一个与数据库表相对应的实体类，封装订单明细数据：

```java
 // com.atguigu.spzx.model.entity.order
@Data
@Schema(description = "订单项实体类")
public class OrderItem extends BaseEntity {

   private static final long serialVersionUID = 1L;

   @Schema(description = "订单id")
   private Long orderId;

   @Schema(description = "商品sku编号")
   private Long skuId;

   @Schema(description = "商品sku名字")
   private String skuName;

   @Schema(description = "商品sku图片")
   private String thumbImg;

   @Schema(description = "商品sku价格")
   private BigDecimal skuPrice;

   @Schema(description = "商品购买的数量")
   private Integer skuNum;

}
```

### 1.5.3 OrderInfoController

业务层代码实现：

```java
@Tag(name = "订单管理")
@RestController
@RequestMapping(value="/api/order/orderInfo")
@SuppressWarnings({"unchecked", "rawtypes"})
public class OrderInfoController {
   
   @Autowired
   private OrderInfoService orderInfoService;

   @Operation(summary = "确认下单")
   @GetMapping("auth/trade")
   public Result<TradeVo> trade() {
      TradeVo tradeVo = orderInfoService.getTrade();
      return Result.build(tradeVo, ResultCodeEnum.SUCCESS);
   }

}
```

### 1.5.4 OrderInfoService

业务层代码实现：

```java
//业务接口
public interface OrderInfoService {
    TradeVo getTrade();
}

//业务接口实现
// com.atguigu.spzx.order.service.impl;
@Service
public class OrderInfoServiceImpl implements OrderInfoService {

    @Autowired
    private CartFeignClient cartFeignClient ;

    @Override
    public TradeVo getTrade() {

        // 获取当前登录的用户的id
        Long userId = AuthContextUtil.getUserInfo().getId();

        // 获取选中的购物项列表数据
        List<CartInfo> cartInfoList = cartFeignClient.getAllCkecked(userId).getData() ;
        List<OrderItem> orderItemList = new ArrayList<>();
        for (CartInfo cartInfo : cartInfoList) {        // 将购物项数据转换成功订单明细数据
            OrderItem orderItem = new OrderItem();
            orderItem.setSkuId(cartInfo.getSkuId());
            orderItem.setSkuName(cartInfo.getSkuName());
            orderItem.setSkuNum(cartInfo.getSkuNum());
            orderItem.setSkuPrice(cartInfo.getCartPrice());
            orderItem.setThumbImg(cartInfo.getImgUrl());
            orderItemList.add(orderItem);
        }

        // 计算总金额
        BigDecimal totalAmount = new BigDecimal(0);
        for(OrderItem orderItem : orderItemList) {
            totalAmount = totalAmount.add(orderItem.getSkuPrice().multiply(new BigDecimal(orderItem.getSkuNum())));
        }
        TradeVo tradeVo = new TradeVo();
        tradeVo.setTotalAmount(totalAmount);
        tradeVo.setOrderItemList(orderItemList);
        return tradeVo;
        
    }

}
```

### 1.5.5 服务网关

服务网关中配置service-order微服务的路由规则：

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: service-order
          uri: lb://service-order
          predicates:
            - Path=/api/order/orderInfo/**
```

启动服务进行测试。

## 1.6 openFeign拦截器使用

### 1.6.1 问题说明

在测试的时候，那么service-cart微服务会报错，如下所示：

```java
java.lang.NullPointerException: Cannot invoke "com.atguigu.spzx.model.entity.user.UserInfo.getId()" because the return value of "com.atguigu.spzx.common.util.AuthContextUtil.getUserInfo()" is null
	at com.atguigu.spzx.cart.service.impl.CartServiceImpl.getAllCkecked(CartServiceImpl.java:147)
```

针对service-cart微服务是获取不到当前登录用户的信息。

原因：service-order微服务调用service-cart微服务的时候，是通过openFeign进行调用，openFeign在调用的时候会丢失请求头

### 1.6.2 问题解决

解决方案：使用feign拦截器拦截请求，获取token，重新传递token

![image-20230709121255146](assets/image-20230709121255146.png) 

注意：为了让feign拦截器更加通用，那么此时可以将拦截器定义到common-service模块中



feign拦截器开发：

1、在common-service模块中加入openFeign的依赖

```xml
<!-- openfeign依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <scope>provided</scope>
</dependency>
```

2、定义feign拦截器

```java
// com.atguigu.spzx.common.feign;
public class UserTokenFeignInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate requestTemplate) {
        ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = requestAttributes.getRequest();
        String token = request.getHeader("token");
        requestTemplate.header("token" , token) ;
    }

}
```

3、定义使用该拦截器的注解

```java
// com.atguigu.spzx.common.anno;
@Retention(value = RetentionPolicy.RUNTIME)
@Target(value = ElementType.TYPE)
@Import(value = UserTokenFeignInterceptor.class)
public @interface EnableUserTokenFeignInterceptor {
    
}
```

4、service-order微服务启动类上使用**@EnableUserTokenFeignInterceptor**注解

# 2 下单

## 2.1 需求说明

需求说明：用户在结算页面点击提交订单按钮，那么此时就需要保存订单信息(order_info)、订单项信息(order_item)及记录订单日志(order_log)，

下单成功重定向到订单支付页面

**查看接口文档：**

下单接口地址及返回结果：

```json
post /api/order/orderInfo/auth/submitOrder
参数：
{
    "orderItemList": [
        {
            "skuId": 6,
            "skuName": "小米 红米Note10 5G手机 颜色:黑色 内存:18G",
            "thumbImg": "http://139.198.127.41:9000/spzx/20230525/665832167-1_u_1.jpg",
            "skuPrice": 2999,
            "skuNum": 1
        },
        ...
    ],
    "userAddressId": 2,
    "feightFee": 0,
    "remark": "赶快发货"
}
返回结果(订单id)：
{
    "code": 200,
    "message": "操作成功",
    "data": 1
}
```

调用流程如下所示：

![image-20230709133513669](assets/image-20230709133513669.png) 

## 2.2 后端接口

### 2.2.1 查询用户地址

#### 业务接口开发

操作模块：service-user

##### UserAddressController

表现层代码：

```java
@Operation(summary = "获取地址信息")
@GetMapping("getUserAddress/{id}")
public UserAddress getUserAddress(@PathVariable Long id) {
   return userAddressService.getById(id);
}
```

##### UserAddressService

业务层代码实现：

```java
//业务接口
UserAddress getById(Long id);

//业务接口实现
@Override
public UserAddress getById(Long id) {
   return userAddressMapper.getById(id);
}
```

##### UserAddressMapper

持久层代码实现：

```java
UserAddress getById(Long id);
```

##### UserAddressMapper.xml

在映射文件中定义对应的sql语句：

```xml
<select id="getById" resultMap="userAddressMap">
   select <include refid="columns" />
   from user_address
   where
   id = #{id}
</select>
```

启动程序进行测试。

#### openFeign接口开发

步骤如下：

步骤：

1、在spzx-service-client模块下创建一个service-user-client的子模块

2、在service-user-client模块下定义远程openFeign接口

```java
// com.atguigu.spzx.feign.user;
@FeignClient(value = "service-user" , fallback = UserFeignClientFallback.class)
public interface UserFeignClient {

    @GetMapping("/api/user/userAddress/getUserAddress/{id}")
    public abstract Result<UserAddress> getUserAddress(@PathVariable Long id) ;

}


// com.atguigu.spzx.feign.user.fallback;
@Slf4j
public class UserFeignClientFallback implements UserFeignClient {

    @Override
    public Result<UserAddress> getUserAddress(Long id) {
        log.info("UserFeignClientFallback...getUserAddress的方法执行了");
        return Result.build(null , ResultCodeEnum.SUCCESS) ;
    }

}
```

3、降级类自动化配置

在resources目录下创建一个MATE-INF/spring文件夹，在该文件夹下创建一个

org.springframework.boot.autoconfigure.AutoConfiguration.imports文件，文件的中的内容如下所示：

```java
com.atguigu.spzx.feign.user.fallback.UserFeignClientFallback
```

4、修改service-order微服务

pom.xml文件中加入依赖：

```xml
<dependency>
    <groupId>com.atguigu.spzx</groupId>
    <artifactId>service-user-client</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

启动类修改：

```java
@EnableFeignClients(basePackages = {
        "com.atguigu.spzx.feign.cart",
        "com.atguigu.spzx.feign.user"
})
```

### 2.2.2 业务接口开发

操作模块：service-order

#### OrderInfoDto

定义一个实体类用来封装下单的参数，具体定义如下所示

```java
@Data
public class OrderInfoDto {

    //送货地址id
    private Long userAddressId;

    //运费
    private BigDecimal feightFee;
    //备注
    private String remark;
    //订单明细
    private List<OrderItem> orderItemList;
}
```

#### OrderLog

订单日志体类定义：

```java
@Data
@Schema(description = "订单日志实体对象")
public class OrderLog extends BaseEntity {

   private static final long serialVersionUID = 1L;

   @Schema(description = "订单id")
   private Long orderId;

   @Schema(description = "操作人：用户；系统；后台管理员")
   private String operateUser;

   @Schema(description = "订单状态")
   private Integer processStatus;

   @Schema(description = "备注")
   private String note;

}
```

#### OrderInfoController

表现层代码：

```java
@Operation(summary = "提交订单")
@PostMapping("auth/submitOrder")
public Result<Long> submitOrder(@Parameter(name = "orderInfoDto", description = "请求参数实体类", required = true) @RequestBody OrderInfoDto orderInfoDto) {
   Long orderId = orderInfoService.submitOrder(orderInfoDto);
   return Result.build(orderId, ResultCodeEnum.SUCCESS);
}
```

#### OrderInfoService

业务层代码实现：

```java
//业务接口
Long submitOrder(OrderInfoDto orderInfoDto);

//业务接口实现
@Transactional
@Override
public Long submitOrder(OrderInfoDto orderInfoDto) {

    // 数据校验
    List<OrderItem> orderItemList = orderInfoDto.getOrderItemList();
    if (CollectionUtils.isEmpty(orderItemList)) {
        throw new GuiguException(ResultCodeEnum.DATA_ERROR);
    }
    for (OrderItem orderItem : orderItemList) {
        ProductSku productSku = productFeignClient.getBySkuId(orderItem.getSkuId()).getData();
        if(null == productSku) {
            throw new GuiguException(ResultCodeEnum.DATA_ERROR);
        }
        if(orderItem.getSkuNum().intValue() > productSku.getStockNum().intValue()) {
            throw new GuiguException(ResultCodeEnum.STOCK_LESS);
        }
    }

    // 构建订单数据，保存订单
    UserInfo userInfo = AuthContextUtil.getUserInfo();
    OrderInfo orderInfo = new OrderInfo();
    orderInfo.setOrderNo(String.valueOf(System.currentTimeMillis()));
    orderInfo.setUserId(userInfo.getId());
    orderInfo.setNickName(userInfo.getUsername());

    UserAddress userAddress = userFeignClient.getUserAddress(orderInfoDto.getUserAddressId()).getData();
    orderInfo.setReceiverName(userAddress.getName());
    orderInfo.setReceiverPhone(userAddress.getPhone());
    orderInfo.setReceiverTagName(userAddress.getTagName());
    orderInfo.setReceiverProvince(userAddress.getProvinceCode());
    orderInfo.setReceiverCity(userAddress.getCityCode());
    orderInfo.setReceiverDistrict(userAddress.getDistrictCode());
    orderInfo.setReceiverAddress(userAddress.getFullAddress());

    BigDecimal totalAmount = new BigDecimal(0);
    for (OrderItem orderItem : orderItemList) {
        totalAmount = totalAmount.add(orderItem.getSkuPrice().multiply(new BigDecimal(orderItem.getSkuNum())));
    }
    orderInfo.setTotalAmount(totalAmount);
    orderInfo.setCouponAmount(new BigDecimal(0));
    orderInfo.setOriginalTotalAmount(totalAmount);
    orderInfo.setFeightFee(orderInfoDto.getFeightFee());
    orderInfo.setPayType(2);
    orderInfo.setOrderStatus(0);
    orderInfoMapper.save(orderInfo);

    //保存订单明细
    for (OrderItem orderItem : orderItemList) {
        orderItem.setOrderId(orderInfo.getId());
        orderItemMapper.save(orderItem);
    }

    //记录日志
    OrderLog orderLog = new OrderLog();
    orderLog.setOrderId(orderInfo.getId());
    orderLog.setProcessStatus(0);
    orderLog.setNote("提交订单");
    orderLogMapper.save(orderLog);
    
    // TODO 远程调用service-cart微服务接口清空购物车数据
    return orderInfo.getId();
}
```

在ResultCodeEnum类添加枚举

```java
STOCK_LESS( 219, "库存不足"),
```

#### Mapper接口

持久层代码实现：

##### OrderInfoMapper

```java
@Mapper
public interface OrderInfoMapper {

    void save(OrderInfo orderInfo);
}
```

##### OrderItemMapper

```java
@Mapper
public interface OrderItemMapper {
    void save(OrderItem orderItem);
}
```

##### OrderLogMapper

```java
@Mapper
public interface OrderLogMapper {
    void save(OrderLog orderLog);
}
```

#### XML映射文件

在映射文件中定义对应的sql语句

##### OrderInfoMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.atguigu.spzx.order.mapper.OrderInfoMapper">

   <resultMap id="orderInfoMap" type="com.atguigu.spzx.model.entity.order.OrderInfo" autoMapping="true">
   </resultMap>
   
   <!-- 用于select查询公用抽取的列 -->
   <sql id="columns">
      id,user_id,nick_name,order_no,coupon_id,total_amount,coupon_amount,original_total_amount,feight_fee,pay_type,order_status,receiver_name,receiver_phone,receiver_tag_name,receiver_province,receiver_city,receiver_district,receiver_address,payment_time,delivery_time,receive_time,remark,cancel_time,cancel_reason,create_time,update_time,is_deleted
   </sql>

   <insert id="save" useGeneratedKeys="true" keyProperty="id">
      insert into order_info (
         id,
         user_id,
         nick_name,
         order_no,
         coupon_id,
         total_amount,
         coupon_amount,
         original_total_amount,
         feight_fee,
         pay_type,
         order_status,
         receiver_name,
         receiver_phone,
         receiver_province,
         receiver_city,
         receiver_district,
         receiver_address,
         payment_time,
         delivery_time,
         receive_time,
         remark,
         cancel_time,
         cancel_reason
      ) values (
         #{id},
         #{userId},
         #{nickName},
         #{orderNo},
         #{couponId},
         #{totalAmount},
         #{couponAmount},
         #{originalTotalAmount},
         #{feightFee},
         #{payType},
         #{orderStatus},
         #{receiverName},
         #{receiverPhone},
         #{receiverProvince},
         #{receiverCity},
         #{receiverDistrict},
         #{receiverAddress},
         #{paymentTime},
         #{deliveryTime},
         #{receiveTime},
         #{remark},
         #{cancelTime},
         #{cancelReason}
      )
   </insert>
</mapper>
```

##### OrderItemMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.atguigu.spzx.order.mapper.OrderItemMapper">

   <resultMap id="orderItemMap" type="com.atguigu.spzx.model.entity.order.OrderItem" autoMapping="true">
   </resultMap>
   
   <!-- 用于select查询公用抽取的列 -->
   <sql id="columns">
      id,order_id,sku_id,sku_name,thumb_img,sku_price,sku_num,create_time,update_time,is_deleted
   </sql>

   <insert id="save" useGeneratedKeys="true" keyProperty="id">
      insert into order_item (
         id,
         order_id,
         sku_id,
         sku_name,
         thumb_img,
         sku_price,
         sku_num
      ) values (
         #{id},
         #{orderId},
         #{skuId},
         #{skuName},
         #{thumbImg},
         #{skuPrice},
         #{skuNum}
      )
   </insert>
</mapper>
```

##### OrderLogMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.atguigu.spzx.order.mapper.OrderLogMapper">

   <resultMap id="orderLogMap" type="com.atguigu.spzx.model.entity.order.OrderLog" autoMapping="true">
   </resultMap>
   
   <!-- 用于select查询公用抽取的列 -->
   <sql id="columns">
      id,order_id,operate_user,process_status,note,create_time,update_time,is_deleted
   </sql>

   <insert id="save" useGeneratedKeys="true" keyProperty="id">
      insert into order_log (
         id,
         order_id,
         operate_user,
         process_status,
         note
      ) values (
         #{id},
         #{orderId},
         #{operateUser},
         #{processStatus},
         #{note}
      )
   </insert>

</mapper>
```

### 2.2.3 清空购物车

需求说明：订单生成成功以后，需要调用service-cart微服务的接口清空选中的购物车数据。

#### service-cart

在service-cart微服务中开发一个清空购物车的接口供service-order微服务进行调用：

##### CartController

```java
// com.atguigu.spzx.cart.controller.CartController
@GetMapping(value = "/auth/deleteChecked")
public Result deleteChecked() {
    cartService.deleteChecked() ;
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}
```

##### CartService

```java
// com.atguigu.spzx.cart.service.impl.CartServiceImpl
@Override
public void deleteChecked() {
    Long userId = AuthContextUtil.getUserInfo().getId();
    String cartKey = getCartKey(userId);
    List<Object> objectList = redisTemplate.opsForHash().values(cartKey);       // 删除选中的购物项数据
    if(!CollectionUtils.isEmpty(objectList)) {
        objectList.stream().map(cartInfoJSON -> JSON.parseObject(cartInfoJSON.toString(), CartInfo.class))
            .filter(cartInfo -> cartInfo.getIsChecked() == 1)
            .forEach(cartInfo -> redisTemplate.opsForHash().delete(cartKey , String.valueOf(cartInfo.getSkuId())));
    }
}
```

#### openFeign接口

在service-cart-client模块CartFeignClient中定义删除选中的购物车数据的远程接口方法：

```java
// com.atguigu.spzx.feign.cart
@FeignClient(value = "service-cart" , fallback = CartFeignClientFallback.class)
public interface CartFeignClient {
    
    @GetMapping(value = "/api/order/cart/auth/deleteChecked")
    public abstract Result deleteChecked() ;

}
```

#### service-order

改造service-order微服务的下单接口方法，添加清空选中购物车数据的远程调用代码：

```java
// com.atguigu.spzx.order.service.impl.OrderInfoServiceImpl
@Transactional
@Override
public Long submitOrder(OrderInfoDto orderInfoDto) {

    // 1、数据校验
    // 2、构建订单数据，保存订单
    // 3、保存订单明细
    // 4、记录日志
    // 5、清空购物车数据
    cartFeignClient.deleteChecked() ;
    
    return orderInfo.getId();
}
```

# 3 支付页

## 3.1 需求说明

提交订单成功，跳转到支付页面，根据订单id获取订单详细信息，展示订单支付信息

**查看接口文档：**

根据订单id获取订单信息接口地址及返回结果：

```json
get /api/order/orderInfo/auth/{orderId}
返回结果(订单id)：
{
    "code": 200,
    "message": "成功",
    "data": {
        "id": 194,
        "createTime": "2023-06-14 19:29:31",
        "userId": 1,
        "nickName": "test",
        "orderNo": "1686713371363",
        "couponId": null,
        "totalAmount": 2999,
        "couponAmount": 0,
        "originalTotalAmount": 2999,
        "feightFee": 0,
        "payType": 2,
        "orderStatus": 0,
        "receiverName": "张三",
        "receiverPhone": "15012563333",
        "receiverTagName": "公司",
        "receiverProvince": "130000",
        "receiverCity": "130700",
        "receiverDistrict": "130724",
        "receiverAddress": "河北省张家口市沽源县快乐家园1号",
        "paymentTime": null,
        "deliveryTime": null,
        "receiveTime": null,
        "remark": null,
        "cancelTime": null,
        "cancelReason": null
    }
}
```

## 3.2 后端接口

### 3.2.1 OrderInfoController

表现层代码：

```java
@Operation(summary = "获取订单信息")
@GetMapping("auth/{orderId}")
public Result<OrderInfo> getOrderInfo(@Parameter(name = "orderId", description = "订单id", required = true) @PathVariable Long orderId) {
   OrderInfo orderInfo = orderInfoService.getOrderInfo(orderId);
   return Result.build(orderInfo, ResultCodeEnum.SUCCESS);
}
```

### 3.2.2 OrderInfoService

业务层代码实现：

```java
//业务接口
OrderInfo getOrderInfo(Long orderId);

//业务接口实现
@Override
public OrderInfo getOrderInfo(Long orderId) {
    return orderInfoMapper.getById(orderId);
}
```

### 3.2.3 OrderInfoMapper

持久层代码实现：

```java
OrderInfo getById(Long orderId);
```

### 3.2.4 OrderInfoMapper.xml

在映射文件中定义对应的sql语句：

```xml
<select id="getById" resultMap="orderInfoMap">
   select <include refid="columns" />
   from order_info
   where
   id = #{id}
</select>
```

# 4 立即购买

## 4.1 需求说明

入口：商品详情页，点击“立即购买”按钮

立即购买直接进入结算页，不经过购物车，结算页返回数据与正常下单结算数据一致，提交订单接口不变，如图所示：

![buy](assets/buy.gif) 



**查看接口文档：**

立即购买接口地址及返回结果：

```json
get /api/order/orderInfo/auth/buy/{skuId}
返回结果：
{
    "code": 200,
    "message": "操作成功",
    "data": {
        "totalAmount": 6997.00,
        "orderItemList": [
            {
                "skuId": 1,
                "skuName": "小米 红米Note10 5G手机 颜色:白色 内存:8G",
                "thumbImg": "http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1 (1).jpg",
                "skuPrice": 1999.00,
                "skuNum": 2
            },
            ...
        ]
    }
}
```

## 4.2 后端接口

### 4.2.1 OrderInfoController

表现层代码：

```java
@Operation(summary = "立即购买")
@GetMapping("auth/buy/{skuId}")
public Result<TradeVo> buy(@Parameter(name = "skuId", description = "商品skuId", required = true) @PathVariable Long skuId) {
   TradeVo tradeVo = orderInfoService.buy(skuId);
   return Result.build(tradeVo, ResultCodeEnum.SUCCESS);
}
```

### 4.2.2 OrderInfoService

业务层代码实现：

```java
//业务接口
TradeVo buy(Long skuId);

//业务接口实现
@Override
public TradeVo buy(Long skuId) {

    // 查询商品
    ProductSku productSku = productFeignClient.getBySkuId(skuId).getData();
    List<OrderItem> orderItemList = new ArrayList<>();
    OrderItem orderItem = new OrderItem();
    orderItem.setSkuId(skuId);
    orderItem.setSkuName(productSku.getSkuName());
    orderItem.setSkuNum(1);
    orderItem.setSkuPrice(productSku.getSalePrice());
    orderItem.setThumbImg(productSku.getThumbImg());
    orderItemList.add(orderItem);

    // 计算总金额
    BigDecimal totalAmount = productSku.getSalePrice();
    TradeVo tradeVo = new TradeVo();
    tradeVo.setTotalAmount(totalAmount);
    tradeVo.setOrderItemList(orderItemList);

    // 返回
    return tradeVo;
}
```

# 5 我的订单

## 5.1 需求说明

我的订单根据订单状态展示列表，如图所示：

![order](assets/order.gif) 

**查看接口文档：**

我的订单接口地址及返回结果：

```json
get /api/order/orderInfo/auth/{page}/{limit}?orderStatus={orderStatus}
返回结果：
{
    "code": 200,
    "message": "成功",
    "data": {
        "total": 63,
        "list": [
            {
                "id": 194,
                "createTime": "2023-06-14 19:29:31",
                "userId": 1,
                "nickName": "test",
                "orderNo": "1686713371363",
                "couponId": null,
                "totalAmount": 2999.00,
                "couponAmount": 0.00,
                "originalTotalAmount": 2999.00,
                "feightFee": 0.00,
                "payType": 2,
                "orderStatus": 0,
                "receiverName": "张三",
                "receiverPhone": "15012563333",
                "receiverTagName": "公司",
                "receiverProvince": "130000",
                "receiverCity": "130700",
                "receiverDistrict": "130724",
                "receiverAddress": "河北省张家口市沽源县快乐家园1号",
                "paymentTime": null,
                "deliveryTime": null,
                "receiveTime": null,
                "remark": null,
                "cancelTime": null,
                "cancelReason": null,
                "orderItemList": [
                    {
                        "id": 428,
                        "createTime": "2023-06-14 19:29:31",
                        "orderId": 194,
                        "skuId": 6,
                        "skuName": "小米 红米Note10 5G手机 颜色:黑色 内存:18G",
                        "thumbImg": "http://139.198.127.41:9000/spzx/20230525/665832167-1_u_1.jpg",
                        "skuPrice": 2999.00,
                        "skuNum": 1
                    }
                ]
            },
            ...
        ],
        "pageNum": 1,
        "pageSize": 10,
        "size": 6,
        "startRow": 1,
        "endRow": 6,
        "pages": 7,
        "prePage": 0,
        "nextPage": 2,
        "isFirstPage": true,
        "isLastPage": false,
        "hasPreviousPage": false,
        "hasNextPage": true,
        "navigatePages": 10,
        "navigatepageNums": [
            1,
            2,
            3,
            4,
            5,
            6,
            7
        ],
        "navigateFirstPage": 1,
        "navigateLastPage": 7
    }
}
```

## 5.2 后端接口

### 5.2.1 OrderInfoController

表现层代码：

```java
@Operation(summary = "获取订单分页列表")
@GetMapping("auth/{page}/{limit}")
public Result<PageInfo<OrderInfo>> list(
      @Parameter(name = "page", description = "当前页码", required = true)
      @PathVariable Integer page,

      @Parameter(name = "limit", description = "每页记录数", required = true)
      @PathVariable Integer limit,

      @Parameter(name = "orderStatus", description = "订单状态", required = false)
      @RequestParam(required = false, defaultValue = "") Integer orderStatus) {
   PageInfo<OrderInfo> pageInfo = orderInfoService.findUserPage(page, limit, orderStatus);
   return Result.build(pageInfo, ResultCodeEnum.SUCCESS);
}
```

### 5.2.2 OrderInfoService

业务层代码实现：

```java
//业务接口
PageInfo<OrderInfo> findUserPage(Integer page, Integer limit, Integer orderStatus);

//业务接口实现
@Override
public PageInfo<OrderInfo> findUserPage(Integer page, Integer limit, Integer orderStatus) {
    PageHelper.startPage(page, limit);
    Long userId = AuthContextUtil.getUserInfo().getId();
    List<OrderInfo> orderInfoList = orderInfoMapper.findUserPage(userId, orderStatus);
    return new PageInfo<>(orderInfoList, limit);
}
```

### 5.2.3 OrderInfoMapper

持久层代码实现：

```java
List<OrderInfo> findUserPage(Long userId, Integer orderStatus);
```

### 5.2.4 OrderInfoMapper.xml

在映射文件中定义对应的sql语句：

```xml
<resultMap id="orderInfoMap" type="com.atguigu.spzx.model.entity.order.OrderInfo" autoMapping="true">
   <!--id:表示主键 property：表示实体类的属性名 column：表示通过sql 执行以后查询出来的字段名-->
   <id property="id" column="id"></id>

   <!--一对多-->
   <collection property="orderItemList" ofType="com.atguigu.spzx.model.entity.order.OrderItem" autoMapping="true"
            column="{orderId = id}"
            select="com.atguigu.spzx.order.mapper.OrderItemMapper.findByOrderId">
   </collection>
</resultMap>
<select id="findUserPage" resultMap="orderInfoMap">
   select <include refid="columns" />
   from order_info
   <where>
      <if test="userId != null">
         and user_id = #{userId}
      </if>
      <if test="orderStatus != null">
         and order_status = #{orderStatus}
      </if>
      and is_deleted = 0
   </where>
   order by id desc
</select>
```

### 5.2.5 OrderInfo

订单实体添加属性：

```java
@Schema(description = "订单项列表")
private List<OrderItem> orderItemList;
```

### 5.2.6 OrderItemMapper.xml

在映射文件中定义对应的sql语句：

```xml
<select id="findByOrderId" resultMap="orderItemMap">
   select <include refid="columns" />
   from order_item
   where
      order_id = #{orderId}
      and is_deleted = 0
   order by id desc
</select>
```

### 5.2.7 OrderItemMapper

持久层代码实现：

```java
List<OrderItem> findByOrderId(Long orderId);
```

















