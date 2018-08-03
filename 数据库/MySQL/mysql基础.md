# 删除语句的异同：
相同点：delete truncate都能删除整个数据库表的数据。
不同点：  
delete:DML语言，可以rollback,可以有条件的删除  
truncate:DDL语言，无法rollback,默认所有的表内容都删除，速度比delete快。  
drop:用于删除表的所有(表的结构，属性以及索引也会被删除)