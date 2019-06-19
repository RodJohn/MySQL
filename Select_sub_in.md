# IN semi-join



# 

mysql> EXPLAIN EXTENDED
    -> SELECT *
    -> FROM test_order
    -> WHERE a NOT in
    -> (SELECT code FROM test_code )
    -> ;
+----+-------------+------------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+------------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | test_order | NULL       | ALL  | NULL          | NULL | NULL    | NULL |  100 |   100.00 | Using where |
|  2 | SUBQUERY    | test_code  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |  200 |   100.00 | NULL        |
+----+-------------+------------+------------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 2 warnings (0.00 sec)

mysql> show warnings;
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1681 | 'EXTENDED' is deprecated and will be removed in a future release.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Note    | 1003 | 
/* select#1 */ 
select `test`.`test_order`.`id` AS `id`,`test`.`test_order`.`a` AS `a`,`test`.`test_order`.`b` AS `b`,`test`.`test_order`.`c` AS `c` from `test`.`test_order` 
where (not(<in_optimizer>(`test`.`test_order`.`a`,`test`.`test_order`.`a` in ( <materialize> 
(/* select#2 */ select `test`.`test_code`.`code` from `test`.`test_code` where 1 ), 
<primary_index_lookup>
(`test`.`test_order`.`a` in <temporary table> on <auto_key> where ((`test`.`test_order`.`a` = `materialized-subquery`.`code`))))))) |
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)


mysql> EXPLAIN EXTENDED
    -> SELECT *
    -> FROM test_order o
    -> WHERE o.a in
    -> (SELECT c.code FROM test_code c WHERE c.id = o.id)
    -> ;
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref       | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------+------+----------+-------------+
|  1 | SIMPLE      | o     | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL      |  100 |   100.00 | NULL        |
|  1 | SIMPLE      | c     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | test.o.id |    1 |    10.00 | Using where |
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------+------+----------+-------------+
2 rows in set, 3 warnings (0.00 sec)

mysql> show warnings;
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                                                                                                                       |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1681 | 'EXTENDED' is deprecated and will be removed in a future release.                                                                                                                                                                                             |
| Note    | 1276 | Field or reference 'test.o.id' of SELECT #2 was resolved in SELECT #1                                                                                                                                                                                         |
| Note    | 1003 | 
/* select#1 */ select `test`.`o`.`id` AS `id`,`test`.`o`.`a` AS `a`,`test`.`o`.`b` AS `b`,`test`.`o`.`c` AS `c` from `test`.`test_code` `c` 
join `test`.`test_order` `o`where ((`test`.`c`.`code` = `test`.`o`.`a`) and (`test`.`c`.`id` = `test`.`o`.`id`)) |
+---------+------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set (0.00 sec)



