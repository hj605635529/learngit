

# on duplicate key update的使用

如果您指定了ON DUPLICATE KEY UPDATE，并且插入行后会导致在一个**UNIQUE索引或PRIMARY KEY**中出现重复值，则执行旧行UPDATE。例如，如果列a被定义为UNIQUE，并且包含值1，则以下两个语句具有相同的效果：

`insert into table(a,b,c) VALUES (1,2,3) ON DUPLICATE KEY UPDATE c = c + 1;`

`UPDATE table SET c = c + 1 WHERE a = 1;`



**注意：ON DUPLICATE KEY UPDATE只是MySQL的特有语法，并不是SQL标准语法！ **

**您可以在UPDATE子句中使用VALUES(col_name)函数从INSERT...UPDATE语句的INSERT部分引用列值,本函数特别适用于多行插入。VALUES()函数只在INSERT...UPDATE语句中有意义，其它时候会返回NULL**

```java
DROP TABLE IF EXISTS `promotion_hotels`;
CREATE TABLE `promotion_hotels`(
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',
  `hotel_seq` VARCHAR(32) NOT NULL DEFAULT '' COMMENT '酒店seq',
  `city_url` VARCHAR(64) NOT NULL DEFAULT '' COMMENT '城市',
  `promotion_level`  VARCHAR(32) NOT NULL  DEFAULT ''  COMMENT  '酒店推荐等级',
  `exposure_limit` INT  UNSIGNED NOT NULL DEFAULT '0'  COMMENT '酒店曝光限制值',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_hotel_seq` (`hotel_seq`)
)ENGINE = INNODB DEFAULT CHARSET = utf8mb4 COMMENT '推广酒店';

SELECT * FROM promotion_hotels;



```

![选区_495.png](https://i.loli.net/2018/09/25/5baa5833b98ac.png)

```java
insert into promotion_hotels(hotel_seq,city_url,promotion_level) VALUES('beijing_city_9','beijing_city','A'),('beijing_city_4','beijing_city','CCCCC') ON DUPLICATE KEY UPDATE promotion_level  = VALUES(promotion_level); 

SELECT * FROM promotion_hotels;
```

![选区_496.png](https://i.loli.net/2018/09/25/5baa58cd236db.png)



引入一个mybatis中用了on duplicate key update的例子：

```java
   <!--int insertOrUpdate(@Param("scoreDims") List<ScoreDim> scoreDims);-->
    <insert id="insertOrUpdate">
        INSERT INTO score_dim
        (id,dim_name,dim_score,remark)
        VALUES
        <foreach collection="scoreDims" item="item" index="index" open="" close="" separator=",">
            (
            #{item.id},
            #{item.dimName},
            #{item.dimScore},
            #{item.remark}
            )
        </foreach>
        ON DUPLICATE KEY
        UPDATE
        dim_name=VALUES(dim_name),
        dim_score=VALUES(dim_score),
        remark=VALUES(remark)

    </insert>
```

