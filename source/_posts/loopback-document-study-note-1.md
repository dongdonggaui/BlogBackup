---
title: loopback document 学习笔记（一）
date: 2016-02-24 16:04:18
tags: 
		- loopback
		- 笔记
---

loopback 是 IBM 的 StrongLoop 团队的一款用于快速搭建 web API 的框架，简单易用，这里记录一下学习过程。
<!--more-->

1. `Model`的有效性验证
	 - 通过在`model-name.js`文件中设置验证逻辑，验证方法主要有：
  		- `validatesAbsenceOf`：验证model不能包含的属性
  		- `validatesExclusionOf`：验证属性的非法取值范围
  		- `validatesFormatOf`：验证属性必须符合的格式，参数为属性名和格式验证，格式验证为包含正则表达式和message的object
  		- `validatesInclusionOf`：验证属性的有效取值范围
  		- `validatesLengthOf`：验证属性的长度，有`min`、`max`和`is`三种验证方式
  		- `validatesNumericalityOf`：验证属性是整型还是浮点型（Integer or Number）
  		- `validatesPresenceOf`：验证model必须要有的属性
  		- `validatesUniquenessOf`：验证属性的唯一性
  	- 通过调用`model`的`isValid()`方法来强制验证，通过调用`validate()`和`validateAsync()`方法来进行自定义的验证逻辑
  	- 当`model`创建和更新时，会自动进行验证
  	- 本地化验证消息，提供`error.details.codes`让客户端进行本地化适配

2. Operation hooks
	- 由执行`CRUD`操作的方法触发
	- 使用`operation hooks`，`model hooks`已经废弃
	- `Operation hook context object`是`operation hooks`特有的，和`remote hooks context`和`loopback.getCurrentContext()`不是同一个
	- 所有`hook`和`operation`的`operation hook context`共有属性：
	 	- `Model`：context.Model
	 	- `options`：自定义操作选项，可用来传递自定义参数
	 	- `hookState：hook` operation之间用来共享的数据
 	- hook和operation特有的属性：
 		- `instance`：当且仅当`operation`只对一个`instance`的全部属性进行了`update/create/delete`操作时才有此属性
 		- `where+data`：当`operation`对多个`instance`进行了操作或对一个`instance`的部分`property`进行了操作时，`context`会提供一个`where filter`和一个包含了受到了影响的属性的`data`对象
 		- `isNewInstance`：用来标识`CREATE`和`UPDATE`操作的flg
 		- `currentInstance`：`context`为使单个`instance`的部分属性受到影响的操作提供的属性，包含一个受到影响的`model instance`，框架推荐将此属性作为`read only`对待
	- Loopback提供的Operation hooks主要有：
		- `access`：当查询Model的方法调用时触发，用来修改查询，如添加额外的查询条件，context包含属性有`Model`和`query`
		- `before save`：在Model instance被创建或更新前触发
			- PersistedModel触发before save的方法有：`create()`、`upsert()`、`findOrCreate()`、`updateAll()`、`prototype.save()`、`prototype.updateAttributes()`
			- `brefore save`在`validators`之前被触发
			- 针对单个Model所有属性的save操作，context提供的属性有：`Model`、`instance`
			- 多个Model部分属性的更新操作，context提供的属性有：`Model`、`where`、`data`、`currentInstance`
			- 当context有instance属性的时候，before save hook提供了`context.isNewInstance`，当操作为`create()`时它的值为`true`，为`update()`则为`false`，如果是`updateOrCreate()`、`prototype.save()`、`prototypeUpdateAttributes()`和`updateAll()`时为`undefined`
			- 删除不需要的属性，针对有instance的属性的context，调用`unsetAttribute(name)`，而对提供`where、data`属性对的context，直接使用`delete`操作符即可，这样可以完全避免将伪造的或不需要的数据写入数据库
		- `after save`：基本同`before save`，不同的地方主要有：
			- 针对单个Model所有属性的save操作，context提供的instance在after save进行修改之后只对REST接口调用的结果有影响，但是不会写入数据库
			- 针对多个Model的更新操作中，context提供的`where`有可能会失效，如`MyModel.updateAll({ color: 'yellow' }, { color: 'red' }, cb);`，当after save被触发时，`where`也就是`{ color: 'yello'}`已经匹配不到数据了
			- 只有部分`connector`支持`context.isNewInstance`
		- `before delete`：
		- `after delete`：
		- `loaded`：在`connector`获取数据之后，通过这些数据创建Model之前被触发，可用于解密数据，context只提供`data`属性
		- `persist`：
		- `afterInitialize`：
3. Environment-Specific Configuration
	- 只能针对server目录下的config进行指定环境
	- 环境变量由`NODE_ENV`指定和获取，通过`app.process.ENV.NODE_ENV`访问
	- 指定环境变量配置主要有`Application wide`、`local`、`{ENV}`三种，如`config.json`、`config.local.json`、`config.{ENV}.json`，{ENV}通常为`development`、`production`、`staging`
	- 通过`local`或`{ENV}`重写`Application wide`配置只能重写`string`和`numbers`，内嵌object和数组类型暂时不支持
	- 加载优先级为：`{ENV}`>`local`>`Application wide`
4. 为内建的Model创建数据库表
	- 在`server/model-config.json`文件中修改对应`Model`的`dataSource`属性，如`mongoDs`
	- 需要创建js脚本，然后在命令行执行
	- 获取dataSrouce：`var ds = server.dataSources.mongoDs;`
	- 指定需要创建数据库表的Model：`var lbTables = ['User', 'AccessToken', 'ACL', 'RoleMapping', 'Role'];`
	- 迁移：`ds.automigrate(laTables, function(err) {...});`
	- 在`automigrate`回调中断开数据库连接：`ds.disconnect();`