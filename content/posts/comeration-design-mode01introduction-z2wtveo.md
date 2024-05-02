---
title: 纵览设计模式-01-导论
slug: comeration-design-mode01introduction-z2wtveo
url: /post/comeration-design-mode01introduction-z2wtveo.html
date: '2024-05-02 17:03:36+08:00'
lastmod: '2024-05-02 17:41:18+08:00'
toc: true
isCJKLanguage: true
---

# 纵览设计模式-01-导论

## 概述

> 在1994年,由Erich Gamma、Richard Helm、Ralph Johnson和John Vlissides 四人合著出版了一本名为Design Patterns-Elements of Reusalole Object-OrientedSoftware(中文译名:设计模式-可复用的面向对象软件元素)的书,该书首次提到了软件开发中设计模式的概念。

*对接口编程而不是对实现编程。* 

*优先使用对象组合而不是继承。* 

​![image](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/20240502170830.png)​![image](https://xducodert-blog.oss-cn-chengdu.aliyuncs.com/test/20240502171207.png)​​

## 设计的7大原则

### 开闭原则(OpenClosedPrinciple,OCP)

软件实体应当对扩展开放,对修改关闭(Software entitiess should be open for extension but closed for modification)

合成复用原则、里氏替换原则相辅相成,都是开闭原则的具体实现规范

扩展新类而不是修改旧类

### 里氏替换原则(Liskov Substitution Principle,LSP)

继承必须确保超类所拥有的性质在子类中仍然成立(Inheritaance should ensure that any property proved about supertype objects also holds for subtype objects)

继承父类而不去改变父类

### 依赖倒置原则(Dependence Inversion Principle, DIP)

高层模块不应该依赖低层模块,两者都应该依赖其抽象;抽象不应该依赖细节,细节应该依赖抽象(High level modules shouldnot depend upon low levelmodules.Both should depend upon abstractions. Abstractions should not depend upon details. Details should

### 单一职责原则(SingleResponsibility Principle,SRP)

一个类应该有且仅有一个引起它变化的原因,否则类应该被拆分(There should never be more than one reason for a class to change)  
每个类只负责自己的事情,而不是变成万能

### 接口隔离原则(Interface Segregation Principle,ISP)

一个类对另一个类的依赖应该建立在最小的接口上(The dependency of one class to another one should depend on the smallest possible interface. .  
各个类建立自己的专用接口,而不是建立万能接口

### 迪米特法则(Law of Demeter,LoD)

最少知识原则(LeastKnowledgePrinciple,LKP)只与你的直接朋友交谈,不跟"陌生人"说话(Talk onlytoyoour immediate friends and not to strangers)  
无需直接交互的两个类,如果需要交互,使用中间者

### 合成复用原则(CompositeReusePrinciple,CRP)

又叫组合/聚合复用原则(Composition/Aggregate Reuse PPrinciple, CARP)软件复用时,要尽量先使用组合或者聚合等关联关系来实现,其次才考虑使用继承关系来实现  
优先组合,其次继承

‍
