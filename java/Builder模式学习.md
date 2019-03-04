# Builder模式学习

[TOC]

## 前提

当 在设计模式中对Builder模式的定义是用于构建复杂对象的一种模式，所构建的对象往往需要多步初始化或赋值才能完成。那么，在实际的开发过程中，我们哪些地方适合用到Builder模式呢？其中使用Builder模式来替代多参数构造函数是一个比较好的实践法则。
 我们常常会面临编写一个这样的实现类(假设类名叫DoDoContact)，这个类拥有多个构造函数，
 DoDoContact(String name);
 DoDoContact(String name, int age);
 DoDoContact(String name, int age, String address);
 DoDoContact(String name, int age, String address, int cardID);
 这样一系列的构造函数主要目的就是为了提供更多的客户调用选择，以处理不同的构造请求。这种方法很常见，也很有效力，但是它的缺点也很多：

- 不同参数，不同顺序有不同的组合方式， 我们需要写很多类似没意义代码
- 当参数的个数达到四个或者更多的时候，你是否会经常忘记这几个参数之间的次序了呢？

Builder模式优点：

- 自己去组装所需要的构造参数和顺序



## 要点

> - 定义一个静态内部类Builder，内部的成员变量和外部类一样 
> - Builder类通过一系列的方法用于成员变量的赋值，并返回当前对象本身（this）。 
> - Builder类提供一个build方法用于创建对应的外部类，该方法内部调用了外部类的一个私有构造函数，该构造函数的参数就是内部类Builder。 



## 代码演示



```java
public class RequestCondition {

    private String cityURL;
    private boolean channelCity;
    private boolean containsCoorFilter;
    private boolean isKeyLocation;
    private boolean isKeySearch;
    private boolean isTreadingAreaSearch;
    private String rootCityurl;

    private RequestCondition(RequestConditionBuilder builder) {
        this.cityURL = builder.cityURL;
        this.channelCity = builder.channelCity;
        this.containsCoorFilter = builder.containsCoorFilter;
        this.isKeyLocation = builder.isKeyLocation;
        this.isKeySearch = builder.isKeySearch;
        this.isTreadingAreaSearch = builder.isTreadingAreaSearch;
        this.rootCityurl = builder.rootCityurl;
    }

    public static class RequestConditionBuilder {
        private String cityURL;
        private boolean channelCity;
        private boolean containsCoorFilter;
        private boolean isKeyLocation;
        private boolean isKeySearch;
        private boolean isTreadingAreaSearch;
        private String rootCityurl;

        private RequestConditionBuilder() {
        }

        public static RequestConditionBuilder create() {
            return new RequestConditionBuilder();
        }

        public RequestConditionBuilder cityURL(String cityURL) {
            this.cityURL = cityURL;
            return this;
        }

        public RequestConditionBuilder channelCity(boolean channelCity) {
            this.channelCity = channelCity;
            return this;
        }

        public RequestConditionBuilder containsCoorFilter(boolean containsCoorFilter) {
            this.containsCoorFilter = containsCoorFilter;
            return this;
        }

        public RequestConditionBuilder isKeyLocation(boolean isKeyLocation) {
            this.isKeyLocation = isKeyLocation;
            return this;
        }

        public RequestConditionBuilder isKeySearch(boolean isKeySearch) {
            this.isKeySearch = isKeySearch;
            return this;
        }

        public RequestConditionBuilder isTreadingAreaSearch(boolean isTreadingAreaSearch) {
            this.isTreadingAreaSearch = isTreadingAreaSearch;
            return this;
        }

        public RequestConditionBuilder rootCityurl(String rootCityurl) {
            this.rootCityurl = rootCityurl;
            return this;
        }

        public RequestCondition build() {
            return new RequestCondition(this);
        }
    }   //内部类结束

    public boolean isChannelCity() {
        return channelCity;
    }

    public boolean isContainsCoorFilter() {
        return containsCoorFilter;
    }

    public boolean isKeyLocation() {
        return isKeyLocation;
    }

    public boolean isKeySearch() {
        return isKeySearch;
    }

    public boolean isTreadingAreaSearch() {
        return isTreadingAreaSearch;
    }

    public String getCityURL() {
        return cityURL;
    }

    public String getRootCityurl() {
        return rootCityurl;
    }
}
```



客户端代码：

```java
private TuiguangContext buildTuiguangContext(RequestParam requestParam, HotelSearcher hotelSearcher) {
    RequestCondition requestCondition = RequestCondition.RequestConditionBuilder.create()
            .cityURL(requestParam.getCityUrl())
            .channelCity(requestParam.isChannelCity())
            .isKeyLocation(hotelSearcher.isLocation())
            .isKeySearch(requestParam.isKeySearch())
            .isTreadingAreaSearch(StrategyHelper.isAreaSearch(hotelSearcher))
            .containsCoorFilter(requestParam.isContainsCoordinateFilter())
            .rootCityurl(context.getContextAttr(BusinessAttrKey.ROOT_CITYURL.getAttrKey()))
            .build();
```



## 参考(重点看看这篇文章)

https://blog.csdn.net/u011240877/article/details/53248917