# mysql常用用法总结



- mysql修改字段类型

  ```java
  -能修改字段类型、类型长度、默认值、注释
  --对某字段进行修改
  ALTER TABLE 表名 MODIFY COLUMN 字段名 新数据类型 新类型长度  新默认值  新注释; -- COLUMN可以省略
  alter table rules modify column rate decimal(10,4) NOT NULL DEFAULT '0.0000' COMMENT '注释'; 
  ```

- mysql修改字段名

  ```java
  ALTER  TABLE 表名 CHANGE 旧字段名 新字段名 新数据类型;	 
  alter  table table1 change column1 column1 varchar(100) DEFAULT 1.2 COMMENT '注释'; -- 正常，此时字段名称没有改变，能修改字段类型、类型长度、默认值、注释
  alter  table table1 change column1 column2 decimal(10,1) DEFAULT NULL COMMENT '注释' -- 正常，能修改字段名、字段类型、类型长度、默认值、注释
  alter  table table1 change column1 column2; 
  -- 报错  
  
  ```

- decimal(10,4)   最大值是 999999.9999 

- DATE_FORMAT函数的使用   具体用法：http://www.w3school.com.cn/sql/func_date_format.asp

  ```java
   select slave_locate  from crawl_result_count where channel='flyPig' and DATE_FORMAT(count_time,'%Y-%m-%d')  =  '2018-10-30' limit 0, 20;
  ```

- substr函数用法

  ```java
  SUBSTR(str,pos,len): 从pos开始的位置，截取len个字符
  
  substr(string ,1,3) ：取string左边第1位置起，3字长的字符串。 
  所以结果为： str
  substr(string, -1,3)：取string右边第1位置起，3字长的字符串。显然右边第一位置起往右不够3字长。结果只能是： g
  substr(string, -3,3)：取string右边第3位置起，3字长的字符串。 
  结果为: ing
  SUBSTR(str,pos): pos开始的位置，一直截取到最后
  
  substr(string ,4) : 从右第4位置截取到最后 
  结果是： ing
  ```

- `CREATE TABLE tab2 LIKE tab1; ` 复制表

