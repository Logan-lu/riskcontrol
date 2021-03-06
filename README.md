# riskcontrol
## 介绍
* 轻量级，可扩展，高性能的Java实时业务风控系统
* 基于Spring boot构建，配置文件能少则少
* 使用drools规则引擎管理风控规则，规则可动态上下线，原则上可以动态配置规则。
* 使用redis、mongodb做风控计算和储存，历史事件支持水平扩展

## 准备
* mysql，数据结构在import.sql中定义了
* redis
* mongodb，建议使用分片集群

## 配置
* 项目配置：application.properties
* 日志配置：logback.xml
* 规则配置：rules/*.drl，默认配置了登录事件的部分规则

## 部署
系统默认采用jar打包和运行，建议系统搭建在多台节点上，使用反向代理做负载均衡。
### 打包
```
mvn clean install
```
### jar方式运行
```
java -jar riskcontrol-*.jar
```

### war包部署
pom.xml

```
<packaging>war</packaging>
```

## 风控请求入口
请求：http://domain/riskcontrol/req?json=事件对象的json格式，参考LoginEvent类

响应：score字段代码该事件的风险值，默认超过100分预警


## 原理
* 统计学
* 实时计算，空间换时间，降低时间复杂度
* redis，使用sortedset计算维度，条件维度作为key，统计维度作为value，时间作为score
* mongodb，使用聚合函数统计维度，比如：max，min，sum，avg，first，last，标准差，采样标准差，行为习惯等

注：redis性能优于mongodb，所以使用较多的频数计算默认在redis中运行，参考代码DimensionService.distinctCountWithRedis方法

## TODO
* 扩展黑白名单，ip，手机号，设备指纹等；
* 扩展维度信息，比如手机号地域运营商，ip地域运营商，ip出口类型，设备指纹，Referer，ua，密码hash，征信等，维度越多，可以建立规则越多，风控越精准；
* 扩展风控规则，针对需要解决的场景问题，添加特定规则，分值也应根据自身场景来调整。**当然，这将是个漫长的过程；**
* 将用户的行为过程综合考虑，建立复合场景的规则条件。比如：登录->活动->订单->支付，综合考虑；
* **减少误报，提高性能！**
