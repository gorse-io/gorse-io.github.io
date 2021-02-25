# 第一章：系统设计

## 分布式数据库

分布式数据库负责存储推荐系统中的所有数据，推荐使用分布式NoSQL存储，例如Redis、HBase、Cassandra、mongodb等。对于任何数据库，只要实现了正确的接口即可使用。关系型数据库肯定也是没有问题的，但是分布式关系型数据库成本高昂，强一致性的要求对于推荐系统来说没有必要。系统目前支持数据库情况如下：

| 数据库 | 已实现接口 | 已添加集成测试 |
|-|-|-|
| Redis | ✔️ | ✔️ |
| MySQL | ✔️ | ✔️ |
| TiDB | ✔️ |  |

## 服务节点

服务节点的作用有两点：

- 将整个推荐系统以RESTful API的形式暴露给外部服务。
- 完成推荐结果的最终去重和排序。

服务节点是无状态的，它水平扩展能力取决于分布式数据库的水平扩展能力以及单机是否能够容纳排序模型。服务节点主要提供了下表所示的REST风格API接口，这是外部服务看到的推荐系统形态。其中出现的一些概念定义如下：

- **事件**：指用户和物品之间发生的交互事件，例如购买、点赞、收藏等正反馈事件，也可以是“踩”等负反馈事件。
- **相似**：指物品之间用户重合度比较高。
- **最新**：指时间戳最新。
- **最热**：指用户量最高。


| URL | 方法 | 描述 |
|-|-|-|
| user | POST | 添加一个用户 |
user/\{uid\} & GET & 获取一个用户 \\
users & POST & 添加一组用户 \\
users & GET & 获取一组用户 \\
user/\{uid\} & DELETE & 删除一个用户 \\
user/\{uid\}/feedback & GET & 获取一个用户所有反馈 \\
user/\{uid\}/ignore & GET & 获取一个用户忽略列表 \\
user/\{uid\}/recommend & GET & 获取一个用户推荐列表 \\
\hline
item & POST & 添加一个物品 \\
item/\{iid\} & GET & 获取一个物品 \\
items & POST & 添加一组物品 \\
items & GET & 获取一组物品 \\
item/\{iid\} & DELETE & 删除一个物品 \\
item/\{iid\}/feedback & GET & 获取一个物品所有的反馈 \\
item/\{iid\}/neighbors & GET & 获取一个物品相似物品 \\
\hline
feedback & POST & 添加一组反馈 \\
feedback & GET & 获取全部的反馈 \\
\hline
labels & GET & 获取所有标签 \\
label/\{label\} & GET & 获取标签下的所有物品 \\
\hline
latest & GET & 获取全局最新物品 \\
latest/\{label\} & GET & 获取标签下的最新物品 \\
\hline
popular & GET & 获取全局最热物品 \\
popular/\{label\} & GET & 获取标签下的最热物品 \\
