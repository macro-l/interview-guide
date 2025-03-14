1. mysql数据排序
    *分为索引排序和文件排序
        *索引排序：order by字段命中索引，并且排列顺序一致
        *文件排序：排序的数据量小-利用内存，排序的数据量大-利用磁盘
    *(延伸)双路排序和单路排序
        * 双路排序
            * 慢
            * 数据量超过max_length_for_sort_data生效
            * 先获取id和order by字段进行排序，完成后回表查询select获得结果
        * 单路排序
            * 快
            * 数据量小过max_length_for_sort_data生效
            * 读取所有列，根据order by 字段排序，无需回表直接返回结果

2. change Buffer作用
    * InnoDB 引擎的一个机制
    * 暂存对二级索引的插入和更新操作变更
    * 作用
        * 提高写入性能
        * 批量处理

3. mysql sql 语句的执行过程
    1. 连接器检验权限
    2. 分析器分析预计
    3. 优化器根据索引和表连接顺序，选择一个最佳执行计划
    4. 执行器调用引擎层查询数据，返回结果集给客户端

4. mysql 引擎类型与区别
    * InnoDB
        * 支持事务、行级锁和外健
        * 提供高并发性能，适合高负载应用
        * 数据以聚集索引的方式存储，提高检索效率
        
    * MyISAM
        * 不支持事务和外健，使用表级锁
        * 舍河读取多，更新少的场景，适合数据仓库
        * 具有较高的读性能和较快的表级锁定
    
    * Memory
        * 数据存储在内存里
        * 适用于临时存储
    
    * NDB
        * 支持高可用和数据分布，适合大规模分布式应用
        * 支持行级锁和自动分区
    
    * Archive(存档)
        * 用于存储大量历史数据，支持高校插入和压缩
        * 不支持索引，适合日志数据存储。

5. mysql的索引
    * 数据结构角度
        * B+树索引
            * 默认索引类型
            * 适用范围查询(eg:BETWEEN)和精确查询(eg:=)
        * 哈希索引
            * 基于哈希表
            * 适用等值查询(eg:=)
            * 不支持范围查询
            * 不存储数据的顺序,常用于Memory引擎
        * 倒排索引
        * R-树索引
    * InnoDB B+ 树索引角度
        * 聚簇索引98
        * 非聚簇索引
    * 索引性质的角度
        * 主键索引
            ```
        * 普通索引
        * 联合索引
        * 唯一索引
        * 全文索引
        * 空间索引

6. 索引的创建
    * 主键索引
        ```
        CREATE TABLE users (
            id INT NOT NULL AUTO_INCREMENT,
            username VARCHAR(50) NOT NULL,
            password VARCHAR(50),
            PRIMARY KEY (id)
        )
        ```
    * 普通索引
        ```
        CREATE INDEX idx_username ON users(username);
        ```
    * 联合索引
        ```
        CREATE INDEX idx_username_email ON users(username, email);
        ```
    * 唯一索引
        ```
        CREATE UNIQUE INDEX idx_username ON users(username);
        ```
    * 全文索引
        ```
        CREATE FULLTEXT INDEX idx_content ON articles(content);
        ```
    * 空间索引
        ```
        CREATE SPATIAL INDEX idx_localtion ON places(location);
        ```
    * 哈希索引
        ```
        CREATE INDEX idx_username_hash ON user(username) USING HASH;
        ```

7. 聚簇索引和非聚簇索引的区别
    * 聚簇索引
        * 索引叶子结点存储的是数据行， 可以直接访问完整数据。
        * 每个表只能有一个聚簇索引，通常是主键索引，适合范围查询和排序
    * 非聚簇索引
        * 索引叶子节点存储的是数据行的主键和对应的索引列，需通过主键才能访问完整的数据行。
        * 一个可以有多个非聚簇索引（称之为非主键索引、辅助索引、二级索引），适用于快速查找特定列的数据

8. mysql的回表是什么？
    * 在使用二级索引（非聚簇索引）作为条件进行查询时，由于二级索引中只存储了索引字段的值和对应的主键值，无法得到其它数据
    * 如果要查数据行中的其它数据，需要根据主键去聚簇索引查找实际的数据行，这个过程称为回表

9. mysql 索引的最左前缀匹配原则是什么？
    * 