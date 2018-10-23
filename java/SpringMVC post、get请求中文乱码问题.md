# SpringMVC post、get请求中文乱码问题

[TOC]

## 前言

- 无论是提交表单，还是利用URL传参时，只要参数中有中文，在程序中不做相应的处理的话，我们在java后台接收参数时就会出现中文乱码问题。

## post乱码

- 在web.xml中配置Sping的`CharacterEncodingFilter`，这是个过滤器，可以解决post请求乱码问题

```java
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## get乱码

我们知道，一般的表单提交都用post方式，但是如果向后台仅仅传递一两个参数时，我们经常将参数拼接在URL中，例如：

`http://test.corp.qunar.com:8080/rule/exportRule?business_name="直通车"`

- 在tomcat的server.xml中指定一下编码格式: (推荐)

```java
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8"/>
```

- 另外一种方法对参数进行重新编码

> String userName new String(request.getParamter("userName").getBytes("ISO8859-1"),"utf-8")

> ISO8859-1是tomcat默认编码，需要将tomcat编码后的内容按utf-8编码

