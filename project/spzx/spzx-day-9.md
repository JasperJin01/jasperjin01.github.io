# 1 商品管理

商品管理就是对电商项目中所涉及到的商品数据进行维护。

## 1.1 添加功能

### 1.1.1 需求说明

当用户点击添加按钮的时候，那么此时就弹出对话框，在该对话框中需要展示添加商品的表单。当用户在该表单中点击提交按钮的时候那么此时就需要

将表单进行提交，在后端需要将提交过来的表单数据保存到数据库中即可。

如下所示： 

![product-add](assets/product-add-16855969918791.gif)  

### 1.1.2 核心概念

在完成商品添加功能之前，必须先要了解电商系统中常见的两个核心的概念。

#### SPU

SPU = Standard Product Unit （标准化产品单元）, SPU是商品信息聚合的最小单位，是一组可复用、易检索的标准化信息的**集合**，该[集合](https://so.csdn.net/so/search?q=集合&spm=1001.2101.3001.7020)描述了一

个产品的特性。通俗点讲就是泛指一类商品，这种商品具有相同的属性。

#### SKU

SKU = stock keeping unit(库存量单位) SKU即库存进出计量的单位（买家购买、商家进货、供应商备货、工厂生产都是依据SKU进行的）。是和具体

的属性值有直接的关联关系。SKU是**物理上不可分割的最小存货单元。**也就是说一款商品，可以**根据SKU来确定具体的货物存量。**



以手机为例，假设有一款名为 "XPhone" 的手机品牌，它推出了一款型号为 "X10" 的手机。这款手机一共有以下几种属性：

1、颜色：黑色、白色、金色

2、存储容量：64GB、128GB

3、内存大小：4GB、6GB

那么，“X10” 这款手机就是这个商品的 SPU，它包含所有属性。而根据不同的属性组合，可以形成多个不同的 SKU。如下所示：

| SPU  |  SKU  | 颜色 | 存储容量 | 内存大小 |
| :--: | :---: | :--: | :------: | :------: |
| X10  | SKU1  | 黑色 |   64GB   |   4GB    |
| X10  | SKU2  | 白色 |   64GB   |   4GB    |
| X10  | SKU3  | 金色 |   64GB   |   4GB    |
| X10  | SKU4  | 黑色 |  128GB   |   4GB    |
| X10  | SKU5  | 白色 |  128GB   |   4GB    |
| X10  | SKU6  | 金色 |  128GB   |   4GB    |
| X10  | SKU7  | 黑色 |   64GB   |   6GB    |
| X10  | SKU8  | 白色 |   64GB   |   6GB    |
| X10  | SKU9  | 金色 |   64GB   |   6GB    |
| X10  | SKU10 | 黑色 |  128GB   |   6GB    |
| X10  | SKU11 | 白色 |  128GB   |   6GB    |
| X10  | SKU12 | 金色 |  128GB   |   6GB    |



再以衣服为例，假设有一家服装网店推出了一款名为 "A衬衫" 的衣服。这款衣服一共有以下几种属性：

1、尺寸：S、M、L、XL

2、颜色：白色、黑色、灰色、蓝色

那么，“A衬衫” 这款衣服就是这个商品的 SPU，它包含所有属性。而根据不同的属性组合，可以形成多个不同的 SKU。如下所示：

|  SPU  |  SKU  | 尺寸 | 颜色 |
| :---: | :---: | :--: | :--: |
| A衬衫 | SKU1  |  S   | 白色 |
| A衬衫 | SKU2  |  M   | 白色 |
| A衬衫 | SKU3  |  L   | 白色 |
| A衬衫 | SKU4  |  XL  | 白色 |
| A衬衫 | SKU5  |  S   | 黑色 |
| A衬衫 | SKU6  |  M   | 黑色 |
| A衬衫 | SKU7  |  L   | 黑色 |
| A衬衫 | SKU8  |  XL  | 黑色 |
| A衬衫 | SKU9  |  S   | 灰色 |
| A衬衫 | SKU10 |  M   | 灰色 |
| A衬衫 | SKU11 |  L   | 灰色 |
| A衬衫 | SKU12 |  XL  | 灰色 |
| A衬衫 | SKU13 |  S   | 蓝色 |
| A衬衫 | SKU14 |  M   | 蓝色 |
| A衬衫 | SKU15 |  L   | 蓝色 |
| A衬衫 | SKU16 |  XL  | 蓝色 |

### 1.1.3 加载品牌数据

需求：当用户选择了三级分类以后，此时需要将三级分类所对应的品牌数据查询出来在品牌下拉框中进行展示

#### CategoryBrandController

表现层代码实现：

```java
// com.atguigu.spzx.manager.controller
@GetMapping("/findBrandByCategoryId/{categoryId}")
public Result findBrandByCategoryId(@PathVariable Long categoryId) {
    List<Brand> brandList =   categoryBrandService.findBrandByCategoryId(categoryId);
    return Result.build(brandList , ResultCodeEnum.SUCCESS) ;
}
```

#### CategoryBrandService

业务层代码实现：

```java
// com.atguigu.spzx.manager.service.impl
@Override
public List<Brand> findBrandByCategoryId(Long categoryId) {
    return categoryBrandMapper.findBrandByCategoryId(categoryId);
}
```

#### CategoryBrandMapper

持久层代码实现：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface CategoryBrandMapper {
    public abstract List<Brand> findBrandByCategoryId(Long categoryId);
}
```

#### CategoryBrandMapper.xml

在CategoryBrandMapper.xml映射文件中添加如下sql语句：

```xml
<resultMap id="brandMap" type="com.atguigu.spzx.model.entity.product.Brand" autoMapping="true"></resultMap>
<select id="findBrandByCategoryId" resultMap="brandMap">
    select
    b.*
    from category_brand cb
    left join brand b  on b.id = cb.brand_id
    where cb.category_id = #{categoryId} and cb.is_deleted = 0
    order by cb.id desc
</select>
```

### 1.1.4 加载商品单元数据

![image-20230821222221662](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230821222221662.png)

需求：当添加商品的表单对话框展示出来以后，此时就需要从数据库中查询出来所有的商品单元数据，并将查询到的商品单元数据在商品单元下拉框中

进行展示。

#### ProductUnitController

表现层代码实现：

```java
// com.atguigu.spzx.manager.controller
@GetMapping("findAll")
public Result<List<ProductUnit>> findAll() {
    List<ProductUnit> productUnitList = productUnitService.findAll();
    return Result.build(productUnitList , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductUnitService

业务层代码实现：

```java
// com.atguigu.spzx.manager.service.impl
@Override
public List<ProductUnit> findAll() {
    return productUnitMapper.findAll() ;
}
```

#### ProductUnitMapper

持久层代码实现：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface ProductUnitMapper {
    public abstract List<ProductUnit> findAll();
}
```

#### ProductUnitMapper.xml

在ProductUnitMapper.xml映射文件中添加如下的sql语句：

```xml
<select id="findAll" resultMap="productUnitMap">
    select <include refid="columns" />
    from product_unit
    where is_deleted = 0
    order by id
</select>
```

### 1.1.5 加载商品规格数据

需求：当添加商品的表单对话框展示出来以后，此时就需要从数据库中查询出来所有的商品规格数据，并将查询到的商品规格数据在商品规格下拉框中

进行展示。

#### ProductSpecController

表现层代码实现：

```java
// com.atguigu.spzx.manager.controller
@GetMapping("findAll")
public Result findAll() {
    List<ProductSpec> list = productSpecService.findAll();
    return Result.build(list , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductSpecService

业务层代码实现：

```java
// com.atguigu.spzx.manager.service.impl;
@Override
public List<ProductSpec> findAll() {
    return productSpecMapper.findAll();;
}
```

#### ProductSpecMapper

持久层代码实现：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface ProductSpecMapper {
    public abstract List<ProductSpec> findAll();
}
```

#### ProductSpecMapper.xml

在ProductSpecMapper.xml映射文件中添加如下sql语句：

```xml
<select id="findAll" resultMap="productSpecMap">
    select <include refid="columns" />
    from product_spec
    order by id desc
</select>
```

### 1.1.6 保存商品数据接口

#### 思路分析

思路分析：

1、前端提交过来的数据，包含了SPU的基本数据，SKU的列表数据，商品详情数据

2、后端可以直接使用Product接收请求参数，但是需要扩展对应的属性

3、保存数据的时候需要操作三张表：product、product_sku、product_detail



#### Product

```sql
CREATE TABLE `product` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL COMMENT '商品名称',
  `brand_id` bigint DEFAULT NULL COMMENT '品牌ID',
  `category1_id` bigint DEFAULT NULL COMMENT '一级分类id',
  `category2_id` bigint DEFAULT NULL COMMENT '二级分类id',
  `category3_id` bigint DEFAULT NULL COMMENT '三级分类id',
  `unit_name` varchar(50) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL COMMENT '计量单位',
  `slider_urls` text COMMENT '轮播图',
  `spec_value` varchar(255) DEFAULT NULL COMMENT '商品规格json',
  `status` tinyint NOT NULL DEFAULT '0' COMMENT '线上状态：0-初始值，1-上架，-1-自主下架',
  `audit_status` tinyint NOT NULL DEFAULT '0' COMMENT '审核状态：0-初始值，1-通过，-1-未通过',
  `audit_message` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL COMMENT '审核信息',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品';

/*Data for the table `product` */

insert  into `product`(`id`,`name`,`brand_id`,`category1_id`,`category2_id`,`category3_id`,`unit_name`,`slider_urls`,`spec_value`,`status`,`audit_status`,`audit_message`,`create_time`,`update_time`,`is_deleted`) values (1,'小米 红米Note10 5G手机',1,1,2,3,'个','http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-6_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-4_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-1_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1 (1).jpg,http://139.198.127.41:9000/spzx/20230525/665832167-3_u_1.jpg\n','[{\"key\":\"颜色\",\"valueList\":[\"白色\",\"红色\",\"黑色\"]},{\"key\":\"内存\",\"valueList\":[\"8G\",\"18G\"]}]',1,1,'审批通过','2023-05-25 14:21:07','2023-06-01 09:40:53',0),(2,'华为笔记本',2,1,41,42,'个','http://139.198.127.41:9000/spzx/20230525/f1d7433dbe88c25e.jpg.avif,http://139.198.127.41:9000/spzx/20230525/4b5a68a9bfbd0795.jpg.avif,http://139.198.127.41:9000/spzx/20230525/13beb824e9dcfc88.jpg.avif,http://139.198.127.41:9000/spzx/20230525/3976e9cfaf7c6ceb.jpg.avif,http://139.198.127.41:9000/spzx/20230525/35b3b7e2e67b98b2.jpg.avif,http://139.198.127.41:9000/spzx/20230525/c8f2eae0d36b6270.jpg.avif,http://139.198.127.41:9000/spzx/20230525/fd5c6bfa6f3c4d43.jpg.avif','[{\"key\":\"内存\",\"valueList\":[\"8G\",\"16G\",\"32G\"]}]',-1,1,'审批通过','2023-05-25 15:18:28','2023-07-01 15:44:17',0),(3,'华为手机最新款1',2,1,2,3,'台','http://127.0.0.1:9000/spzx-bucket/20230719/7c03fe0fde114c5792c2b4de0bed9adc.jpg','[{\"key\":\"高度\",\"valueList\":[\"200\"]},{\"key\":\"容积\",\"valueList\":[\"60L\"]}]',1,1,'审批通过','2023-07-19 11:01:11','2023-07-19 11:28:03',1);
```

修改Product实体类，添加接收sku列表和商品详情的属性，如下所示：

```java
// com.atguigu.spzx.model.entity.product;
@Data
public class Product extends BaseEntity {

	private String name;					// 商品名称
	private Long brandId;					// 品牌ID
	private Long category1Id;				// 一级分类id
	private Long category2Id;				// 二级分类id
	private Long category3Id;				// 三级分类id
	private String unitName;				// 计量单位
	private String sliderUrls;				// 轮播图
	private String specValue;				// 商品规格值json串
	private Integer status;					// 线上状态：0-初始值，1-上架，-1-自主下架
	private Integer auditStatus;			// 审核状态
	private String auditMessage;			// 审核信息

	// 扩展的属性，用来封装响应的数据
	private String brandName;				// 品牌
	private String category1Name;			// 一级分类
	private String category2Name;			// 二级分类
	private String category3Name;			// 三级分类


	private List<ProductSku> productSkuList;		// sku列表集合
	private String detailsImageUrls;				// 图片详情列表

}
```

#### ProductSku

```sql
CREATE TABLE `product_sku` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `sku_code` varchar(30) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL COMMENT '商品编号',
  `sku_name` varchar(255) DEFAULT NULL,
  `product_id` bigint DEFAULT NULL COMMENT '商品ID',
  `thumb_img` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL COMMENT '缩略图路径',
  `sale_price` decimal(10,2) DEFAULT NULL COMMENT '售价',
  `market_price` decimal(10,2) DEFAULT NULL COMMENT '市场价',
  `cost_price` decimal(10,2) DEFAULT NULL COMMENT '成本价',
  `stock_num` int DEFAULT NULL COMMENT '库存数',
  `sale_num` int NOT NULL DEFAULT '0' COMMENT '销量',
  `sku_spec` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT '' COMMENT 'sku规格信息json',
  `weight` decimal(10,2) DEFAULT NULL COMMENT '重量',
  `volume` decimal(10,2) DEFAULT NULL COMMENT '体积',
  `status` tinyint DEFAULT NULL COMMENT '线上状态：0-初始值，1-上架，-1-自主下架',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品sku';

insert  into `product_sku`(`id`,`sku_code`,`sku_name`,`product_id`,`thumb_img`,`sale_price`,`market_price`,`cost_price`,`stock_num`,`sale_num`,`sku_spec`,`weight`,`volume`,`status`,`create_time`,`update_time`,`is_deleted`) values (1,'1_0','小米 红米Note10 5G手机 颜色:白色 内存:8G',1,'http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1 (1).jpg','1999.00','2019.00','1599.00',96,4,'颜色:白色,内存:8G','1.00','1.00',NULL,'2023-05-25 14:21:07','2023-06-16 08:49:45',0),(2,'1_1','小米 红米Note10 5G手机 颜色:白色 内存:18G',1,'http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1 (1).jpg','2999.00','3019.00','2599.00',97,3,'颜色:白色,内存:18G','1.00','1.00',NULL,'2023-05-25 14:21:07','2023-06-16 08:49:45',0),(3,'1_2','小米 红米Note10 5G手机 颜色:红色 内存:8G',1,'http://139.198.127.41:9000/spzx/20230525/665832167-3_u_1.jpg','1999.00','2019.00','1599.00',100,0,'颜色:红色,内存:8G','1.00','1.00',NULL,'2023-05-25 14:21:07','2023-05-25 14:32:20',0),(4,'1_3','小米 红米Note10 5G手机 颜色:红色 内存:18G',1,'http://139.198.127.41:9000/spzx/20230525/665832167-3_u_1.jpg','2999.00','3019.00','2599.00',100,0,'颜色:红色,内存:18G','1.00','1.00',NULL,'2023-05-25 14:21:07','2023-05-25 14:32:23',0),(5,'1_4','小米 红米Note10 5G手机 颜色:黑色 内存:8G',1,'http://139.198.127.41:9000/spzx/20230525/665832167-1_u_1.jpg','1999.00','2019.00','1599.00',100,0,'颜色:黑色,内存:8G','1.00','1.00',NULL,'2023-05-25 14:21:07','2023-05-25 14:32:26',0),(6,'1_5','小米 红米Note10 5G手机 颜色:黑色 内存:18G',1,'http://139.198.127.41:9000/spzx/20230525/665832167-1_u_1.jpg','2999.00','3019.00','2599.00',99,1,'颜色:黑色,内存:18G','1.00','1.00',NULL,'2023-05-25 14:21:07','2023-06-01 15:36:23',0),(7,'2_0','华为笔记本 内存:8G',2,'http://139.198.127.41:9000/spzx/20230525/4b5a68a9bfbd0795.jpg.avif','3999.00','3999.00','3999.00',100,0,'内存:8G','1.00','2.00',NULL,'2023-05-25 15:18:28','2023-07-01 15:44:17',0),(8,'2_1','华为笔记本 内存:16G',2,'http://139.198.127.41:9000/spzx/20230525/35b3b7e2e67b98b2.jpg.avif','4999.00','4999.00','4999.00',100,0,'内存:16G','1.00','2.00',NULL,'2023-05-25 15:18:28','2023-07-01 15:44:17',0),(9,'2_2','华为笔记本 内存:32G',2,'http://139.198.127.41:9000/spzx/20230525/c8f2eae0d36b6270.jpg.avif','5999.00','5999.00','5999.00',100,0,'内存:32G','1.00','2.00',NULL,'2023-05-25 15:18:28','2023-07-01 15:44:17',0),(10,'3_0','华为手机最新款200 + 60L',3,'http://127.0.0.1:9000/spzx-bucket/20230719/25f6566d9bdd4d349b051f9223c4c672.jpg','2300.00','2500.00','200.00',1,0,'200 + 60L','1.00','20.00',0,'2023-07-19 11:01:11','2023-07-19 11:28:03',1)
```

商品sku实体类定义：

```java
// com.atguigu.spzx.model.entity.product;
@Data
public class ProductSku extends BaseEntity { 

	private String skuCode;
	private String skuName;
	private Long productId;
	private String thumbImg;
	private BigDecimal salePrice;
	private BigDecimal marketPrice;
	private BigDecimal costPrice;
	private Integer stockNum;
	private Integer saleNum;
	private String skuSpec;
	private String weight;
	private String volume;
	private Integer status;

}
```

#### ProductDetails

```sql
CREATE TABLE `product_details` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `product_id` bigint DEFAULT NULL COMMENT '商品id',
  `image_urls` text COMMENT '详情图片地址',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品sku属性表';

/*Data for the table `product_details` */

insert  into `product_details`(`id`,`product_id`,`image_urls`,`create_time`,`update_time`,`is_deleted`) values (1,1,'http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-6_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-4_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-1_u_1.jpg,http://139.198.127.41:9000/spzx/20230525/665832167-5_u_1 (1).jpg,http://139.198.127.41:9000/spzx/20230525/665832167-3_u_1.jpg','2023-05-25 14:21:07','2023-05-25 14:24:46',0),(2,2,'http://139.198.127.41:9000/spzx/20230525/3976e9cfaf7c6ceb.jpg.avif,http://139.198.127.41:9000/spzx/20230525/f1d7433dbe88c25e.jpg.avif,http://139.198.127.41:9000/spzx/20230525/13beb824e9dcfc88.jpg.avif,http://139.198.127.41:9000/spzx/20230525/4b5a68a9bfbd0795.jpg.avif,http://139.198.127.41:9000/spzx/20230525/35b3b7e2e67b98b2.jpg.avif,http://139.198.127.41:9000/spzx/20230525/c8f2eae0d36b6270.jpg.avif,http://139.198.127.41:9000/spzx/20230525/fd5c6bfa6f3c4d43.jpg.avif','2023-05-25 15:18:28','2023-07-01 15:44:17',0),(3,3,'http://127.0.0.1:9000/spzx-bucket/20230719/48aa930d779d486da56cdb85c3862a71.jpg,http://127.0.0.1:9000/spzx-bucket/20230719/5090c475fc7344ebae61011d21ea88c5.jpg,http://127.0.0.1:9000/spzx-bucket/20230719/7b391bb468bb4ab8a181808cf4241512.jpg','2023-07-19 11:01:11','2023-07-19 11:28:03',1);
```

商品详情实体类：

```java
// com.atguigu.spzx.model.entity.product;
@Data
public class ProductDetails extends BaseEntity {

	private Long productId;
	private String imageUrls;

}
```

#### ProductController

表现层代码：

```java
// com.atguigu.spzx.manager.controller;
@PostMapping("/save")
public Result save(@RequestBody Product product) {
    productService.save(product);
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductService

业务层代码：

```java
// com.atguigu.spzx.manager.service.impl;
@Transactional
@Override
public void save(Product product) {

    // 保存商品数据
    product.setStatus(0);              // 设置上架状态为0
    product.setAuditStatus(0);         // 设置审核状态为0
    productMapper.save(product);

    // 保存商品sku数据
    List<ProductSku> productSkuList = product.getProductSkuList();
    for(int i=0,size=productSkuList.size(); i<size; i++) {

        // 获取ProductSku对象
        ProductSku productSku = productSkuList.get(i);
        productSku.setSkuCode(product.getId() + "_" + i);       // 构建skuCode
        productSku.setProductId(product.getId());               // 设置商品id
        productSku.setSkuName(product.getName() + productSku.getSkuSpec());
        productSku.setSaleNum(0);                               // 设置销量
        productSku.setStatus(0);
        productSkuMapper.save(productSku);                    // 保存数据

    }

    // 保存商品详情数据
    ProductDetails productDetails = new ProductDetails();
    productDetails.setProductId(product.getId());
    productDetails.setImageUrls(product.getDetailsImageUrls());
    productDetailsMapper.save(productDetails);

}
```

#### ProductMapper

商品持久层代码：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface ProductMapper {
    public abstract void save(Product product);
}
```

#### ProductMapper.xml

在ProductMapper.xml文件中添加如下的SQL语句：

```xml
<insert id="save" useGeneratedKeys="true" keyProperty="id">
    insert into product (
        id,
        name,
        brand_id,
        category1_id,
        category2_id,
        category3_id,
        unit_name,
        slider_urls,
        spec_value,
        status,
        audit_status,
        audit_message,
        create_time,
        update_time,
        is_deleted
    ) values (
        #{id},
        #{name},
        #{brandId},
        #{category1Id},
        #{category2Id},
        #{category3Id},
        #{unitName},
        #{sliderUrls},
        #{specValue},
        #{status},
        #{auditStatus},
        #{auditMessage},
        now(),
        now() ,
        0
    )
</insert>
```

#### ProductSkuMapper

商品SKU的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductSkuMapper {
    public abstract void save(ProductSku productSku);
}
```

#### ProductSkuMapper.xml

在ProductSkuMapper.xml映射文件中添加如下的SQL语句：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.spzx.manager.mapper.ProductSkuMapper">

    <insert id="save" useGeneratedKeys="true" keyProperty="id">
        insert into product_sku (
            id,
            sku_code,
            sku_name,
            product_id,
            thumb_img,
            sale_price,
            market_price,
            cost_price,
            stock_num,
            sku_spec,
            weight,
            volume,
            status,
            sale_num,
            create_time,
            update_time,
            is_deleted
        ) values (
             #{id},
             #{skuCode},
             #{skuName},
             #{productId},
             #{thumbImg},
             #{salePrice},
             #{marketPrice},
             #{costPrice},
             #{stockNum},
             #{skuSpec},
             #{weight},
             #{volume},
             #{status},
             #{saleNum},
             now(),
             now(),
             0
         )
    </insert>

</mapper>
```

#### ProductDetailsMapper

商品详情的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductDetailsMapper {
    public abstract void save(ProductDetails productDetails);
}
```

#### ProductDetailsMapper.xml

在ProductDetailsMapper.xml映射文件中添加如下的SQL语句：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.spzx.manager.mapper.ProductDetailsMapper">

    <insert id="save" useGeneratedKeys="true" keyProperty="id">
        insert into product_details (
        	id,
            product_id,
            image_urls,
            create_time,update_time,is_deleted
        ) values (
            #{id},
            #{productId},
            #{imageUrls},
            now(),
            now(),
            0
        )
    </insert>

</mapper>
```

## 1.2 修改功能

### 1.2.1 需求说明

当用户点击修改按钮的时候，那么此时就弹出对话框，在该对话框的商品表单中回显商品相关数据，此时用户对商品数据进行修改，修改完毕以后点击

提交按钮将表单进行提交，后端服务修改数据库中数据即可。

效果如下所示：

![product-update](assets/product-update.gif) 

修改功能实现的时候需要开发两个接口：

1、根据id查询商品详情

2、保存修改数据接口

### 1.2.2 查询商品详情

#### ProductController

表现层代码：

```java
// com.atguigu.spzx.manager.controller;
@GetMapping("/getById/{id}")
public Result<Product> getById(@PathVariable Long id) {
    Product product = productService.getById(id);
    return Result.build(product , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductService

业务层代码：

```java
// com.atguigu.spzx.manager.service.impl;
@Override
public Product getById(Long id) {

    // 根据id查询商品数据
    Product product = productMapper.selectById(id);

    // 根据商品的id查询sku数据
    List<ProductSku> productSkuList = productSkuMapper.selectByProductId(id);
    product.setProductSkuList(productSkuList);

    // 根据商品的id查询商品详情数据
    ProductDetails productDetails = productDetailsMapper.selectByProductId(product.getId());
    product.setDetailsImageUrls(productDetails.getImageUrls());

    // 返回数据
    return product;
}
```

#### ProductMapper

商品持久层代码：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface ProductMapper {
    public abstract Product selectById(Long id);
}
```

#### ProductMapper.xml

在ProductMapper.xml文件中添加如下的SQL语句：

```xml
<select id="selectById" resultMap="productMap">
    select
        p.id, p.name , p.brand_id , p.category1_id , p.category2_id , p.category3_id, p.unit_name,
        p.slider_urls , p.spec_value , p.status , p.audit_status , p.audit_message , p.create_time , p.update_time , p.is_deleted ,
        b.name brandName , c1.name category1Name , c2.name category2Name , c2.name category3Name
    from product p
        LEFT JOIN brand b on b.id = p.brand_id
        LEFT JOIN category c1 on c1.id = p.category1_id
        LEFT JOIN category c2 on c2.id = p.category2_id
        LEFT JOIN category c3 on c3.id = p.category2_id
    where
    p.id = #{id}
</select>
```

#### ProductSkuMapper

商品SKU的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductSkuMapper {
    public abstract List<ProductSku> selectByProductId(Long id);
}
```

#### ProductSkuMapper.xml

在ProductSkuMapper.xml映射文件中添加如下的SQL语句：

```xml
<resultMap id="productSkuMap" type="com.atguigu.spzx.model.entity.product.ProductSku" autoMapping="true"></resultMap>

<!-- 用于select查询公用抽取的列 -->
<sql id="columns">
    id,sku_code,sku_name,product_id,thumb_img,sale_price,market_price,cost_price,stock_num,sku_spec,weight,volume,status,create_time,update_time,is_deleted
</sql>

<select id="selectByProductId" resultMap="productSkuMap">
    select <include refid="columns" />
    from product_sku
    where product_id = #{productId}
    and is_deleted = 0
    order by id desc
</select>
```

#### ProductDetailsMapper

商品详情的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductDetailsMapper {
    public abstract ProductDetails selectByProductId(Long id);
}
```

#### ProductDetailsMapper.xml

在ProductDetailsMapper.xml映射文件中添加如下的SQL语句：

```xml
<resultMap id="productDetailsMap" type="com.atguigu.spzx.model.entity.product.ProductDetails" autoMapping="true"></resultMap>

<!-- 用于select查询公用抽取的列 -->
<sql id="columns">
    id,product_id,image_urls,create_time,update_time,is_deleted
</sql>

<select id="selectByProductId" resultMap="productDetailsMap">
    select <include refid="columns" />
    from product_details
    where
    product_id = #{productId}
</select>
```

### 1.2.3 保存修改数据接口

#### ProductController

表现层代码：

```java
// com.atguigu.spzx.manager.controller;
@PutMapping("/updateById")
public Result updateById(@Parameter(name = "product", description = "请求参数实体类", required = true) @RequestBody Product product) {
    productService.updateById(product);
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductService

业务层代码：

```java
// com.atguigu.spzx.manager.service.impl;
@Transactional
@Override
public void updateById(Product product) {

    // 修改商品基本数据
    productMapper.updateById(product);

    // 修改商品的sku数据
    List<ProductSku> productSkuList = product.getProductSkuList();
    productSkuList.forEach(productSku -> {
        productSkuMapper.updateById(productSku);
    });

    // 修改商品的详情数据
    ProductDetails productDetails = productDetailsMapper.selectByProductId(product.getId());
    productDetails.setImageUrls(product.getDetailsImageUrls());
    productDetailsMapper.updateById(productDetails);

}
```

#### ProductMapper

商品持久层代码：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface ProductMapper {
    public abstract  void updateById(Product product);
}
```

#### ProductMapper.xml

在ProductMapper.xml文件中添加如下的SQL语句：

```xml
<update id="updateById" >
    update product set
    <if test="name != null and name != ''">
        name = #{name},
    </if>
    <if test="brandId != null and brandId != ''">
        brand_id = #{brandId},
    </if>
    <if test="category1Id != null and category1Id != ''">
        category1_id = #{category1Id},
    </if>
    <if test="category2Id != null and category2Id != ''">
        category2_id = #{category2Id},
    </if>
    <if test="category3Id != null and category3Id != ''">
        category3_id = #{category3Id},
    </if>
    <if test="unitName != null and unitName != ''">
        unit_name = #{unitName},
    </if>
    <if test="sliderUrls != null and sliderUrls != ''">
        slider_urls = #{sliderUrls},
    </if>
    <if test="specValue != null and specValue != ''">
        spec_value = #{specValue},
    </if>
    <if test="status != null and status != ''">
        status = #{status},
    </if>
    <if test="auditStatus != null and auditStatus != ''">
        audit_status = #{auditStatus},
    </if>
    <if test="auditMessage != null and auditMessage != ''">
        audit_message = #{auditMessage},
    </if>
    update_time =  now()
    where
    id = #{id}
</update>
```

#### ProductSkuMapper

商品SKU的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductSkuMapper {
    public abstract  void updateById(ProductSku productSku);
}
```

#### ProductSkuMapper.xml

在ProductSkuMapper.xml映射文件中添加如下的SQL语句：

```xml
<update id="updateById" >
    update product_sku set
    <if test="skuCode != null and skuCode != ''">
        sku_code = #{skuCode},
    </if>
    <if test="skuName != null and skuName != ''">
        sku_name = #{skuName},
    </if>
    <if test="productId != null and productId != ''">
        product_id = #{productId},
    </if>
    <if test="thumbImg != null and thumbImg != ''">
        thumb_img = #{thumbImg},
    </if>
    <if test="salePrice != null and salePrice != ''">
        sale_price = #{salePrice},
    </if>
    <if test="marketPrice != null and marketPrice != ''">
        market_price = #{marketPrice},
    </if>
    <if test="costPrice != null and costPrice != ''">
        cost_price = #{costPrice},
    </if>
    <if test="stockNum != null and stockNum != ''">
        stock_num = #{stockNum},
    </if>
    <if test="skuSpec != null and skuSpec != ''">
        sku_spec = #{skuSpec},
    </if>
    <if test="weight != null and weight != ''">
        weight = #{weight},
    </if>
    <if test="volume != null and volume != ''">
        volume = #{volume},
    </if>
    <if test="status != null and status != ''">
        status = #{status},
    </if>
    update_time =  now()
    where
    id = #{id}
</update>
```

#### ProductDetailsMapper

商品详情的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductDetailsMapper {
    public abstract void updateById(ProductDetails productDetails);
}
```

#### ProductDetailsMapper.xml

在ProductDetailsMapper.xml映射文件中添加如下的SQL语句：

```xml
<update id="updateById" >
    update product_details set
    <if test="productId != null and productId != ''">
        product_id = #{productId},
    </if>
    <if test="imageUrls != null and imageUrls != ''">
        image_urls = #{imageUrls},
    </if>
    update_time =  now()
    where
    id = #{id}
</update>
```

## 1.3 删除商品

### 1.3.1 需求说明

当点击删除按钮的时候此时需要弹出一个提示框，询问是否需要删除数据？如果用户点击是，那么此时向后端发送请求传递id参数，后端接收id参数进

行逻辑删除。

效果如下所示：

![product-delete](assets/product-delete.gif)

### 1.3.2 后端接口

#### ProductController

表现层代码：

```java
// com.atguigu.spzx.manager.controller;
@DeleteMapping("/deleteById/{id}")
public Result deleteById(@Parameter(name = "id", description = "商品id", required = true) @PathVariable Long id) {
    productService.deleteById(id);
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductService

业务层代码：

```java
// com.atguigu.spzx.manager.service.impl;
@Transactional
@Override
public void deleteById(Long id) {
    productMapper.deleteById(id);                   // 根据id删除商品基本数据
    productSkuMapper.deleteByProductId(id);         // 根据商品id删除商品的sku数据
    productDetailsMapper.deleteByProductId(id);     // 根据商品的id删除商品的详情数据
}
```

#### ProductMapper

商品持久层代码：

```java
// com.atguigu.spzx.manager.mapper
@Mapper
public interface ProductMapper {
    public abstract void deleteById(Long id);
}
```

#### ProductMapper.xml

在ProductMapper.xml文件中添加如下的SQL语句：

```xml
<update id="deleteById">
    update product set
    update_time = now() ,
    is_deleted = 1
    where
    id = #{id}
</update>
```

#### ProductSkuMapper

商品SKU的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductSkuMapper {
    public abstract void deleteByProductId(Long id);
}
```

#### ProductSkuMapper.xml

在ProductSkuMapper.xml映射文件中添加如下的SQL语句：

```xml
<update id="deleteByProductId">
    update product_sku set
    update_time = now() ,
    is_deleted = 1
    where
    product_id = #{productId}
</update>
```

#### ProductDetailsMapper

商品详情的持久层代码：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface ProductDetailsMapper {
    public abstract void deleteByProductId(Long id);
}
```

#### ProductDetailsMapper.xml

在ProductDetailsMapper.xml映射文件中添加如下的SQL语句：

```xml
<update id="deleteByProductId">
    update product_details set
    update_time = now() ,
    is_deleted = 1
    where
    product_id = #{productId}
</update>
```

## 1.4 商品审核

### 1.4.1 需求说明

当点击审核按钮的时候此时需要弹出一个对话框，在该对话框中展示商品的详情信息，用户可以在该对话框中点击通过或者驳回按钮对商品进行审核操

作。

效果如下所示：

![product-update-auti](assets/product-update-auti.gif)

### 1.4.2 后端接口

#### ProductController

表现层代码：

```java
// com.atguigu.spzx.manager.controller;
@GetMapping("/updateAuditStatus/{id}/{auditStatus}")
public Result updateAuditStatus(@PathVariable Long id, @PathVariable Integer auditStatus) {
    productService.updateAuditStatus(id, auditStatus);
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductService

业务层代码：

```java
// com.atguigu.spzx.manager.service.impl;
@Override
public void updateAuditStatus(Long id, Integer auditStatus) {
    Product product = new Product();
    product.setId(id);
    if(auditStatus == 1) {
        product.setAuditStatus(1);
        product.setAuditMessage("审批通过");
    } else {
        product.setAuditStatus(-1);
        product.setAuditMessage("审批不通过");
    }
    productMapper.updateById(product);
}
```

## 1.5 商品上下架

### 1.5.1 需求说明

当用户点击上架按钮的时候对商品进行上架操作，点击下架按钮的时候对商品进行下架操作。

实现思路：更改商品的上下架状态

效果如下所示：

![product-update-status](assets/product-update-status.gif) 

### 1.5.2 后端接口

#### ProductController

表现层代码：

```java
// com.atguigu.spzx.manager.controller;
@GetMapping("/updateStatus/{id}/{status}")
public Result updateStatus(@PathVariable Long id, @PathVariable Integer status) {
    productService.updateStatus(id, status);
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}
```

#### ProductService

业务层代码：

```java
// com.atguigu.spzx.manager.service.impl;
@Override
public void updateStatus(Long id, Integer status) {
    Product product = new Product();
    product.setId(id);
    if(status == 1) {
        product.setStatus(1);
    } else {
        product.setStatus(-1);
    }
    productMapper.updateById(product);
}
```

# 2 订单管理

用户在前台系统中购买完商品以后生成对应的订单数据，在后台管理系统中就可以查看到订单数据，在后台管理系统中也可以对订单数据进行统计，形

成图形化报表【柱状图、饼状图、曲线图、散点图..】用于数据分析。

## 2.1 订单数据统计需求说明

需求说明：统计指定时间段每一天订单的总金额，并且以柱状图的形式进行展示。

效果如下所示：

 ![order-sta](assets/order-sta.gif) 

## 2.2 Echarts

要想将统计的数据以图表的方式进行展示，那么就需要使用到一些图形报表工具。常见的图形报表工具：Echarts

### 2.2.1 Echarts简介

官网地址：https://echarts.apache.org/zh/index.html

![image-20230601203943078](assets/image-20230601203943078.png) 

Echarts是一个基于JavaScript的开源可视化图表库，由百度前端开发团队研发和维护。它提供了丰富的图表类型、数据统计分析、动态数据更新、多

维数据展示等功能，可以帮助开发人员在 Web 应用、大屏展示、移动端等各种场景下，快速构建出高度定制化的交互式可视化图表。

Echarts支持多种图表类型，如**线图、柱状图、饼图、雷达图、散点图**等，还支持动态数据显示、图表联动、混搭图表等复杂功能。同时，ECharts

还支持多种数据格式、数据预处理和自定义附加组件等扩展功能，让用户能够方便地实现各种需求和定制化要求。

ECharts易于上手，提供了丰富的API，以及完善的文档和示例，使得开发人员可以更快速、更便捷地使用它来进行数据可视化。

### 2.2.2 Echarts入门

需求：使用Echarts的柱状图展示每一天的商品的订单总金额。

效果如下所示：

![order](assets/order.gif) 

实现步骤：

1、安装Echarts，官网地址：https://echarts.apache.org/handbook/zh/basics/import

```shell
npm install echarts --save
```

2、添加订单管理菜单

通过系统管理的菜单管理，添加订单管理菜单，如下所示：

![image-20230601205954147](assets/image-20230601205954147.png) 

给管理员角色分配订单管理访问权限如下所示：

![image-20230601210128807](assets/image-20230601210128807.png) 

在src/views目录下创建orderStatistics.vue文件，如下所示：

![image-20230601210204878](assets/image-20230601210204878.png) 

在src/router/modules目录下创建order.js文件，配置路由：

```javascript
const Layout = () => import('@/layout/index.vue')
const orderStatistics = () => import('@/views/order/orderStatistics.vue')

export default [
    {
      path: '/order',
      component: Layout,
      name: 'order',
      meta: {
        title: '订单管理',
      },
      icon: 'Operation',
      children: [
            {
            path: '/orderStatistics',
            name: 'orderStatistics',
            component: orderStatistics,
            meta: {
                title: '订单统计',
            },
            }
        ],
    },
  ]
```

在src/router/index.js中添加订单管理路由：

```javascript
import order from './modules/order'

// 动态菜单
export const asyncRoutes = [...system,...base,...product,...order]
```

3、参考官方文档示例代码，完成入门案例，代码如下所示：

```vue
<template>
    <div ref="orderTotalAmountDiv" style="width: 100%; height: 100%;"></div>
</template>

<script setup>
import { ref , onMounted } from 'vue'
import * as echarts from 'echarts';         // 导入Echart库中所有的图形报表组件

// 定义chart数据模型，用来选中div组件
const orderTotalAmountDiv = ref() 

// 需要在onMounted钩子函数中对div区域使用echarts进行初始化
onMounted( () => {

    // 基于准备好的dom，初始化echarts实例
    var orderTotalAmountChart = echarts.init(orderTotalAmountDiv.value);

    // 绘制图表
    orderTotalAmountChart.setOption({
        title: {
            text: '订单数据统计'
        },
        tooltip: {},
        xAxis: {
            data: ['2023-06-01', '2023-06-02', '2023-06-03', '2023-06-04', '2023-06-05', '2023-06-06']
        },
        yAxis: {},
        series: [
            {
                name: '订单总金额（万元）',
                type: 'bar',
                data: [20, 40, 30, 100, 50, 25]
            }
        ]
	});

})

</script>

<style scoped>

</style>
```

其他的常见的配置项以及属性介绍可以参看官方文档：https://echarts.apache.org/zh/option.html#tooltip.axisPointer

## 2.3 订单相关数据库表介绍

要进行订单数据的统计，首先就需要了解一下和订单相关的数据库表，如下所示：

```sql
-- 订单信息表
CREATE TABLE `order_info` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  `user_id` bigint NOT NULL DEFAULT '0' COMMENT '会员_id',
  `nick_name` varchar(200) DEFAULT NULL COMMENT '昵称',
  `order_no` char(64) NOT NULL DEFAULT '' COMMENT '订单号',
  `coupon_id` bigint DEFAULT NULL COMMENT '使用的优惠券',
  `total_amount` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '订单总额',
  `coupon_amount` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '优惠券',
  `original_total_amount` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '原价金额',
  `feight_fee` decimal(10,2) NOT NULL DEFAULT '0.00' COMMENT '运费',
  `pay_type` tinyint DEFAULT NULL COMMENT '支付方式【1->微信】',
  `order_status` tinyint NOT NULL DEFAULT '0' COMMENT '订单状态【0->待付款；1->待发货；2->已发货；3->待用户收货，已完成；-1->已取消】',
  `receiver_name` varchar(100) DEFAULT NULL COMMENT '收货人姓名',
  `receiver_phone` varchar(32) DEFAULT NULL COMMENT '收货人电话',
  `receiver_post_code` varchar(32) DEFAULT NULL COMMENT '收货人邮编',
  `receiver_province` bigint DEFAULT NULL COMMENT '省份/直辖市',
  `receiver_city` bigint DEFAULT NULL COMMENT '城市',
  `receiver_district` bigint DEFAULT NULL COMMENT '区',
  `receiver_address` varchar(200) DEFAULT NULL COMMENT '详细地址',
  `payment_time` datetime DEFAULT NULL COMMENT '支付时间',
  `delivery_time` datetime DEFAULT NULL COMMENT '发货时间',
  `receive_time` datetime DEFAULT NULL COMMENT '确认收货时间',
  `remark` varchar(500) DEFAULT NULL COMMENT '订单备注',
  `cancel_time` datetime DEFAULT NULL COMMENT '取消订单时间',
  `cancel_reason` varchar(255) DEFAULT NULL COMMENT '取消订单原因',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=165 DEFAULT CHARSET=utf8mb3 COMMENT='订单';

-- 订单明细表
CREATE TABLE `order_item` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  `order_id` bigint DEFAULT NULL COMMENT 'order_id',
  `sku_id` bigint DEFAULT NULL COMMENT '商品sku编号',
  `sku_name` varchar(255) DEFAULT NULL COMMENT '商品sku名字',
  `thumb_img` varchar(500) DEFAULT NULL COMMENT '商品sku图片',
  `sku_price` decimal(10,2) DEFAULT NULL COMMENT '商品sku价格',
  `sku_num` int DEFAULT NULL COMMENT '商品购买的数量',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=364 DEFAULT CHARSET=utf8mb3 COMMENT='订单项信息';
```

注意：订单和订单明细之间的关系是一对多

## 2.4 订单数据统计实现思路

在入门案例中展示柱状图所需要的数据是假数据，现在就需要将数据换成真实统计数据。

数据统计的实现方案：

方案一：每一次展示数据的时候都从订单数据库中进行一次统计查询，对应的SQL语句如下所示

```sql
select DATE_FORMAT(oi.create_time ,'%Y-%m-%d') date , sum(oi.total_amount)  totalAmount from order_info oi GROUP BY DATE_FORMAT(oi.create_time ,'%Y-%m-%d');
```

弊端：在进行分组查询时，如果数据量较大，直接进行分组查询的效率会受到一定影响，因为需要对所有数据进行聚合计算，耗费时间和资源。

方案二：将分组结果计算好，写入到一张数据统计结果表中，然后从该表中直接查询统计以后的结果数据，不需要进行额外的计算，提高了查询效率。

实现思路如下图所示：

![image-20230601220404916](assets/image-20230601220404916.png) 

流程说明：

1、**定时任务程序**每天凌晨2点运行一次，从order_info表中查询出前一天的订单总金额数据，然后将总金额数据写入到统计结果表中

2、当用户发起查询订单统计结果请求的时候，此时后端系统从统计结果表中查询到对应的数据返回给前端系统

3、前端系统通过Echarts的柱状图展示数据

## 2.5 Spring Task

### 2.5.1 Spring Task简介

官网地址：https://docs.spring.io/spring-framework/reference/6.1-SNAPSHOT/integration/scheduling.html

Spring Task是Spring框架中的一个定时任务调度模块，它提供了一种简单的方式来实现基于时间的调度任务。

使用Spring Task，可以通过Java代码配置或注解的方式定义定时任务，并设置任务的执行时间、间隔周期、触发条件等。当达到指定的时间或条件

时，Spring Task会自动触发任务的执行，可以执行任何有意义的操作，例如数据备份、缓存清理、邮件发送等。

Spring Task还具有以下特点：

1、简单易用：简单的XML或注解配置即可实现定时任务调度。

2、易于集成：与Spring框架集成无缝，支持Spring Boot应用快速启动。

3、可靠性高：支持在分布式环境下进行任务调度，并支持并发控制和异常管理。

4、监控调试：支持日志记录、任务执行状态监控和调试，方便排除问题。

总之，Spring Task是一种非常有效和灵活的定时任务调度方案，适用于各种规模的Web应用和后台系统。

### 2.5.2 Spring Task入门案例

需求：每5秒在控制台输出一次"HelloWorld"

代码实现：

1、定义一个任务方法，在该方法上使用**@Scheduled**注解，并通常cron属性来指定该方法的执行的时间规则

```java
@Component
@Slf4j
public class OrderStatisticsTask {

    @Scheduled(cron = "0/5 * * * * ?")  // 定义定时任务，使用@Scheduled注解指定调度时间表达式
    public void helloWorldTask() {
        log.info("HelloWorld");
    }

}

```

2、在启动类上添加**@EnableScheduling**注解开启定时任务功能

```java
...
@EnableScheduling
public class ManagerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ManagerApplication.class , args) ;
    }

}
```

启动程序进行测试。

### 2.5.3 cron表达式

**cron表达式**：

定时任务触发时间的一个[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)表达形式，分为6或7个域，每一个域代表一个含义。

cron的结构从左到右（用空格隔开）：秒 分 小时 月份中的日期 月份 星期中的日期 年份

这些字段的取值范围如下所示：

| 字段             | 允许的值                     | 允许的特殊字符 |
| ---------------- | ---------------------------- | -------------- |
| 秒（Seconds）    | 0-59                         | , - * /        |
| 分（Minutes）    | 0-59                         | , - * /        |
| 时（Hours）      | 0-23                         | , - * /        |
| 日（DayofMonth） | 1-31                         | , - * ? / L W  |
| 月（Month）      | 1-12 or JAN-DEC              | , - * /        |
| 周（DayofWeek）  | 0-6 or SUN-SAT (0表示星期天) | , - * ? / L #  |
| 年（Year）       | 留空或1970-2099              | , - * /        |

注：如日和月同时维护，例如：3 50 18 15 2 4，需要注意二月的星期四，不一定是15号，此时星期和日是有冲突的，通常需要舍掉一个，被舍掉的参

数用**?**占位

Cron表达式的时间字段除允许设置数值外，还可使用一些特殊的字符，提供列表、范围、通配符等功能。

可查看阿里云开发者社区手册：https://developer.aliyun.com/article/942392

## 2.6 订单数据统计

```sql
CREATE TABLE `order_info`  (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  `user_id` bigint NOT NULL DEFAULT 0 COMMENT '会员_id',
  `nick_name` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '昵称',
  `order_no` char(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '订单号',
  `coupon_id` bigint NULL DEFAULT NULL COMMENT '使用的优惠券',
  `total_amount` decimal(10, 2) NOT NULL DEFAULT 0.00 COMMENT '订单总额',
  `coupon_amount` decimal(10, 2) NOT NULL DEFAULT 0.00 COMMENT '优惠券',
  `original_total_amount` decimal(10, 2) NOT NULL DEFAULT 0.00 COMMENT '原价金额',
  `feight_fee` decimal(10, 2) NOT NULL DEFAULT 0.00 COMMENT '运费',
  `pay_type` tinyint NULL DEFAULT NULL COMMENT '支付方式【1->微信 2->支付宝】',
  `order_status` tinyint NOT NULL DEFAULT 0 COMMENT '订单状态【0->待付款；1->待发货；2->已发货；3->待用户收货，已完成；-1->已取消】',
  `receiver_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '收货人姓名',
  `receiver_phone` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '收货人电话',
  `receiver_tag_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '收货人地址标签',
  `receiver_province` bigint NULL DEFAULT NULL COMMENT '省份/直辖市',
  `receiver_city` bigint NULL DEFAULT NULL COMMENT '城市',
  `receiver_district` bigint NULL DEFAULT NULL COMMENT '区',
  `receiver_address` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '详细地址',
  `payment_time` datetime(0) NULL DEFAULT NULL COMMENT '支付时间',
  `delivery_time` datetime(0) NULL DEFAULT NULL COMMENT '发货时间',
  `receive_time` datetime(0) NULL DEFAULT NULL COMMENT '确认收货时间',
  `remark` varchar(500) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '订单备注',
  `cancel_time` datetime(0) NULL DEFAULT NULL COMMENT '取消订单时间',
  `cancel_reason` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '取消订单原因',
  `create_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) COMMENT '创建时间',
  `update_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) ON UPDATE CURRENT_TIMESTAMP(0) COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT 0 COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 203 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '订单' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of order_info
-- ----------------------------
INSERT INTO `order_info` VALUES (202, 30, '13121031203', '1689003850665', NULL, 1900.00, 0.00, 1900.00, 0.00, 2, 1, '珍惜每天', '13121031203', '家', NULL, NULL, NULL, '陕西省西安', '2023-07-10 15:47:10', NULL, NULL, NULL, NULL, NULL, '2023-07-10 15:44:13', '2023-07-10 15:47:12', 0);
INSERT INTO `order_info` VALUES (203, 30, '13121031203', '1689004107831', NULL, 1800.00, 0.00, 1800.00, 0.00, 2, 1, '珍惜每天', '13121031203', '家', NULL, NULL, NULL, '陕西省西安', '2023-07-10 15:48:58', NULL, NULL, NULL, NULL, NULL, '2023-07-10 15:48:30', '2023-07-10 15:49:00', 0);

-- ----------------------------
-- Table structure for order_item
-- ----------------------------
CREATE TABLE `order_item`  (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  `order_id` bigint NULL DEFAULT NULL COMMENT 'order_id',
  `sku_id` bigint NULL DEFAULT NULL COMMENT '商品sku编号',
  `sku_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '商品sku名字',
  `thumb_img` varchar(500) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '商品sku图片',
  `sku_price` decimal(10, 2) NULL DEFAULT NULL COMMENT '商品sku价格',
  `sku_num` int NULL DEFAULT NULL COMMENT '商品购买的数量',
  `create_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) COMMENT '创建时间',
  `update_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) ON UPDATE CURRENT_TIMESTAMP(0) COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT 0 COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 371 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '订单项信息' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of order_item
-- ----------------------------
INSERT INTO `order_item` VALUES (370, 202, 12, 'Apple iPhone 14 Pro Max (A2896) 暗紫色 支持移动联通电信5G 双卡双待手机白色 + 8G', 'http://192.168.136.142:9001/spzx-bucket/20230601/68cb63d9d0674e77ad3100552bbe1c7c.jpg', 1900.00, 1, '2023-07-10 15:44:13', '2023-07-10 15:44:13', 0);
INSERT INTO `order_item` VALUES (371, 203, 13, 'Apple iPhone 14 Pro Max (A2896) 暗紫色 支持移动联通电信5G 双卡双待手机白色 + 12G', 'http://192.168.136.142:9001/spzx-bucket/20230601/68cb63d9d0674e77ad3100552bbe1c7c.jpg', 1800.00, 1, '2023-07-10 15:48:30', '2023-07-10 15:48:30', 0);

-- ----------------------------
-- Table structure for order_log
-- ----------------------------
CREATE TABLE `order_log`  (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `order_id` bigint NULL DEFAULT NULL COMMENT '订单id',
  `operate_user` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '操作人：用户；系统；后台管理员',
  `process_status` int NULL DEFAULT NULL COMMENT '订单状态',
  `note` varchar(500) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备注',
  `create_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) COMMENT '创建时间',
  `update_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) ON UPDATE CURRENT_TIMESTAMP(0) COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT 0 COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 182 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '订单操作日志记录' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of order_log
-- ----------------------------
INSERT INTO `order_log` VALUES (179, 202, NULL, 0, '提交订单', '2023-07-10 15:44:13', '2023-07-10 15:44:13', 0);
INSERT INTO `order_log` VALUES (180, 202, NULL, 1, '支付宝支付成功', '2023-07-10 15:47:12', '2023-07-10 15:47:12', 0);
INSERT INTO `order_log` VALUES (181, 203, NULL, 0, '提交订单', '2023-07-10 15:48:30', '2023-07-10 15:48:30', 0);
INSERT INTO `order_log` VALUES (182, 203, NULL, 1, '支付宝支付成功', '2023-07-10 15:49:00', '2023-07-10 15:49:00', 0);

-- ----------------------------
-- Table structure for order_statistics
-- ----------------------------
CREATE TABLE `order_statistics`  (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `order_date` date NULL DEFAULT NULL COMMENT '订单统计日期',
  `total_amount` decimal(10, 2) NULL DEFAULT NULL COMMENT '总金额',
  `total_num` int NULL DEFAULT NULL COMMENT '订单总数',
  `create_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) COMMENT '创建时间',
  `update_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) ON UPDATE CURRENT_TIMESTAMP(0) COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT 0 COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 191 CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '订单统计' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of order_statistics
-- ----------------------------
INSERT INTO `order_statistics` VALUES (187, '2023-05-25', 19200.00, 120, '2023-05-30 09:43:17', '2023-05-30 09:43:17', 0);
INSERT INTO `order_statistics` VALUES (188, '2023-05-26', 20000.00, 100, '2023-05-30 09:45:16', '2023-05-30 09:45:16', 0);
INSERT INTO `order_statistics` VALUES (189, '2023-05-28', 23000.00, 2, '2023-05-30 09:49:28', '2023-05-30 10:09:57', 0);
```



### 2.6.1 数据库表介绍

统计数据结果表结构如下所示：

```sql
CREATE TABLE `order_statistics` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `order_date` date DEFAULT NULL COMMENT '订单统计日期',
  `total_amount` decimal(10,2) DEFAULT NULL COMMENT '总金额',
  `total_num` int DEFAULT NULL COMMENT '订单总数',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `is_deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记（0:不可用 1:可用）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=191 DEFAULT CHARSET=utf8mb3 COMMENT='订单统计';
```

### 2.6.3 代码实现

#### OrderStatistics

创建一个与订单统计结果表相对应的实体类：

```java
// com.atguigu.spzx.model.entity.order;
@Data
public class OrderStatistics extends BaseEntity {

    private Date orderDate;
    private BigDecimal totalAmount;
    private Integer totalNum;
    
}
```

#### OrderInfo

创建一个与订单数据库表对应的实体类：

```java
// com.atguigu.spzx.model.entity.order;
@Data
@Schema(description = "OrderInfo")
public class OrderInfo extends BaseEntity {

	private Long userId;
	private String nickName;
	private String orderNo;
	private Long couponId;
	private BigDecimal totalAmount;
	private BigDecimal couponAmount;
	private BigDecimal originalTotalAmount;
	private BigDecimal feightFee;
	private Integer payType;
	private Integer orderStatus;
	private String receiverName;
	private String receiverPhone;
	private String receiverTagName;
	private String receiverProvince;
	private String receiverCity;
	private String receiverDistrict;
	private String receiverAddress;
	private Date paymentTime;
	private Date deliveryTime;
	private Date receiveTime;
	private String remark;
	private Date cancelTime;
	private String cancelReason;

}
```

#### OrderStatisticsTask

定时任务程序代码实现：

```java
// com.atguigu.spzx.manager.task
@Component
@Slf4j
public class OrderStatisticsTask {

    @Autowired
    private OrderInfoMapper orderInfoMapper;

    @Autowired
    private OrderStatisticsMapper orderStatisticsMapper;

    @Scheduled(cron = "0 0 2 * * ?")
    public void orderTotalAmountStatistics() {
        String createTime = DateUtil.offsetDay(new Date(), -1).toString(new SimpleDateFormat("yyyy-MM-dd"));
        OrderStatistics orderStatistics = orderInfoMapper.selectOrderStatistics(createTime);
        if(orderStatistics != null) {
            orderStatisticsMapper.insert(orderStatistics) ;
        }
    }

}
```

#### OrderInfoMapper

OrderInfo持久层代码

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface OrderInfoMapper {

    // 查询指定日期产生的订单数据
    public abstract OrderStatistics selectOrderStatistics(String creatTime);

}
```

#### OrderInfoMapper.xml

在OrderInfoMapper.xml映射文件中添加如下的SQL语句：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.spzx.manager.mapper.OrderInfoMapper">

    <select id="selectOrderStatistics" resultType="com.atguigu.spzx.model.entity.order.OrderStatistics">
        select DATE_FORMAT(oi.create_time ,'%Y-%m-%d') orderDate, sum(oi.total_amount)  totalAmount , count(oi.id) totalNum
        from order_info oi where DATE_FORMAT(oi.create_time ,'%Y-%m-%d') = #{createTime}
        GROUP BY DATE_FORMAT(oi.create_time ,'%Y-%m-%d')
    </select>

</mapper>
```

#### OrderStatisticsMapper

OrderStatisticsMapper持久层代码

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface OrderStatisticsMapper {

    public abstract void insert(OrderStatistics orderStatistics);

}
```

#### OrderStatisticsMapper.xml

在OrderStatisticsMapper.xml映射文件中添加如下的SQL语句：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.spzx.manager.mapper.OrderStatisticsMapper">

    <!-- 用于select查询公用抽取的列 -->
    <sql id="columns">
        id,order_date,total_amount,total_num,create_time,update_time,is_deleted
    </sql>

    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        insert into order_statistics (
            id,
            order_date,
            total_amount,
            total_num
        ) values (
             #{id},
             #{orderDate},
             #{totalAmount},
             #{totalNum}
         )
    </insert>

</mapper>
```

## 2.7 统计查询

### 2.7.1 后端接口

#### OrderStatisticsDto

创建封装查询参数的实体类：

```java
// com.atguigu.spzx.model.dto.order;
@Data
public class OrderStatisticsDto {

    private String createTimeBegin;
    private String createTimeEnd;

}
```

#### OrderStatisticsVo

创建封装响应结果的实体类：

```java
// com.atguigu.spzx.model.vo.order;
@Data
public class OrderStatisticsVo {

    private List<String> dateList ;
    private List<BigDecimal> amountList ;
}
```

#### OrderInfoController

表现层代码实现：

```java
// com.atguigu.spzx.manager.controller;
@RestController
@RequestMapping(value="/admin/order/orderInfo")
public class OrderInfoController {

    @Autowired
    private OrderInfoService orderInfoService ;

    @GetMapping("/getOrderStatisticsData")
    public Result<OrderStatisticsVo> getOrderStatisticsData( OrderStatisticsDto orderStatisticsDto) {
        OrderStatisticsVo orderStatisticsVo = orderInfoService.getOrderStatisticsData(orderStatisticsDto) ;
        return Result.build(orderStatisticsVo , ResultCodeEnum.SUCCESS) ;
    }

}
```

#### OrderInfoService

业务层代码实现：

```java
// com.atguigu.spzx.manager.service.impl;
@Service
public class OrderInfoServiceImpl implements OrderInfoService {

    @Autowired
    private OrderStatisticsMapper orderStatisticsMapper ;

    @Override
    public OrderStatisticsVo getOrderStatisticsData(OrderStatisticsDto orderStatisticsDto) {

        // 查询统计结果数据
        List<OrderStatistics> orderStatisticsList = orderStatisticsMapper.selectList(orderStatisticsDto) ;

        //日期列表
        List<String> dateList = orderStatisticsList.stream().map(orderStatistics -> DateUtil.format(orderStatistics.getOrderDate(), "yyyy-MM-dd")).collect(Collectors.toList());

        //统计金额列表
        List<BigDecimal> amountList = orderStatisticsList.stream().map(OrderStatistics::getTotalAmount).collect(Collectors.toList());

        // 创建OrderStatisticsVo对象封装响应结果数据
        OrderStatisticsVo orderStatisticsVo = new OrderStatisticsVo() ;
        orderStatisticsVo.setDateList(dateList);
        orderStatisticsVo.setAmountList(amountList);

        // 返回数据
        return orderStatisticsVo;
    }
}
```

#### OrderStatisticsMapper

持久层代码实现：

```java
// com.atguigu.spzx.manager.mapper;
@Mapper
public interface OrderStatisticsMapper {
    List<OrderStatistics> selectList(OrderStatisticsDto orderStatisticsDto);
}
```

#### OrderStatisticsMapper.xml

在OrderStatisticsMapper.xml映射文件中添加如下的sql语句：

```xml
<resultMap id="orderStatisticsMap" type="com.atguigu.spzx.model.entity.order.OrderStatistics" autoMapping="true"></resultMap>

<select id="selectList" resultMap="orderStatisticsMap">
    select <include refid="columns" />
    from order_statistics
    <where>
        <if test="createTimeBegin != null and createTimeBegin != ''">
            and order_date >= #{createTimeBegin}
        </if>
        <if test="createTimeEnd != null and createTimeEnd != ''">
            and order_date &lt;= #{createTimeEnd}
        </if>
    </where>
    order by order_date
</select>
```

### 2.7.2 前端对接

导入课程资料中所提供的orderInfo.js和orderStatistics.vue到指定的目录下。























