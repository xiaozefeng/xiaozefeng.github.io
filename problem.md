# 遇到的问题

### Feign 整合 Hystrix 后首次请求失败

Hystrix 默认超时时间为 1秒，由于Spring 懒加载机制，首次请求往往比较慢，特别是配置低的机器

**解决方案：**

1. 设置Hystrix超时时间
2. 不设置Hystrix超时
3. 禁用Hystrix



### 哪些情况下Hystrix会fallback

1. 主动抛出异常  (需要注意的是如果抛出 HystrixBadRequestException是不会触发fallback的)
2. 调用超时
3. 线程池拒绝 (任务队列长度为-1,所以只要线程全部被占有了就会直接拒绝)
4. 断路器打开



### MySQL `!=` 会过滤NULL值

```mysql
-- 例如
select * from tb_user where code != 'xxx'; -- 如果 code字段有null值会被过滤掉

-- 解决方法
select * from tb_user where code != 'xxx' or code is null; -- 效率不好

select * from tb_user where ifnull(code, '') != 'xxx';
```



### MySQL 排序后分组

需求:  一个商品有多个sku, 查询商品列表，同时列出这个商品最低价格的sku, 需要向对sku 排序，然后根据 商品id分组

#### 第一种方式 使用min函数

```mysql
select a.id  productId,
       a.name,
       a.description,
       b.img mainImg,
       b.origin_price,
       min(b.selling_price) sellingPrice,
       b.stock
from tb_product a
         inner join tb_product_sku b on a.id = b.product_id
         inner join tb_product_shop_relation d on a.id = d.product_id
where a.category_id = 6 and d.shop_id = 1
group by b.product_id;

-- 这种方式的缺点是查询到的skuId 是不准确的，所以在不需要skuId的情况下使用

```

#### 第二种方式 子查询

```mysql
select tmp.productId, tmp.name, tmp.mainImg, tmp.origin_price, tmp.selling_price, tmp.stock, tmp.skuId
from (
         select a.id productId, a.name, b.img mainImg, b.origin_price, b.selling_price, b.stock, b.id skuId, a.category_id
         from tb_product a
                  inner join tb_product_sku b on a.id = b.product_id
         order by b.selling_price
         limit 100) tmp
inner join tb_product_shop_relation d on tmp.productId = d.product_id
where  tmp.category_id = 6
group by tmp.productId
-- 在子查询中 先排序， 然后在外层再 group by 
-- 需要注意的是 子查询的中的 limit 在 5.7 以后是必要的，否则 5.7会自动优化，去掉里面的排序
-- 这种方式的缺点是 limit 的值不好取，当数据很大的时候，会漏数据
```

#### 第三种 利用 group_concat 函数的排序

```mysql
select a.id  productId,
       a.name,
       b.img mainImg,
       b.origin_price,
       b.selling_price,
       b.stock,
       b.id  skuId,
from tb_product a
         inner join tb_product_sku b on a.id = b.product_id
         inner join tb_product_shop_relation d on a.id = d.product_id
         inner join (
    select substring_index(group_concat(c.id order by c.selling_price asc), ',', 1) skuId
    from tb_product_sku c
    group by c.product_id) t1 on t1.skuId = b.id
where a.category_id = 6 and d.shop_id = 10;

desc tb_product_shop_relation;

-- group_concat 会把sku的id根据 selling_price 排序, 然后变成 1,2,3的形式
-- substring_index 再根据 ,  分割，并取第一个元素, 形成多行数据
```





### Mybati Plus 开发中遇到的问题

#### 1. 查询时 LocalDateTime 不能自动 set 值

其实 mp 本身是支持的，是由于 `druid` 的问题, 将druid 的版本升级到 `1.1.21`

```xml
 <dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.21</version>
</dependency>
```



#### 2. MP 插入数据和更新数据时，要写创建时间和更新时间

首先要在字段上加 `FieldFill.INSERT` `FieldFill.INSERT_UPDATE`

```java
  @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedTime;

    @TableField(fill = FieldFill.INSERT)
    private String createdBy;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private String updatedBy;

```

编写配置类，实现 `MetaObjectHandler`

```java

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import com.otmgroup.mall.product.bean.vo.core.BaseLoginInfo;
import com.otmgroup.mall.product.bean.vo.core.RequestHolder;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        LocalDateTime now = LocalDateTime.now();
        this.strictInsertFill(metaObject, "createdTime", LocalDateTime.class, now);
        this.strictInsertFill(metaObject, "updatedTime", LocalDateTime.class, now);
        BaseLoginInfo loginInfo = RequestHolder.get();
        this.strictInsertFill(metaObject, "createdBy", String.class, loginInfo.getUid());
        this.strictInsertFill(metaObject, "updatedBy", String.class, loginInfo.getUid());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        LocalDateTime now = LocalDateTime.now();
        // 这里一定要用 setFieldValByName 否则更新时不会写值
        this.setFieldValByName("updatedTime", now, metaObject);
        BaseLoginInfo loginInfo = RequestHolder.get();
        this.setFieldValByName("updatedBy", loginInfo.getUid(), metaObject);
    }
}
```

#### 3. mp 自定义查询语句时，如果传入的参数是一个对象，而这个对象不是entity

这种情况下，需要在入参处 加入 @Param("search"), 然后再 xml 文件中通过， #{search.keyword} 来获取参数的值

