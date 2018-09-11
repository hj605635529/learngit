# jackson实现Object对象与json串的互相转换

[TOC]

## jar包
如果你需要使用jackson，你必须得导入相应的架包，有如下三个包jackson-annotations；jackson-core；jackson-databind。
```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.4.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.4.2</version>
</dependency>
```

## 对象与json互相转换

```java
package com.jackson;

import com.fasterxml.jackson.databind.annotation.JsonSerialize;

import java.util.Date;

/**
 * @author jia.huang
 * @create 18-9-12 上午1:18
 */
public class UserBean {
   private int userId;
   private String userName;
   private String password;
   private String email;

    //日期转换
   @JsonSerialize(using = JsonDateFormatFull.class)
   private Date createDate;

   public UserBean() {
   }

   public UserBean(int userId, String userName, String password, String email) {
      this.userId = userId;
      this.userName = userName;
      this.password = password;
      this.email = email;
   }

   public int getUserId() {
      return userId;
   }

   public void setUserId(int userId) {
      this.userId = userId;
   }

   public String getUserName() {
      return userName;
   }

   public void setUserName(String userName) {
      this.userName = userName;
   }

   public String getPassword() {
      return password;
   }

   public void setPassword(String password) {
      this.password = password;
   }

   public String getEmail() {
      return email;
   }

   public void setEmail(String email) {
      this.email = email;
   }

   public Date getCreateDate() {
      return createDate;
   }

   public void setCreateDate(Date createDate) {
      this.createDate = createDate;
   }

   @Override
   public String toString() {
      return "UserBean{" + "userId=" + userId + ", userName='" + userName + '\'' + ", password='" + password + '\''
            + ", email='" + email + '\'' + ", createDate=" + createDate + '}';
   }
}
```



```java
package com.jackson;

import com.fasterxml.jackson.databind.annotation.JsonSerialize;

import java.util.Date;
import java.util.List;

/**
 * @author jia.huang
 * @create 18-9-12 上午1:19
 */
public class DeptBean {
   private int deptId;
   private String deptName;
   private List<UserBean> userBeanList;

   @JsonSerialize(using = JsonDateFormatFull.class)
   private Date createDate;

   public DeptBean() {
   }

   public DeptBean(int deptId, String deptName, List<UserBean> userBeanList) {
      this.deptId = deptId;
      this.deptName = deptName;
      this.userBeanList = userBeanList;
   }

   public int getDeptId() {
      return deptId;
   }

   public void setDeptId(int deptId) {
      this.deptId = deptId;
   }

   public String getDeptName() {
      return deptName;
   }

   public void setDeptName(String deptName) {
      this.deptName = deptName;
   }

   public List<UserBean> getUserBeanList() {
      return userBeanList;
   }

   public void setUserBeanList(List<UserBean> userBeanList) {
      this.userBeanList = userBeanList;
   }

   public Date getCreateDate() {
      return createDate;
   }

   public void setCreateDate(Date createDate) {
      this.createDate = createDate;
   }

   @Override
   public String toString() {
      return "DeptBean{" + "deptId=" + deptId + ", deptName='" + deptName + '\'' + ", userBeanList=" + userBeanList
            + ", createDate=" + createDate + '}';
   }
}
```



```java
package com.jackson;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @author jia.huang
 * @create 18-9-12 上午1:48
 */

/**
 * jackson日期转换工具类
 */
public class JsonDateFormatFull extends JsonSerializer<Date> {

   @Override
   public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider)
         throws IOException, JsonProcessingException {

      SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
      String format1 = format.format(date);
      jsonGenerator.writeString(format1);
   }
}
```

**带日期的实体对象与json转换**

​        jackson实现带日期的实体对象与json转换有两种方法：

- 将实体对象中的日期对象定义为String型，在使用的时候再将String型转换为Date型使用，其他就无需修改。
- 当实体对象中的日期对象定义为Date型，就需要通过集成JsonSerializer<Date>对象完成日期的转换。

```java
package com.jackson;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.joda.time.DateTime;
import org.junit.Test;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;


/**
 * @author jia.huang
 * @create 18-9-12 上午12:46
 */
public class JacksonDemo {

    @Test
    public void test1() throws IOException {
        UserBean userBean1 = new UserBean(1, "liubei", "123", "liubei@163.com");
        userBean1.setCreateDate(DateTime.now().toDate());

        UserBean userBean2 = new UserBean(2, "guanyu", "123", "guanyu@163.com");
        userBean2.setCreateDate(DateTime.now().toDate());

        UserBean userBean3 = new UserBean(3, "zhangfei", "123", "zhangfei@163.com");
      userBean3.setCreateDate(DateTime.now().toDate());

        List<UserBean> userBeans = new ArrayList<>();
        userBeans.add(userBean1);
        userBeans.add(userBean2);
        userBeans.add(userBean3);

        DeptBean deptBean = new DeptBean(1, "sanguo", userBeans);
        deptBean.setCreateDate(DateTime.now().toDate());

        /**
         * ObjectMapper是JSON操作的核心，Jackson的所有JSON操作都是在ObjectMapper中实现。
         * ObjectMapper有多个JSON序列化的方法，可以把JSON字符串保存File、OutputStream等不同的介质中。 writeValue(File arg0, Object
         * arg1)把arg1转成json序列，并保存到arg0文件中。 writeValue(OutputStream arg0, Object arg1)把arg1转成json序列，并保存到arg0输出流中。
         * writeValueAsBytes(Object arg0)把arg0转成json序列，并把结果输出成字节数组。 writeValueAsString(Object
         * arg0)把arg0转成json序列，并把结果输出成字符串。
         */
        ObjectMapper mapper = new ObjectMapper();

        // 对象转json
        String userBean1ToJson = mapper.writeValueAsString(userBean1);
        String deptBeanToJson = mapper.writeValueAsString(deptBean);

        System.out.println(userBean1ToJson);
        System.out.println(deptBeanToJson);

        // json转实体对象
        UserBean userBean = mapper.readValue(userBean1ToJson, UserBean.class);
        DeptBean deptBean1 = mapper.readValue(deptBeanToJson, DeptBean.class);

        System.out.println("================json转实体对象==============");
        System.out.println(userBean);
        System.out.println(deptBean1);

        // List转json字符串
        String listToJson = mapper.writeValueAsString(userBeans);
        System.out.println("=================List转json串===================");
        System.out.println(listToJson);

        // 数值json转List 当需要转换成List或Map时，就需要使用TypeReference<T>范型了
        List<UserBean> jsonToUserBeans = mapper.readValue(listToJson, new TypeReference<List<UserBean>>() {} );
      System.out.println("==========数组json转List========================");
      System.out.println(jsonToUserBeans);
    }
}
```



结果：

```java
{"userId":1,"userName":"liubei","password":"123","email":"liubei@163.com","createDate":"2018-09-12"}
{"deptId":1,"deptName":"sanguo","userBeanList":[{"userId":1,"userName":"liubei","password":"123","email":"liubei@163.com","createDate":"2018-09-12"},{"userId":2,"userName":"guanyu","password":"123","email":"guanyu@163.com","createDate":"2018-09-12"},{"userId":3,"userName":"zhangfei","password":"123","email":"zhangfei@163.com","createDate":"2018-09-12"}],"createDate":"2018-09-12"}
================json转实体对象==============
UserBean{userId=1, userName='liubei', password='123', email='liubei@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}
DeptBean{deptId=1, deptName='sanguo', userBeanList=[UserBean{userId=1, userName='liubei', password='123', email='liubei@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}, UserBean{userId=2, userName='guanyu', password='123', email='guanyu@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}, UserBean{userId=3, userName='zhangfei', password='123', email='zhangfei@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}], createDate=Wed Sep 12 08:00:00 CST 2018}
=================List转json串===================
[{"userId":1,"userName":"liubei","password":"123","email":"liubei@163.com","createDate":"2018-09-12"},{"userId":2,"userName":"guanyu","password":"123","email":"guanyu@163.com","createDate":"2018-09-12"},{"userId":3,"userName":"zhangfei","password":"123","email":"zhangfei@163.com","createDate":"2018-09-12"}]
==========数组json转List========================
[UserBean{userId=1, userName='liubei', password='123', email='liubei@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}, UserBean{userId=2, userName='guanyu', password='123', email='guanyu@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}, UserBean{userId=3, userName='zhangfei', password='123', email='zhangfei@163.com', createDate=Wed Sep 12 08:00:00 CST 2018}]

```

