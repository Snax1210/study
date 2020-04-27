###  事务的四种隔离级别与七种传播行为
<!-- TOC -->

- [事务的四种隔离级别与七种传播行为](#事务的四种隔离级别与七种传播行为)
- [事务A：启动一个事务start transaction;select \* from tx;+------+------+](#事务a启动一个事务start-transactionselect-\-from-tx------------)
- [事务B：也启动一个事务(那么两个事务交叉了)](#事务b也启动一个事务那么两个事务交叉了)
- [事务A：那么这时候事务A能看到这个更新了的数据吗?select \* from tx;+------+------+](#事务a那么这时候事务a能看到这个更新了的数据吗select-\-from-tx------------)
- [事务B：事务B回滚,仍然未提交rollback;select \* from tx;+------+------+](#事务b事务b回滚仍然未提交rollbackselect-\-from-tx------------)
- [事务A：在事务A里面看到的也是B没有提交的数据select \* from tx;+------+------+](#事务a在事务a里面看到的也是b没有提交的数据select-\-from-tx------------)
        - [**第2级别：Read Committed(读取提交内容)**](#第2级别read-committed读取提交内容)
- [首先修改隔离级别](#首先修改隔离级别)
- [事务A：启动一个事务start transaction;select \* from tx;+------+------+](#事务a启动一个事务start-transactionselect-\-from-tx-------------1)
- [事务B：也启动一个事务(那么两个事务交叉了)](#事务b也启动一个事务那么两个事务交叉了-1)
- [事务A：这个时候我们在事务A中能看到数据的变化吗?select \* from tx;](#事务a这个时候我们在事务a中能看到数据的变化吗select-\-from-tx)
- [事务B：如果提交了事务B呢?      |](#事务b如果提交了事务b呢- - - )
- [事务A:                         |](#事务a- - - - - - - - - - - - -)
        - [第3级别：Repeatable Read(可重读)](#第3级别repeatable-read可重读)
- [首先，更改隔离级别](#首先更改隔离级别)
- [事务A：启动一个事务start transaction;](#事务a启动一个事务start-transaction)
- [事务B：开启一个新事务(那么这两个事务交叉了)](#事务b开启一个新事务那么这两个事务交叉了)
- [事务A：这时候即使事务B已经提交了,但A能不能看到数据变化？select \* from tx;](#事务a这时候即使事务b已经提交了但a能不能看到数据变化select-\-from-tx)
- [事务A：只有当事务A也提交了，它才能够看到数据变化](#事务a只有当事务a也提交了它才能够看到数据变化)
        - [第4级别：Serializable(可串行化)](#第4级别serializable可串行化)
- [首先修改隔离界别](#首先修改隔离界别)
- [事务A：开启一个新事务start transaction;](#事务a开启一个新事务start-transaction)
- [事务B：在A没有commit之前，这个交叉事务是不能更改数据的start transaction;](#事务b在a没有commit之前这个交叉事务是不能更改数据的start-transaction)

<!-- /TOC -->

**一、** **事务的基本要素（ACID）**

**1** **、原子性（Atomicity）：事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。**

**2** **、一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。**

**3** **、隔离性（Isolation）：同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。**

**4** **、持久性（Durability）：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。**

**二、事务的并发问题**

**1** **、脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据**

**2** **、不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。** 

**3** **、幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。**

**小结：不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表**

**三、事务隔离级别**

  

### Read Uncommitted(读取未提交内容)

(1) 所有事务都可以看到其他未提交事务的执行结果  
(2)本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少  
(3)该级别引发的问题是——脏读(Dirty Read)：读取到了未提交的

修改事务隔离级别

set tx\_isolation='READ-UNCOMMITTED';

select @@tx\_isolation;

+------------------+

| @@tx\_isolation   |

+------------------+

| READ-UNCOMMITTED |

+------------------+

#事务A：启动一个事务start transaction;select \* from tx;+------+------+

| id   | num  |

+------+------+

|    1 |    1 |

|    2 |    2 |

|    3 |    3 |

+------+------+

#事务B：也启动一个事务(那么两个事务交叉了)  
       在事务B中执行更新语句，且不提交start transaction;

update tx set num=10 where id=1;

select \* from tx;+------+------+

| id   | num  |

+------+------+

|    1 |   10 |

|    2 |    2 |

|    3 |    3 |

+------+------+

#事务A：那么这时候事务A能看到这个更新了的数据吗?select \* from tx;+------+------+

| id   | num  |

**|    1 |   10 |** 

|    2 |    2 |

|    3 |    3 |

+------+------+

**可以看到！说明我们读到了事务B还没有提交的数据**

#事务B：事务B回滚,仍然未提交rollback;select \* from tx;+------+------+

| id   | num  |

+------+------+

|    1 |    1 |

|    2 |    2 |

|    3 |    3 |

+------+------+

#事务A：在事务A里面看到的也是B没有提交的数据select \* from tx;+------+------+

| id   | num  |

|    1 |    1 |      

|    2 |    2 |

|    3 |    3 |

+------+------+

**脏读意味着我在这个事务中(A中)，事务B虽然没有提交，但它任何一条数据变化，都可以看到！**

### **第2级别：Read Committed(读取提交内容)**

(1)这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）  
(2)它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变  
(3)这种隔离级别出现的问题是——不可重复读(Nonrepeatable Read)：不可重复读意味着我们在同一个事务中执行完全相同的select语句时可能看到不一样的结果。  
     导致这种情况的原因可能有：(1)有一个交叉的事务有新的commit，导致了数据的改变;(2)一个数据库被多个实例操作时,同一事务的其他实例在该实例处理其间可能会有新的commit

#首先修改隔离级别  
set tx\_isolation='read-committed';

select @@tx\_isolation;

+----------------+

| @@tx\_isolation |

+----------------+

| READ-COMMITTED |

+----------------+

#事务A：启动一个事务start transaction;select \* from tx;+------+------+

| id   | num  |

+------+------+

|    1 |    1 |

|    2 |    2 |

|    3 |    3 |

+------+------+

#事务B：也启动一个事务(那么两个事务交叉了)  
       在这事务中更新数据，且未提交start transaction;

update tx set num=10 where id=1;

select \* from tx;

+------+------+

| id   | num  |

|    1 |   10 |

|    2 |    2 |

|    3 |    3 |

#事务A：这个时候我们在事务A中能看到数据的变化吗?select \* from tx; 

| id   | num  |                |

|    1 |    1 |                          **  |**

|    2 |    2 |                |

|    3 |    3 |                |

**相同的select语句，结果却不一样**

                               |

#事务B：如果提交了事务B呢?      |

commit;                        |

                               |

#事务A:                         |

select \* from tx; 

| id   | num  |

|    1 |   10 |**\--->****因为事务B已经提交了，所以在A中我们看到了数据变化**

|    2 |    2 |

|    3 |    3 |

### 第3级别：Repeatable Read(可重读)

(1)这是MySQL的默认事务隔离级别  
(2)它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行  
(3)此级别可能出现的问题——幻读(Phantom Read)：当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行  
(4)InnoDB和Falcon存储引擎通过多版本并发控制(MVCC，Multiversion Concurrency Control)机制解决了该问题

#首先，更改隔离级别  
set tx\_isolation='repeatable-read';

select @@tx\_isolation;

+-----------------+

| @@tx\_isolation  |

+-----------------+

| REPEATABLE-READ |

#事务A：启动一个事务start transaction;

select \* from tx;

| id   | num  |

|    1 |    1 |

|    2 |    2 |

|    3 |    3 |

#事务B：开启一个新事务(那么这两个事务交叉了)  
       在事务B中更新数据，并提交start transaction;

update tx set num=10 where id=1;

select \* from tx;

| id   | num  |

|    1 |   10 |

|    2 |    2 |

|    3 |    3 |

commit;

#事务A：这时候即使事务B已经提交了,但A能不能看到数据变化？select \* from tx;

| id   | num  |

|    1 |    1 | --->还是看不到的！(这个级别2不一样，也说明级别3解决了不可重复读问题)

|    2 |    2 |

|    3 |    3 |

#事务A：只有当事务A也提交了，它才能够看到数据变化

commit;

select \* from tx;

| id   | num  |

|    1 |   10 |

|    2 |    2 |

|    3 |    3 |

### 第4级别：Serializable(可串行化)

(1)这是最高的隔离级别  
(2)它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之,它是在每个读的数据行上加上共享锁。  
(3)在这个级别，可能导致大量的超时现象和锁竞争

#首先修改隔离界别  
set tx\_isolation='serializable';

select @@tx\_isolation;

| @@tx\_isolation |

| SERIALIZABLE   |

#事务A：开启一个新事务start transaction;

#事务B：在A没有commit之前，这个交叉事务是不能更改数据的start transaction;

insert tx values('4','4');

ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transactionupdate tx set num=10 where id=1;

ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

四、事务传播行为
========

事务传播行为（propagation behavior）指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行。   
例如：methodA事务方法调用methodB事务方法时，methodB是继续在调用者methodA的事务中运行呢，还是为自己开启一个新事务运行，这就是由methodB的事务传播行为决定的。

  
