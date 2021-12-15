---
title: MySQL两款引擎的不同
date: 2021-09-15
tag: 问题总结
categories: 数据库
mathjax: false
top: false
---

## MyISAM 和 InnoDB 的区别

> `MyISAM` 和 `InnoDB` 是在 `MySQL` 中常用的两个引擎，这篇文章翻译自一篇来自 StackOverflow 的回答文章。

### 1. 题目原文

MyISAM is designed with the idea that your database is queried far more than its updated and as a result it performs very fast read operations. If your read to write(insert|update) ratio is less than 15% its better to use MyISAM.

InnoDB uses row level locking, has commit, rollback, and crash-recovery capabilities to protect user data. It supports transaction and fault tolerance

above differences is correct between MyISAM and InnobDB? **please guide if any other limitations are there for MYISAM and InnobDB. when should i use MyiSAM or when Innodb?** Thank you!

> MyISAM 的设计理念是基于你的数据库的查询远比更新频繁，因此它拥有很快的读取操作性能。如果你的读/写（插入、更新）比率小于15%，用 MyISAM 更好。
>
> InnoDB 使用了行等级锁，拥有提交、回滚和快速恢复能力以用来保护用户数据。它支持了事务和差错容忍。
>
> 上述的区别正确吗？如果还有区别的话，请给予指导。什么时候我该使用这两个引擎？谢谢🙏

### 2. 回答

**MyISAM:**

The [MyISAM](http://dev.mysql.com/doc/refman/5.1/en/myisam-storage-engine.html) storage engine in MySQL.

- Simpler to design and create, thus better for beginners. No worries about the foreign relationships between tables.
- Faster than InnoDB on the whole as a result of the simpler structure thus much less costs of server resources. -- Mostly no longer true.
- Full-text indexing. -- InnoDB has it now
- Especially good for read-intensive (select) tables. -- Mostly no longer true.
- Disk footprint is 2x-3x less than InnoDB's. -- As of Version 5.7, this is perhaps the only real advantage of MyISAM.

> MyISAM：
>
> 内置于 MySQL 的存储引擎有如下特点。
>
> * 更容易设计和创建，因此对初学者更友好。不必担心表间关联查询的问题
> * 得益于更简单的结构，它各方面比 InnoDB 要快，因此需要更少的服务器资源。
> * 全文本搜索功能。（InnoDB 在 MySQL 5.6 以后也支持了这一功能）
> * 尤其善于主要执行读（select）的表
> * 硬盘占用要比 InnoDB 少 2～3 倍。在 5.7 以后，这可能是 MyISAM 唯一的真正优势所在

**InnoDB:**

The [InnoDB](http://dev.mysql.com/doc/refman/5.1/en/innodb-storage-engine.html) storage engine in MySQL.

- Support for transactions (giving you support for the [ACID](http://en.wikipedia.org/wiki/ACID) property).
- Row-level locking. Having a more fine grained locking-mechanism gives you higher concurrency compared to, for instance, [MyISAM](http://dev.mysql.com/doc/refman/5.1/en/myisam-storage-engine.html).
- Foreign key constraints. Allowing you to let the database ensure the integrity of the state of the database, and the relationships between tables.
- InnoDB is more resistant to table corruption than MyISAM.
- Support for large buffer pool for both data and indexes. MyISAM key buffer is only for indexes.
- MyISAM is stagnant; all future enhancements will be in InnoDB. This was made abundantly clear with the roll out of Version 8.0.

> InnoDB：
>
> 内置于 MySQL 的 InnoDB 引擎有以下特点。
>
> * 对事务的支持（很好地支持了 ACID ）ACID 指的是原子性、一致性、隔离性、持久性，这是关系数据库可以正确处理事物的关键
> * 行级锁。拥有一个更好的上锁机制，拥有相对更高的并发处理
> * 外键限制。允许用户决定数据库确保数据库状态的完整性和表间的关系
> * InnoDB 相比于 MyISAM 对表出错的抵抗性更强
> * 支持对数据和索引的大缓冲区。MyISAM 的主缓冲区仅可用于索引。
> * MyISAM 是停滞的，未来所有的增强都会基于 InnoDB。这在 8.0 版本之后会变得相当明显。

从这里我们可以得知，InnoDB 是 MySQL 在 5.6 版本及其之后的发展主流，由于其良好的关系数据库特性以及日益增强的查询特性，在较新版本的 MySQL 中引擎用 InnoDB 就好。之后可以查询一下历史沿革。这类问题在 StackOverflow 上有很多，可以看看。
