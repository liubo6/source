---
title: 高性能JDBC连接池HikariCP
date: 2016-09-26 09:32:20
categories: JDBC
tags: 连接池
---
性能方面 hikariCP>druid>tomcat-jdbc>dbcp>c3p0 。hikariCP的高性能得益于最大限度的避免锁竞争
<!--more-->

# 连接池对比
功能|dbcp|durid|c3p0|tomcat-jdbc|HikariCP
---|---|---
是否支持PSCache| 是|   是|   是|   否|   否
监控 | jmx| jmx/log/http|    jmx,log| jmx| jmx
扩展性| 弱|   好|   弱|   弱|   弱
sql拦截及解析|    无|   支持|  无|   无|   无
代码|  简单|  中等|  复杂|  简单|  简单
特点 | 依赖于common-pool|  阿里开源，功能全面   |历史久远，代码逻辑复杂，且不易维护 |      优化力度大，功能简单，起源于boneCP
连接池管理|LinkedBlockingDeque | 数组|     | FairBlockingQueue |  threadlocal+CopyOnWriteArrayList

psCache的key为prepare执行的sql和catalog等，value对应的为prepareStatement对象。开启缓存主要是减少了解析sql的开销
开启psCache缓存,性能大概有20%幅度的提升。可考虑开启pscache

![](http://ww1.sinaimg.cn/mw690/69045600gw1f7xib7okgcj20lp09agob.jpg)

# 使用HikariCP

## maven依赖
```
Java 8 maven artifact
<dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>2.5.0</version>
</dependency>

Java 7 maven artifact
<dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP-java7</artifactId>
        <version>2.4.8</version>
</dependency>
```

### property file based:
```
HikariConfig config = new HikariConfig("some/path/hikari.properties");
HikariDataSource ds = new HikariDataSource(config);
Example property file:
dataSourceClassName=com.mysql.jdbc.jdbc2.optional.MysqlDataSource
dataSource.user=test
dataSource.password=test
dataSource.databaseName=mydb
dataSource.serverName=localhost
```

### HikariConfig
```
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/simpsons");
config.setUsername("bart");
config.setPassword("51mp50n");
config.addDataSourceProperty("cachePrepStmts", "true");
config.addDataSourceProperty("prepStmtCacheSize", "250");
config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");

HikariDataSource ds = new HikariDataSource(config);
```



## Popular DataSource Class Names
Database  |  Driver|  DataSource class
---|---|---
Apache Derby |   Derby  | org.apache.derby.jdbc.ClientDataSource
Firebird   | Jaybird| org.firebirdsql.pool.FBSimpleDataSource
IBM DB2| DB2 com.ibm.db2.jcc.DB2SimpleDataSource
H2 | H2 | org.h2.jdbcx.JdbcDataSource
HSQLDB | HSQLDB  |org.hsqldb.jdbc.JDBCDataSource
MariaDB & MySQL| MariaDB |org.mariadb.jdbc.MySQLDataSource
MySQL  | Connector/J| com.mysql.jdbc.jdbc2.optional.MysqlDataSource
MS SQL Server|   Microsoft |  com.microsoft.sqlserver.jdbc.SQLServerDataSource
Oracle | Oracle | oracle.jdbc.pool.OracleDataSource
PostgreSQL|  pgjdbc-ng |  com.impossibl.postgres.jdbc.PGDataSource
PostgreSQL | PostgreSQL | org.postgresql.ds.PGSimpleDataSource
SyBase  |jConnect  |  com.sybase.jdbcx.SybDataSource






