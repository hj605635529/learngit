# Mybatis用法总结

[TOC]

## 概念

### mysql查询

- 对于查询不是vachar字段 where条件后面可以不用加单引号。

  > select * from promotion_rule where id = 14;

- 对于查询是vachar字段 where条件后面一定要加单引号。

  >select * from promotion_rule where hotel_name = '杭州123234';

  

### {}占位符:占位 

如果传入的是基本类型,那么#{}中的变量名称可以随意写 
如果传入的参数是pojo类型,那么#{}中的变量名称必须是pojo中的属性.属性.属性…

${}拼接符:字符串原样拼接 
如果传入的是基本类型,那么${}中的变量名必须是value 
如果传入的参数是pojo类型,那么${}中的变量名称必须是pojo中的属性.属性.属性… 



```JAVA
<select id="queryHotelRule" resultMap="HotelRule">
    select
    id,
    hotel_seq,
    hotel_name,
    FROM  qpms.promotion_rule
    ORDER BY #{dgm.sort} #{dgm.order}    //需要改成${dgm.sort} ${dgm.order}
    limit ${(dgm.page-1)*dgm.rows}, #{dgm.rows}
</select>
```

在动态 SQL 解析阶段， #{ } 和$ { } 会有不同的表现,# {} 在动态解析的时候， 会解析成一个参数标记符,

$ {}在动态解析的时候，会将我们传入的参数当做String字符串(我这是整数？？)填充到我​们的语句中,预编译之前的 SQL 语句已经不包含变量了，完全已经是常量数据了。相当于我们普通没有变量的sql了

```java
select id, hotel_seq, hotel_name, city_url, start_time, end_time, business_name, rate, submitter, submit_time FROM qpms.promotion_rule ORDER BY ? ? limit 0, ? 
```

**就是order by 传参的时候要用$符号，用#不生效**（尽管你在控制台看到的生成语句是正确的，但是结果就是不对！！^_^，这个记住就好了。）从图中也能看出没有按id逆序排列,

![选区_683.png](https://i.loli.net/2018/10/23/5bceff4e32cc5.png)

**因为用#{ }会多个单引号'  ' ** 类似：select id,hotel_seq from promotion_rule  order by 'id' limit 0, 20; 没有按id排序。

![选区_686.png](https://i.loli.net/2018/10/23/5bcf044fe7f93.png)







## 实战



```java
/**
 * 通过酒店Seq 查询出对应的挂牌信息以及挂牌的时间
 * @param seqList
 * @return
 */
List<BoardHotelLevel> queryLevelBySeq( @Param("seqList") List<String> seqList, @Param("current") String currentTime);
```

```java
<resultMap id="BoardHotelLevel" type="com.qunar.mobile.web.po.BoardHotelLevel">
    <result column="hotel_seq" property="hotelSeq"/>
    <result column="from_date" property="fromDate"/>
    <result column="level" property="level"/>
</resultMap>

<select id="queryLevelBySeq" resultMap="BoardHotelLevel" >
    SELECT
    hotel_seq,
    from_date,
    level
    from mobrank.hotel_board
    <where>
        <![CDATA[from_date <= #{current}]]> AND  to_date >= #{current}
        <if test="seqList != null">
            AND hotel_seq in
            <foreach collection="seqList" item="seq" index = "index" open="(" 		              close=")" separator=",">
                #{seq}
            </foreach>
        </if>
    </where>
</select>
```