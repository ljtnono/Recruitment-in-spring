# MySQL学习

## 存储引擎

插件式存储引擎是MySQL数据库最重要的特性之一，用户可以根据应用的需要选择如何存储和索引数据、是否使用事务等。MySQL默认支持多种存储引擎，以适用于不同领域的数据库应用需要，用户可以通过选择使用不同的存储引擎提高应用的效率，提供灵活的存储，用户甚至可以按照自己的需要定制和使用自己的存储引擎，以实现最大程度的可定制性。

MySQL 5.0支持的存储引擎包括MyISAM、InnoDB、BDB、MEMORY、MERGE、EXAMPLE、NDB Cluster、ARCHIVE、CSV、BLACKHOLE、FEDERATED等，其中 InnoDB和BDB提供事务安全表，其他存储引擎都是非事务安全表。其中除了InnoDB和BDB支持事务，其他存储引擎均不支持事务，各种引擎的特性对比表如下。

