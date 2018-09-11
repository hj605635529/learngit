# rank推广优化



- 由 [黄佳](http://wiki.corp.qunar.com/confluence/display/~jia.huang)创建, 最后修改于[大约1分钟以前](http://wiki.corp.qunar.com/confluence/pages/diffpagesbyversion.action?pageId=211973390&selectedPageVersions=10&selectedPageVersions=11)

转至元数据起始

- [1. pmo](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-1.pmo)
- [2. 目标](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-2.%E7%9B%AE%E6%A0%87)
- [3. 概述](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-3.%E6%A6%82%E8%BF%B0)
- [4.可配置策略整体流程图](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-4.%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%AD%96%E7%95%A5%E6%95%B4%E4%BD%93%E6%B5%81%E7%A8%8B%E5%9B%BE)
- [5. 技术方案](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-5.%E6%8A%80%E6%9C%AF%E6%96%B9%E6%A1%88)

# 1. pmo

http://pmo.corp.qunar.com/browse/DZS-28658

# 2. 目标

在不同的渠道基于差异化规则处理待推广酒店

# 3. 概述

1. 本需求暂时只需要rank一个业务组开发，没有上下游依赖。
2. 分渠道推广活动由于频率很低，酒店规则的来源是人工导入。每次更新都是一个渠道的所有酒店规则。
3. 这个策略影响List列表页酒店排序的顺序。

# 4.可配置策略整体流程图

![img](http://wiki.corp.qunar.com/confluence/download/attachments/211973390/%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF.png?version=1&modificationDate=1536310697000&api=v2)

# 5. 技术方案

1. 在SearchParam类中增加一个渠道参数 strategyChannel

   **strategyChannel枚举**



   枚举维护方式：
   //

2. 分渠道推荐数据库设计



   数据量分析
   每个渠道待推广酒店不会超过5000，所有渠道待推广酒店之和不会超过20000，同一家酒店可能在不同渠道推广， 分渠道表中存放不会超过30000条记录。

3. 从数据库中获取各个分渠道号对应的有效推荐酒店规则  数据结构为Map<Integer,List<String> > 

   **ChannelPromotionRulesDao**



   更新策略

   **ChannelPromotionRulesUpdater**

4. 新增Qconfig配置文件：strategy_channel.properties

5. 在conf目录下写一个判断策略对应生效的渠道

6. 在ConfigurableService中加入分渠道的逻辑判断。主要是过滤和插入操作， 分渠道插入的酒店scenePromotion字段设置为2,防止分页统计主渠道推广酒店误算。

- 由 [黄佳](http://wiki.corp.qunar.com/confluence/display/~jia.huang)创建, 最后修改于[大约1分钟以前](http://wiki.corp.qunar.com/confluence/pages/diffpagesbyversion.action?pageId=211973390&selectedPageVersions=10&selectedPageVersions=11)

转至元数据起始

- [1. pmo](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-1.pmo)
- [2. 目标](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-2.%E7%9B%AE%E6%A0%87)
- [3. 概述](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-3.%E6%A6%82%E8%BF%B0)
- [4.可配置策略整体流程图](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-4.%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%AD%96%E7%95%A5%E6%95%B4%E4%BD%93%E6%B5%81%E7%A8%8B%E5%9B%BE)
- [5. 技术方案](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-5.%E6%8A%80%E6%9C%AF%E6%96%B9%E6%A1%88)

# 1. pmo

http://pmo.corp.qunar.com/browse/DZS-28658

# 2. 目标

在不同的渠道基于差异化规则处理待推广酒店

# 3. 概述

1. 本需求暂时只需要rank一个业务组开发，没有上下游依赖。
2. 分渠道推广活动由于频率很低，酒店规则的来源是人工导入。每次更新都是一个渠道的所有酒店规则。
3. 这个策略影响List列表页酒店排序的顺序。

# 4.可配置策略整体流程图

![img](http://wiki.corp.qunar.com/confluence/download/attachments/211973390/%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF.png?version=1&modificationDate=1536310697000&api=v2)

# 5. 技术方案

1. 在SearchParam类中增加一个渠道参数 strategyChannel

   **strategyChannel枚举**



   枚举维护方式：
   //

2. 分渠道推荐数据库设计



   数据量分析
   每个渠道待推广酒店不会超过5000，所有渠道待推广酒店之和不会超过20000，同一家酒店可能在不同渠道推广， 分渠道表中存放不会超过30000条记录。

3. 从数据库中获取各个分渠道号对应的有效推荐酒店规则  数据结构为Map<Integer,List<String> > 

   **ChannelPromotionRulesDao**



   更新策略

   **ChannelPromotionRulesUpdater**

4. 新增Qconfig配置文件：strategy_channel.properties

5. 在conf目录下写一个判断策略对应生效的渠道

6. 在ConfigurableService中加入分渠道的逻辑判断。主要是过滤和插入操作， 分渠道插入的酒店scenePromotion字段设置为2,防止分页统计主渠道推广酒店误算。

- 由 [黄佳](http://wiki.corp.qunar.com/confluence/display/~jia.huang)创建, 最后修改于[大约1分钟以前](http://wiki.corp.qunar.com/confluence/pages/diffpagesbyversion.action?pageId=211973390&selectedPageVersions=10&selectedPageVersions=11)

转至元数据起始

- [1. pmo](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-1.pmo)
- [2. 目标](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-2.%E7%9B%AE%E6%A0%87)
- [3. 概述](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-3.%E6%A6%82%E8%BF%B0)
- [4.可配置策略整体流程图](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-4.%E5%8F%AF%E9%85%8D%E7%BD%AE%E7%AD%96%E7%95%A5%E6%95%B4%E4%BD%93%E6%B5%81%E7%A8%8B%E5%9B%BE)
- [5. 技术方案](http://wiki.corp.qunar.com/confluence/pages/viewpage.action?pageId=211973390#DZS-28658Rank%E5%8C%BA%E5%88%86%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF-5.%E6%8A%80%E6%9C%AF%E6%96%B9%E6%A1%88)

# 1. pmo

http://pmo.corp.qunar.com/browse/DZS-28658

# 2. 目标

在不同的渠道基于差异化规则处理待推广酒店

# 3. 概述

1. 本需求暂时只需要rank一个业务组开发，没有上下游依赖。
2. 分渠道推广活动由于频率很低，酒店规则的来源是人工导入。每次更新都是一个渠道的所有酒店规则。
3. 这个策略影响List列表页酒店排序的顺序。

# 4.可配置策略整体流程图

![img](http://wiki.corp.qunar.com/confluence/download/attachments/211973390/%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF.png?version=1&modificationDate=1536310697000&api=v2)

# 5. 技术方案

1. 在SearchParam类中增加一个渠道参数 strategyChannel

   **strategyChannel枚举**



   枚举维护方式：
   //

2. 分渠道推荐数据库设计



   数据量分析
   每个渠道待推广酒店不会超过5000，所有渠道待推广酒店之和不会超过20000，同一家酒店可能在不同渠道推广， 分渠道表中存放不会超过30000条记录。

3. 从数据库中获取各个分渠道号对应的有效推荐酒店规则  数据结构为Map<Integer,List<String> > 

   **ChannelPromotionRulesDao**



   更新策略

   **ChannelPromotionRulesUpdater**

4. 新增Qconfig配置文件：strategy_channel.properties

5. 在conf目录下写一个判断策略对应生效的渠道

6. 在ConfigurableService中加入分渠道的逻辑判断。主要是过滤和插入操作， 分渠道插入的酒店scenePromotion字段设置为2,防止分页统计主渠道推广酒店误算。

# 1. pmo

http://pmo.corp.qunar.com/browse/DZS-28658

# 2. 目标

在不同的渠道基于差异化规则处理待推广酒店

# 3. 概述

1. 本需求暂时只需要rank一个业务组开发，没有上下游依赖。
2. 分渠道推广活动由于频率很低，酒店规则的来源是人工导入。每次更新都是一个渠道的所有酒店规则。
3. 这个策略影响List列表页酒店排序的顺序。

# 4.可配置策略整体流程图

![大住宿技术 > DZS-28658 Rank区分渠道推广 > 渠道推广.png](http://wiki.corp.qunar.com/confluence/download/attachments/211973390/%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF.png?version=1&modificationDate=1536310697000&api=v2)

# 5. 技术方案

1. 在SearchParam类中增加一个渠道参数 strategyChannel



   枚举维护方式：
   //

2. 分渠道推荐数据库设计



   数据量分析
   每个渠道待推广酒店不会超过5000，所有渠道待推广酒店之和不会超过20000，同一家酒店可能在不同渠道推广， 分渠道表中存放不会超过30000条记录。

3. 从数据库中获取各个分渠道号对应的有效推荐酒店规则  数据结构为Map<Integer,List<String> > 



   更新策略

4. 新增Qconfig配置文件：strategy_channel.properties

5. 在conf目录下写一个判断策略对应生效的渠道

6. 在ConfigurableService中加入分渠道的逻辑判断。主要是过滤和插入操作， 分渠道插入的酒店scenePromotion字段设置为2,防止分页统计主渠道推广酒店误算。

# 1. pmo

http://pmo.corp.qunar.com/browse/DZS-28658

# 2. 目标

在不同的渠道基于差异化规则处理待推广酒店

# 3. 概述

1. 本需求暂时只需要rank一个业务组开发，没有上下游依赖。
2. 分渠道推广活动由于频率很低，酒店规则的来源是人工导入。每次更新都是一个渠道的所有酒店规则。
3. 这个策略影响List列表页酒店排序的顺序。

# 4.可配置策略整体流程图

![大住宿技术 > DZS-28658 Rank区分渠道推广 > 渠道推广.png](http://wiki.corp.qunar.com/confluence/download/attachments/211973390/%E6%B8%A0%E9%81%93%E6%8E%A8%E5%B9%BF.png?version=1&modificationDate=1536310697000&api=v2)

# 5. 技术方案

1. 在SearchParam类中增加一个渠道参数 strategyChannel



   枚举维护方式：
   //

2. 分渠道推荐数据库设计



   数据量分析
   每个渠道待推广酒店不会超过5000，所有渠道待推广酒店之和不会超过20000，同一家酒店可能在不同渠道推广， 分渠道表中存放不会超过30000条记录。

3. 从数据库中获取各个分渠道号对应的有效推荐酒店规则  数据结构为Map<Integer,List<String> > 



   更新策略

4. 新增Qconfig配置文件：strategy_channel.properties

5. 在conf目录下写一个判断策略对应生效的渠道

6. 在ConfigurableService中加入分渠道的逻辑判断。主要是过滤和插入操作， 分渠道插入的酒店scenePromotion字段设置为2,防止分页统计主渠道推广酒店误算。