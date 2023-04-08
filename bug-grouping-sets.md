已经被官方修了commit id: 1cf57a985d4ab330565d8d90a0cb6dba011d84e2

# 复现环境
```
create database test;

CREATE TABLE test.kv_test
(
    k varchar(64),
    v INT
)
UNIQUE KEY(k)
DISTRIBUTED BY HASH(k) BUCKETS 1
PROPERTIES("replication_num" = "1");

insert into test.kv_test values('a', 1);
insert into test.kv_test values('a', 2);
insert into test.kv_test values('a', 3);
insert into test.kv_test values('b', 1);
insert into test.kv_test values('c', 1);
```

```
select flag, sum(v) from(select if(k = 'a',  'A', 'B') as flag, v from test.kv_test) t1 group by cube(flag);
```


![](doris-bug-cube-desc-graph.png)

select flag, sum(v) from (select if(k = 'abc', 'a', 'b') as flag, sum(v) as v from test.kv_test group by flag) t1 group by cube(flag);
![](doris-bug-cube-explain.png)

select flag, sum(v) from (select if(k = 'abc', 'a', 'b') as flag, v from test.kv_test) t1 group by cube(flag);
![](doris-bug-cube-diff.png)


# Commit
根据commit的改动点，理解实现逻辑
first: fc554230326b734ed2bf7aaa6697cd02ffa27985

# Class and call stack

- class StmtExecutor
    - method execute()
        - method analyze()
            - method analyzeAndGenerateQueryPlan(): 对有subquery的，做改写。改写后的，要重新再analyse一遍。
                - class Planner(NereidsPlanner, OriginalPlanner)
                    - plan()  
                        - method createPlanFragments()
                            - new SingleNodePlanner(plannerContext);
                                - method createSingleNodePlan()
    - handleQueryStmt()
    - SelectStmt(extend QueryStmt)
        - GroupByClause

RepeatNode(extend PlanNode): 对于 GroupingSets 的聚合，在执行计划中插入 RepeatNode


GroupingFunctionCallExpr: function，添加id等虚拟列
VirtualSlot：虚拟列和真实列的映射关系

# fault point assumption
1. repeatnode的具体repeat数据生成逻辑有误，没考虑if语句情况
