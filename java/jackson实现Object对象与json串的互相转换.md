# jackson实现Object对象与json串的互相转换

[TOC]

## 0.json学习

使用中，json有以下三种结构：

对象：{"name":"Michael","age":24}

数组：比如：[{"name":"Michael","age":24},{"name":"Tom","age":25}]

值：{"name":"Michael", "birthday":{"month":8,"day":26}}，类似于对象嵌套对象.

 **疑惑{}和[]形式的区别**：

大括号{}用来描述一组“不同类型的无序键值对集合”（每个键值对可以理解为OOP的属性描述），方括号[]用来描述一组“相同类型的有序数据集合”（可对应OOP的数组）

名称始终需要加上双引号，多个键值对使用逗号隔开。

## 1.jar包

如果你需要使用jackson，你必须得导入相应的架包，有如下三个包jackson-annotations；jackson-core；jackson-databind。只要在maven项目中导入jacson-dateabind，其他两个就会导入。
```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```

![image-20190116155933023](https://ws1.sinaimg.cn/large/006tNc79ly1fz8hf7o6h8j30hc09774h.jpg)



## 2.Api介绍

```JAVA
ObjectMapper是JSON操作的核心，Jackson的所有JSON操作都是在ObjectMapper中实现。ObjectMapper有多个JSON序列化的方法，可以把JSON字符串保存File、OutputStream等介质中。
---------------------------------------------------------------------------------------
						Object对象转json字符串
writeValue(OutputStream arg0, Object arg1)把arg1转成json，并保存到arg0输出流中。
writeValue(File arg0, Object arg1)把arg1转成json序列，并保存到arg0文件中。        
writeValueAsBytes(Object arg0)把arg0转成json序列，并把结果输出成字节数

writeValueAsString(Object arg0)把arg0转成json序列，并把结果输出成字符串。(重点)

---------------------------------------------------------------------------------------
                        json字符串转Object
public <T> T readValue(String content, Class<T> valueType)   json转普通的bean
public <T> T readValue(String content, TypeReference valueTypeRef) 复杂对象, List, Map
    
```

## 3.用法

```java
package com.qunar.hotel.bean;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;


/**
 * 测试jackson
 *
 * @author jia.huang
 * @date 2019/1/16
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserBean {

    private int userId;

    private String userName;

    private String password;

    private String email;
}
```
```
package com.qunar.hotel.bean;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

/**
 * 测试jackson
 *
 * @author jia.huang
 * @date 2019/1/16
 */
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class DeptBean {

    private int deptId;

    private String deptName;

    private List<UserBean> userBeanList;
}
```

```java
package com.qunar.hotel.utils;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;

/**
 * json字符串与对象之间的转换工具
 *
 * @author jia.huang
 * @date 2019/1/16
 */
@Slf4j
public class JsonUtils {
    private static ObjectMapper objectMapper;

    /**
     * 把JavaBean转换成Json字符串
     *
     * @param object
     * @return
     */
    public static String ObjectToJson(Object object){
        if (objectMapper == null){
            objectMapper = new ObjectMapper();
        }

        try {
            return  objectMapper.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            log.error("json序列化失败", e);
        }

        return null;
    }

    /**
     * 把json字符串转换为相应的JavaBean对象
     * 转换为普通JavaBean：readValue(json,UserBean.class)
     *
     * @param json
     * @param valueType
     * @param <T>
     * @return
     */
    public static<T>  T JsonToObject(String json, Class<T> valueType){
        if (objectMapper == null){
            objectMapper = new ObjectMapper();
        }

        try {
            return  objectMapper.readValue(json, valueType);
        } catch (IOException e) {
            log.error("json反序列化失败", e);
        }

        return null;

    }

    /**
     * 把json字符串转换为相应的List，Map复杂对象
     * 转换为普通JavaBean：readValue(json,new TypeReference<List<UserBean>>(){})
     *
     * @param json
     * @param valueTypeRef
     * @param <T>
     * @return
     */
    public static<T>  T JsonToObject(String json, TypeReference<T> valueTypeRef){
        if (objectMapper == null){
            objectMapper = new ObjectMapper();
        }

        try {
            return  objectMapper.readValue(json, valueTypeRef);
        } catch (IOException e) {
            log.error("json反序列化失败", e);
        }

        return null;

    }

}
```
```java
@Test
public void jsonTest(){
   UserBean userBean1 = UserBean.builder().userId(1).userName("huangjia").email("123@163.com").password("123").build();
   UserBean userBean2 = UserBean.builder().userId(1).userName("huangjia2").email("1234@163.com").password("1234").build();
   List<UserBean> userBeanList = Lists.newArrayList();
   userBeanList.add(userBean1);
   userBeanList.add(userBean2);

   DeptBean deptBean = new DeptBean(1, "hotel", userBeanList);

  //简单的Bean对象转换
    String userToJson = JsonUtils.ObjectToJson(userBean1);
    String deptToJson = JsonUtils.ObjectToJson(deptBean);

    UserBean userBean = JsonUtils.JsonToObject(userToJson, UserBean.class);
    DeptBean deptBean1 = JsonUtils.JsonToObject(deptToJson, DeptBean.class);
    System.out.println("=====简单的json对象转换=====");
    System.out.println(userBean);
    System.out.println(deptBean1);

    //List转json
    String s = JsonUtils.ObjectToJson(userBeanList);
    //生成一个匿名内部类实现TypeReference抽象类
    List<UserBean> userBeanList1 = JsonUtils.JsonToObject(s, new TypeReference<List<UserBean>>(){});
    System.out.println("=======list转json==========");
    System.out.println(userBeanList1);
}
```

结果：

![image-20190116161347069](https://ws4.sinaimg.cn/large/006tNc79ly1fz8htztmcvj3186032gmi.jpg)

## 4.日期属性转换

**带日期的实体对象与json转换**

​        jackson实现带日期的实体对象与json转换有两种方法：

- 将实体对象中的日期对象定义为String型，在使用的时候再将String型转换为Date型使用，其他就无需修改。
- 当实体对象中的日期对象定义为Date型，就需要通过集成JsonSerializer<Date>对象完成自定义输出格式日期的转换。

![image-20190116164753764](https://ws3.sinaimg.cn/large/006tNc79ly1fz8ithj1nhj30ts0cjjst.jpg)

![image-20190116164826366](https://ws4.sinaimg.cn/large/006tNc79ly1fz8iu1rw5hj30vz0d00ul.jpg)

![image-20190116164857965](https://ws2.sinaimg.cn/large/006tNc79ly1fz8iulgayxj30zn09dgn3.jpg)

![image-20190116164915547](https://ws3.sinaimg.cn/large/006tNc79ly1fz8iuwrvndj313e06a76u.jpg)

## 5.jackson与fastjson对比

https://www.zhihu.com/question/44199956



