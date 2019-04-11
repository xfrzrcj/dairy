# **MYSQL**
## 1、事务隔离级别

事务隔离级别|中文|脏读|重复读|幻读
:--|:--:|:--:|:--:|:--:
read-uncommitted|读未提交|是|是|是
read-uncommitted|读已提交|否|是|是
repeatable-read|可重复读|否|否|是
serializable|串行化|否|否|否

**读未提交**：对于A,B两个事务，A在执行事务时，可以直接读到B未完成的事务。所以会发生脏读，例如A读到B正在插入的数据，但后来B回滚了数据，但A已经读取了B插入的脏数据。

**读已提交**：对A,B两个事务，只有当A完成提交了事务，B才能读到A插入更新的数据。虽然解决了脏读问题，但考虑一种情况，A的事务对同一数据X查询了两次，但在A执行事务期间，B已经完成了对数据X的修改，那么A在两次查询中就会发生前后读取数据X不一致的问题。

**可重复读**：确保A,B两个实例在并发读取相同数据时看到相同的数据行（单表操作时，相当于行锁）。解决了重复读的问题，但在A读取一个范围内的数据时，可能存在B实例插入了一条数据(相当于没有表锁)。导致A在事务内前后查询的数据个数不一致，发生幻读。

**串行化**：最高事务隔离级别，强制对事务排序，使之不会发生冲突。但性能最差。(单表操作时，相当于表锁)

---

## 2、锁