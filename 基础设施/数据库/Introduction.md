# Introduction

> 这一部分内容过于抽象，不理解没关系，知道有这么一个概念存在就行。以后会补充一些实例来辅助理解的

[TOC]

## DBMS

A **database-management system** (**DBMS**) is a collection of interrelated data and a set of programs to access those data. The collection of data, usually referred to as the **database**.

在早期，程序所要访问大批数据的物理层与其逻辑层紧密耦合，这难以维护与扩展。于是就提出了数据库来解决这一问题。

the primary goal of DBMS includes :

-  store and retrieve database information that is both **convenient **and **efficient**
- provide users with an abstract view of data



由DBMS管理的数据具有以下特点：

- highly valuable
- relatively large
- accessed by multiple users and applications, often at the same time（权限 + 并发）



Broadly speaking, there are two modes in which databases are used：

- **online transaction processing**：each user retrieving relatively small amounts of data, and performing small updates
- **data analytics**： the processing of data to draw conclusions, and infer rules or decision procedures



用操作系统提供的文件系统来管理数据（例如，使用CSV文件）， 它有以下问题：

- 复杂性高，难以维护与扩展，对程序员编程并不友好，使得程序员心智负担过大。
- 访问数据低效
- 并发问题、安全性问题、分布式
- 应用的数据层与某一种编程语言绑定，不易扩展。
- 模型简单，表达力不丰富，外键约束等难以实现





## Data Model & Data Abstraction

**data model**: a collection of conceptual tools for **describing data, data relationships, data semantics, and consistency constraints**. 通常对现实世界中的各种数据进行抽象和概念化



- Relational Model
- NoSQL：例如，Key/Value、Graph、Documnet/XML/JSON、Wide-Column
- Array/Matrix/Vector：用于MachineLearning
- Network、Hierarchical等是已经废用了





> 数据抽象建立在合适的数据模型之上，它需要根据数据模型中定义的结构和关系来组织和抽象数据

在确定数据模型后，我们还要考虑数据抽象，它将复杂的数据模型实现细节或者数据模型本身的细节隐藏起来，向用户提供简单和合适的接口（更具体来说，是提供一个更高级的概念模型）。



数据抽象有三个层次：

- **Physical level**. The lowest level of abstraction describes **how** the data are actually stored.
- **Logical level**. The next-higher level of abstraction describes **what** data are stored in the database, and **what** relationships exist among those data. 一般是 **physical data independent**的
- **View level**. The highest level of abstraction describes only part of the entire database. the views also provide a security mechanism to prevent users from accessing certain parts of the database.

![image-20230812111056187](C:\Users\AtsukoRuo\Desktop\note\数据库\assets\image-20230812111056187.png)



在数据抽象的基础上，我们来设计整个数据库。data schema是数据模型在具体系统中的具体实现方式

> instance与schema的关系就如同编程语言中type与variable的关系

The collection of information stored in the database at a particular moment is called an **instance** of the database. The overall design of the database is called the database **schema**. 它也对应地分为三个层次：

- **physical schema** describes the database design at the physical level
- **logical schema** describes the database design at the logical level
- **subschemas**, that describe different views of the database.

 

data model侧重概念，data schema侧重实现，data abstraction侧重封装。

## Database Languages



> 数据库语言可以看作是数据抽象的一部分，它正是这样一种接口和抽象工具。

对于Relation Database Languages可以划分为以下两种子语言：

- **data-defifinition language** (**DDL**)： specify the database schema and other things，例如 storage structure、access methods、constraints、View、indices、Security等. 

  The processing of DDL statements generates some output. The output of the DDL is placed in the **data dictionary**, which contains **metadata**。例如表结构、视图、索引、存储位置等
  
  

- **data-manipulation language** (**DML**）： enables users to access data. The types of access are:

  - Retrieval
  - Insertion
  - Deletion
  - Modifification

  

  There are basically two types of data-manipulation language:

  - **Procedural** **DML**： require a user to specify *what* data are needed and *how* to get those data
  - **Declarative** **DML**： require a user to specify *what* data are needed *without* specifying how to get those data.

  > DML that involves information retrieval is called a **query language**. it is common practice to use the terms query language and data-manipulation language synonymously. 因为查询语句是最常用的DML



The Java Database Connectivity (JDBC) standard定义了一些接口，这些接口允许JAVA向Database发送DML语句

## Constraint

In general,**a constraint can be an arbitrary predicate pertaining** to the database. However, arbitrary predicates **may be costly to test**. Thus, database systems **implement only those integrity constraints（完整性约束）** that can be tested with minimal overhead:

- **Domain Constraints**（值域约束）. A domain of possible values must be associated with every attribute
- **Referential Integrity**（参照完整性）：a value that appears in one relation for a given set of attributes also appears in a certain set of attributes in another relation
- **Authorization**： **read authorization**、**insert authorization**、**update authorization**、**delete authorization**
- Other： Key Constraints、Not Null、Default、Check...



## Database Engine



![image-20230812114437180](C:\Users\AtsukoRuo\Desktop\note\数据库\assets\image-20230812114437180.png)



