# mysql_exfmt
'mysql_exfmt' is a dba tool for tuning sql, purposed to collect related information, and format them.

## 1. Prepare

### Version
  it was running for python2 on mysql_v56, now transferred to python3 on mysql_v57;
  <br>and extended feature and fixed bugs etc..

### Grant
  e.g `grant all on *.* to testuser@'localhost' identified by 'testpwd'`

### Parameter
  ~~mysql_v57 may need to 'set global show_compatibility_56=on'~~

  // had tried to compatible this -should be not req anymore..

## 2. Usage
  `python3 ./mysql_tuning.py -p [config_ini] { -s [sql] | -f [sql_file] }`

### Config
```ini
      [database]
      server_ip   = 127.0.0.1
      server_port = 3306
      db_user     = testuser
      db_pwd      = testpwd
      db_name     = test
      [option]
      sys_parm    = ON	// ON or OFF to get sys parm
      sql_plan    = ON	// ON or OFF to get sql plan
      obj_stat    = ON	// ON or OFF to get obj stat
      ses_status  = ON	// ON or OFF to get status diff (sql would really run if on)
      sql_profile = ON	// ON or OFF to get profile info (sql would really run if on)
```

## 3. Output Example
  // just some example, perhaps sth changed later..

#### BASIC INFORMATION

    ===== BASIC INFORMATION =====
    +-----------+-------------+-----------+---------+------------+
    | server_ip | server_port | user_name | db_name | db_version |
    +-----------+-------------+-----------+---------+------------+
    | localhost |     3501    |  testuser |   test  |   5.7.12   |
    +-----------+-------------+-----------+---------+------------+

#### SYSTEM PARAMETER

    ===== SYSTEM PARAMETER =====
    +-------------------------+-----------------+
    | parameter_name          |           value |
    +-------------------------+-----------------+
    | binlog_cache_size       |           1.0 M |
    | bulk_insert_buffer_size |          64.0 M |
    | join_buffer_size        |           8.0 M |
    | key_buffer_size         |         256.0 M |
    | key_cache_block_size    |           1.0 K |
    | max_binlog_cache_size   | 17179869183.0 G |
    | max_binlog_size         |           1.0 G |
    | max_join_size           | 17179869183.0 G |
    | query_cache_size        |             0 B |
    | query_prealloc_size     |           8.0 K |
    | range_alloc_block_size  |           4.0 K |
    | read_buffer_size        |           2.0 M |
    | read_rnd_buffer_size    |           8.0 M |
    | sort_buffer_size        |           2.0 M |
    | thread_cache_size       |            20 B |
    | tmp_table_size          |           1.0 G |
    +-------------------------+-----------------+

  // just chose some paramters may related to sql performance.

#### OPTIMIZER SWITCH

    ===== OPTIMIZER SWITCH =====
    +-------------------------------------+-------+
    | switch_name                         | value |
    +-------------------------------------+-------+
    | index_merge                         |    on |
    | index_merge_union                   |    on |
    | index_merge_sort_union              |    on |
    | index_merge_intersection            |    on |
    | engine_condition_pushdown           |    on |
    | index_condition_pushdown            |    on |
    | mrr                                 |    on |
    | mrr_cost_based                      |    on |
    | block_nested_loop                   |    on |
    | batched_key_access                  |   off |
    | materialization                     |    on |
    | semijoin                            |    on |
    | loosescan                           |    on |
    | firstmatch                          |    on |
    | duplicateweedout                    |    on |
    | subquery_materialization_cost_based |    on |
    | use_index_extensions                |    on |
    | condition_fanout_filter             |    on |
    | derived_merge                       |    on |
    +-------------------------------------+-------+

#### ORIGINAL SQL TEXT

    ===== ORIGINAL SQL TEXT =====
    SELECT d.dname,
           e.empno
    FROM big_dept d,
         big_emp e
    WHERE d.deptno=e.deptno LIMIT 10

#### SQL PLAN

    ===== SQL PLAN =====
    +----+-------------+-------+------------+-------+---------------+----------------+---------+---------------+------+----------+-------------+
    | id | select_type | table | partitions | type  | possible_keys | key            | key_len | ref           | rows | filtered | Extra       |
    +----+-------------+-------+------------+-------+---------------+----------------+---------+---------------+------+----------+-------------+
    |  1 | SIMPLE      | d     | None       | index | PRIMARY       | idx_dept_dname | 17      | None          | 1000 |    100.0 | Using index |
    |  1 | SIMPLE      | e     | None       | ref   | fk_deptno     | fk_deptno      | 5       | test.d.deptno |  996 |    100.0 | Using index |
    +----+-------------+-------+------------+-------+---------------+----------------+---------+---------------+------+----------+-------------+

#### OPTIMIZER REWRITE SQL

    ===== OPTIMIZER REWRITE SQL =====
    SELECT `test`.`d`.`dname` AS `dname`,
           `test`.`e`.`empno` AS `empno`
    FROM `test`.`big_dept` `d`
    JOIN `test`.`big_emp` `e`
    WHERE (`test`.`e`.`deptno` = `test`.`d`.`deptno`) LIMIT 10

#### OBJECT STATISTICS

    ===== OBJECT STATISTICS =====
    +------------+--------+---------+------------+---------+----------+---------+----------+
    | table_name | engine | format  | table_rows | avg_row | total_mb | data_mb | index_mb |
    +------------+--------+---------+------------+---------+----------+---------+----------+
    | big_dept   | InnoDB | Dynamic |       1000 |      81 |     0.08 |    0.08 |     0.00 |
    +------------+--------+---------+------------+---------+----------+---------+----------+
    +----------------+------------+--------------+-------------+-----------+-------------+----------+------------+
    | index_name     | non_unique | seq_in_index | column_name | collation | cardinality | nullable | index_type |
    +----------------+------------+--------------+-------------+-----------+-------------+----------+------------+
    | idx_dept_dname |          1 |            1 | dname       |     A     |        1000 |      YES | BTREE      |
    | PRIMARY        |          0 |            1 | deptno      |     A     |        1000 |          | BTREE      |
    +----------------+------------+--------------+-------------+-----------+-------------+----------+------------+
    +------------+--------+---------+------------+---------+----------+---------+----------+---------------------+---------------------+
    | table_name | engine | format  | table_rows | avg_row | total_mb | data_mb | index_mb |     create_time     |    last_analyzed    |
    +------------+--------+---------+------------+---------+----------+---------+----------+---------------------+---------------------+
    | big_emp    | InnoDB | Dynamic |     996293 |      62 |   110.17 |   59.58 |    50.59 | 2016-11-22 01:26:35 | 2017-01-13 01:18:55 |
    +------------+--------+---------+------------+---------+----------+---------+----------+---------------------+---------------------+
    +------------+------------+--------------+-------------+-----------+-------------+----------+------------+
    | index_name | non_unique | seq_in_index | column_name | collation | cardinality | nullable | index_type |
    +------------+------------+--------------+-------------+-----------+-------------+----------+------------+
    | fk_deptno  |          1 |            1 | deptno      |     A     |        1000 |      YES | BTREE      |
    | idx_sal    |          1 |            1 | sal         |     A     |       10145 |      YES | BTREE      |
    | PRIMARY    |          0 |            1 | empno       |     A     |      996293 |          | BTREE      |
    +------------+------------+--------------+-------------+-----------+-------------+----------+------------+
    +------------+---------------------+--------------+------------+-------------+-----------------------------------+
    | index_name |    last_analyzed    | stat_name    | stat_value | sample_size | stat_description                  |
    +------------+---------------------+--------------+------------+-------------+-----------------------------------+
    | PRIMARY    | 2017-01-13 01:18:55 | n_diff_pfx01 |     996293 |          20 | empno                             |
    | PRIMARY    | 2017-01-13 01:18:55 | n_leaf_pages |       3781 |        None | Number of leaf pages in the index |
    | PRIMARY    | 2017-01-13 01:18:55 | size         |       3813 |        None | Number of pages in the index      |
    | fk_deptno  | 2017-01-13 01:18:55 | n_diff_pfx01 |       1000 |          20 | deptno                            |
    | fk_deptno  | 2017-01-13 01:18:55 | n_diff_pfx02 |    1050373 |          20 | deptno,empno                      |
    | fk_deptno  | 2017-01-13 01:18:55 | n_leaf_pages |       1311 |        None | Number of leaf pages in the index |
    | fk_deptno  | 2017-01-13 01:18:55 | size         |       1507 |        None | Number of pages in the index      |
    | idx_sal    | 2017-01-13 01:18:55 | n_diff_pfx01 |      10001 |          20 | sal                               |
    | idx_sal    | 2017-01-13 01:18:55 | n_diff_pfx02 |    1047187 |          20 | sal,empno                         |
    | idx_sal    | 2017-01-13 01:18:55 | n_leaf_pages |       1373 |        None | Number of leaf pages in the index |
    | idx_sal    | 2017-01-13 01:18:55 | size         |       1731 |        None | Number of pages in the index      |
    +------------+---------------------+--------------+------------+-------------+-----------------------------------+

#### SESSION STATUS

    ===== SESSION STATUS (DIFFERENT) =====
    +----------------------------------+-----------+---------------+---------------+
    | status_name                      |    before |         after |          diff |
    +----------------------------------+-----------+---------------+---------------+
    | Bytes_received                   |       515 |           812 |         297.0 |
    | Bytes_sent                       |       661 |         12002 |       11341.0 |
    | Com_select                       |         2 |             4 |           2.0 |
    | Com_show_warnings                |         2 |             3 |           1.0 |
    | Created_tmp_tables               |         2 |             3 |           1.0 |
    | Handler_commit                   |         0 |             1 |           1.0 |
    | Handler_external_lock            |         0 |             4 |           4.0 |
    | Handler_read_first               |         0 |             1 |           1.0 |
    | Handler_read_key                 |         0 |             2 |           2.0 |
    | Handler_read_next                |         0 |             9 |           9.0 |
    | Handler_read_rnd                 |         0 |           380 |         380.0 |
    | Handler_read_rnd_next            |         6 |           387 |         381.0 |
    | Handler_write                    |       193 |           573 |         380.0 |
    | Innodb_buffer_pool_bytes_data    |  43204608 |      43270144 |       65536.0 |
    | Innodb_buffer_pool_pages_data    |      2637 |          2641 |           4.0 |
    | Innodb_buffer_pool_pages_free    |     30115 |         30111 |          -4.0 |
    | Innodb_buffer_pool_read_requests |  26779348 |      26779357 |           9.0 |
    | Innodb_buffer_pool_reads         |      2565 |          2569 |           4.0 |
    | Innodb_data_read                 |  42095104 |      42160640 |       65536.0 |
    | Innodb_data_reads                |      2595 |          2600 |           5.0 |
    | Innodb_num_open_files            |        23 |            24 |           1.0 |
    | Innodb_pages_read                |      2564 |          2568 |           4.0 |
    | Innodb_rows_read                 |  54637053 |      54637064 |          11.0 |
    | Last_query_cost                  | 10.499000 | 201556.132981 | 201545.633981 |
    | Last_query_partial_plans         |         1 |             3 |           2.0 |
    | Open_tables                      |        54 |            56 |           2.0 |
    | Opened_tables                    |         0 |             2 |           2.0 |
    | Queries                          |   2734384 |       2734387 |           3.0 |
    | Questions                        |         6 |             9 |           3.0 |
    | Select_scan                      |         2 |             4 |           2.0 |
    | Sort_rows                        |         0 |           380 |         380.0 |
    | Sort_scan                        |         0 |             1 |           1.0 |
    | Table_open_cache_misses          |         0 |             2 |           2.0 |
    +----------------------------------+-----------+---------------+---------------+

  // show the difference before/after sql ran,

  // but note perhaps a bit flaw (e.g Com_select) since it using 'select' to get data.

#### SQL PROFILING

    ===== SQL PROFILING(DETAIL) =====
    +----------------+----------+----------+----------+-------+--------+-------+-------+--------+--------+-------+
    | state          | duration | cpu_user |  cpu_sys | bk_in | bk_out | msg_s | msg_r | p_f_ma | p_f_mi | swaps |
    +----------------+----------+----------+----------+-------+--------+-------+-------+--------+--------+-------+
    | starting       | 0.000079 | 0.000000 | 0.000000 |     0 |      0 |     0 |     0 |      0 |      0 |     0 |
    | query end      | 0.000006 | 0.000000 | 0.000000 |     0 |      0 |     0 |     0 |      0 |      0 |     0 |
    | closing tables | 0.000004 | 0.000000 | 0.000000 |     0 |      0 |     0 |     0 |      0 |      0 |     0 |
    | freeing items  | 0.000010 | 0.000000 | 0.000000 |     0 |      0 |     0 |     0 |      0 |      0 |     0 |
    | cleaning up    | 0.000073 | 0.000000 | 0.000000 |     0 |      0 |     0 |     0 |      0 |      0 |     0 |
    +----------------+----------+----------+----------+-------+--------+-------+-------+--------+--------+-------+
    bk_in:   block_ops_in
    bk_out:  block_ops_out
    msg_s:   message sent
    msg_r:   message received
    p_f_ma:  page_faults_major
    p_f_mi:  page_faults_minor

    ===== SQL PROFILING(SUMMARY) =====
    +----------------+----------+-------+-------+--------------+
    | state          |  total_r | pct_r | calls |       r/call |
    +----------------+----------+-------+-------+--------------+
    | starting       | 0.000079 | 45.93 |     1 | 0.0000790000 |
    | cleaning up    | 0.000073 | 42.44 |     1 | 0.0000730000 |
    | freeing items  | 0.000010 |  5.81 |     1 | 0.0000100000 |
    | query end      | 0.000006 |  3.49 |     1 | 0.0000060000 |
    | closing tables | 0.000004 |  2.33 |     1 | 0.0000040000 |
    +----------------+----------+-------+-------+--------------+

#### EXECUTE TIME

    ===== EXECUTE TIME =====
    0 day 0 hour 0 minute 0 second 162 microsecond

  // would run 2 times if status & profile both on.

# Author
  - mysql_exfmt.py mig/upd by Shane.Qian#foxmail.com
  - based on mysql_tuning.py v2.0 originally by hanfeng
