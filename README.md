heisenberg

High performance distributed  database for mysql, define shardings by velocity&groovy scripts, 
can be expanded nodes flexible...
==========

强大好用的mysql分库分表中间件，改编自cobar, 结合了cobar和TDDL的优势，让其分片策略变为分库表策略，节约了大量连接

其优点： 分库分表与应用脱离，分库表如同使用单库表一样
减少db 连接数压力 
热重启配置
可水平扩容
遵守Mysql原生协议
读写分离
无语言限制，mysqlclient,c,java等都可以使用
Heisenberg服务器通过管理命令可以查看，如连接数，线程池，结点等，并可以调整
采用velocity的分库分表脚本进行自定义分库表，相当的灵活

邮箱:brucest0078@gmail.com 或183320433@qq.com

1.0
正式发版

1.0.3.2
主要解决连接上带有脏数据问题
修复其它小bug

1.0.4
增加了后端连接再次回收利用过程
日志保存时间问题

1.0.5  2016.5.3
增加NIO后端能力，暂废弃

1.0.6  2016.10.31
1.增加连接复用问题
2.提高多并发连接利用效率
 
1.0.7  2017.1.10
1.修复killChannel的问题
2.修复读取数据递归问题

1.0.8.1  2018.8.2
```
1.使用expression完美支持分片替换问题
2.支持insert批量以及select in的分片替换，以及支持嵌套问题
mysql> explain insert into `test` (id,name) values (1,'brucexx'),('2','brucexx');
No connection. Trying to reconnect...
Connection id:    1
Current database: *** NONE ***

+------------+----------------------------------------------------------+
| DATA_NODE  | SQL                                                      |
+------------+----------------------------------------------------------+
| local_node | INSERT INTO `test_01` (id, name) VALUES (1, 'brucexx')   |
| local_node | INSERT INTO `test_02` (id, name) VALUES ('2', 'brucexx') |
+------------+----------------------------------------------------------+
2 rows in set (0.31 sec)
mysql> explain select * from test where id in ('1',2,3,4,5,6);
+------------+----------------------------------------------+
| DATA_NODE  | SQL                                          |
+------------+----------------------------------------------+
| local_node | SELECT * FROM `test_00` WHERE id IN (4)      |
| local_node | SELECT * FROM `test_01` WHERE id IN ('1', 5) |
| local_node | SELECT * FROM `test_02` WHERE id IN (2, 6)   |
| local_node | SELECT * FROM `test_03` WHERE id IN (3)      |
+------------+----------------------------------------------+
4 rows in set (0.08 sec)


mysql> explain select * from (select * from test union select * from test where id =1) as t where t.id in ('1',2,3,4,5,6);
+------------+--------------------------------------------------------------------------------------------------------------------+
| DATA_NODE  | SQL                                                                                                                |
+------------+--------------------------------------------------------------------------------------------------------------------+
| local_node | SELECT * FROM ((SELECT * FROM `test_00`) UNION (SELECT * FROM `test_00` WHERE id = 1)) AS T WHERE t.id IN (4)      |
| local_node | SELECT * FROM ((SELECT * FROM `test_01`) UNION (SELECT * FROM `test_01` WHERE id = 1)) AS T WHERE t.id IN ('1', 5) |
| local_node | SELECT * FROM ((SELECT * FROM `test_02`) UNION (SELECT * FROM `test_02` WHERE id = 1)) AS T WHERE t.id IN (2, 6)   |
| local_node | SELECT * FROM ((SELECT * FROM `test_03`) UNION (SELECT * FROM `test_03` WHERE id = 1)) AS T WHERE t.id IN (3)      |
+------------+--------------------------------------------------------------------------------------------------------------------+
4 rows in set (0.02 sec)

 mysql> explain select t.name from (select * from test ) as t left join test on t.id=test.id where t.id in ('1',2,3,4,5,6);
+------------+----------------------------------------------------------------------------------------------------------------+
| DATA_NODE  | SQL                                                                                                            |
+------------+----------------------------------------------------------------------------------------------------------------+
| local_node | SELECT t.name FROM (SELECT * FROM `test_00`) AS T LEFT JOIN `test_00` ON t.id = test.id WHERE t.id IN (4)      |
| local_node | SELECT t.name FROM (SELECT * FROM `test_01`) AS T LEFT JOIN `test_01` ON t.id = test.id WHERE t.id IN ('1', 5) |
| local_node | SELECT t.name FROM (SELECT * FROM `test_02`) AS T LEFT JOIN `test_02` ON t.id = test.id WHERE t.id IN (2, 6)   |
| local_node | SELECT t.name FROM (SELECT * FROM `test_03`) AS T LEFT JOIN `test_03` ON t.id = test.id WHERE t.id IN (3)      |
+------------+----------------------------------------------------------------------------------------------------------------+
4 rows in set (0.01 sec)
 
mysql> explain select t.name from (select * from test ) as t  left join test  on t.id = test.id where t.id =1 ;
+------------+--------------------------------------------------------------------------------------------------------+
| DATA_NODE  | SQL                                                                                                    |
+------------+--------------------------------------------------------------------------------------------------------+
| local_node | SELECT t.name FROM (SELECT * FROM `test_01`) AS T LEFT JOIN `test_01` ON t.id = test.id WHERE t.id = 1 |
+------------+--------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
````
