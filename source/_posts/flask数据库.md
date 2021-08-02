---
title: flask数据库
tags:
  - flask
  - SQL
comments: true
typora-root-url: flask数据库
date: 2021-08-02 23:42:21
categories: Flask
index_img:
banner_img:
---

关系型数据库把数据存储在**表**中，表为应用中不同的实体建模。例如，订单管理应用的数据库中可能有`customers`、`products`和`orders`等表。

表中的列数是固定的，行数是可变的。列定义表所表示的实体的数据属性。例如，customers表中可能有name、address、phone等列。表中的行定义部分或所有列对应的真实数据。

表中有个特殊的列，称为**主键**，其值为表中各行的唯一标识符。表中还可以有称为**外键**的列，引用同一个表或不同表中某一行的主键。行之间的这种联系称为关系，这正是关系型数据库模型的基础。

```mermaid
classDiagram
	class roles
	roles: id
	roles: name
	
	class users
	users: id
	users: username
	users: password
	users: role_id
	
	roles <|-- users
	
```









[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


