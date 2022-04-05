1. 取得每个部门最高薪水的人员名称

   ```sql
   select 
   	e.ename, t.*
   from 
   	emp e
   join
   	(select deptno,max(sal) as maxsal from emp group by deptno) t
   on
   	t.deptno = e.deptno and t.maxsal = e.sal;
   ```

2. 哪些人的薪水在部门的平均薪水之上

   ```sql
   1.部门的平均薪水
   select avg(sal)as avgSal from emp group by deptno; 
   2.哪些人的薪水大于部门的平均薪水
   select 
      e.ename, e.sal
   from
   	emp e
   join
   	(select deptno,avg(sal) as avgsal from emp group by deptno) t
   on
   	e.deptno = t.deptno and e.sal > t.avgsal;
   ```

3. 取得部门中（所有人的）平均的薪水等级，如下：

   ```sql
   mysql> select deptno, avg(grade)
       -> from
       -> (select e.deptno, s.grade from emp e join salgrade s on e.sal between s.losal and s.hisal) t
       -> group by deptno;
   ```

4. 不准用组函数（Max），取得最高薪水（给出两种解决方案）

   * 第一种方案

     ```sql
     mysql> select sal from emp order by sal desc limit 1;
     ```

   * 第二种方案

     ```sql
     mysql> select sal from emp where sal not in (select distinct a.sal from emp a join emp b on a.sal < b.sal);//这个不是自己写的
     ```

5. 取得平均薪水最高的部门的部门编号（至少给出两种解决方案）

   * 第一种方案

     ```sql
     mysql> select t.deptno from
         -> (select avg(sal)as avgsal, deptno from emp group by deptno order by avgsal desc limit 1) t;
     ```

   * 第二种方案

     ```sql
     mysql> select deptno, avg(sal) as avgsal
         -> from emp
         -> group by deptno
         -> having
         -> avgsal = (select max(t.avgsal) from (select avg(sal) as avgsal from emp group by deptno ) t);
     ```
     
     ```sq
     mysql> select max(avgsal), t.deptno from (select avg(sal) as avgsal, deptno from emp group by deptno) t;
     ```
     
     上面这种虽然结果看着是一样的，但是错误的，t.deptno是随机查出来的，并不一定是最大平均工资的编号。

6. 取得平均薪水最高的部门的部门名称

   ```sql
   mysql> select d.dname
       -> from dept d
       -> join
       -> (select avg(sal) as avgsal, deptno from emp group by deptno order by avgsal desc limit 1) t
       -> on d.deptno = t.deptno;
   ```
   
   ```sql
   //标准答案
   mysql> select d.dname, avg(e.sal) as avgsal
       -> from emp e
       -> join dept d
       -> on e.deptno = d.deptno
       -> group by d.dname
       -> order by avgsal desc
       -> limit 1;
   ```
   
7. 求平均薪水的等级最低的部门的部门名称

   ```sql
   //多表连接
   mysql> select d.dname, s.grade
       -> from
       -> (select avg(sal) as avgsal, deptno from emp group by deptno) t
       -> join salgrade s
       -> on t.avgsal between s.losal and s.hisal
       -> join dept d
       -> on t.deptno = d.deptno
       -> order by grade
       -> limit 1;
   ```

8. 取得比普通员工(员工代码没有在mgr字段上出现的)的最高薪水还要高的领导人姓名）

   ```sql
   //怎么实现员工代码没有在mgr字段上
   select empno from emp where empno not in (select distinct mgr from emp where mgr is not null);
   //我写的，结果为空集，--》not in 关键字用的不行， 后面的结果集中不能出现null，否则返回0条记录。
   mysql> select empno from emp where empno not in (select mgr from emp);
   
   mysql> select ename, sal
       -> from emp
       -> where sal > (select max(sal) from emp where empno not in (select distinct mgr from emp where mgr is not null));
   ```

   >not in 后面的结果集中不能出现null，否则返回0条记录。

9. 取得薪水最高的前五名员工

   ```sql
   mysql> select ename, sal from emp order by sal desc limit 5;
   ```

10. 取得薪水最高的第六到第十名员工

    ```sql
    mysql> select ename, sal from emp order by sal desc limit 5, 5;
    //mysql的记录是从0开始的，第六就是第五条记录
    ```

11. 取得最后入职的5名员工

    ```sql
    mysql> select ename, hiredate from emp order by hiredate desc limit 5;
    ```

12. 取得每个薪水等级有多少员工

    ```sql
    mysql> select s.grade, count(s.grade) as count
        -> from emp e
        -> join salgrade s
        -> on e.sal between s.losal and s.hisal
        -> group by s.grade;
    ```

13. 有3 个表 S( 学生表 ），C （课程表 ），SC （学生选课表), S（SNO,SNAME）代表(学号，姓名)

    C(CNO,CNAME,CTEACHER) 代表(课号，课名，教师)

    SC(SNO,CNO,SCGRADE) 代表(学号，课号，成绩)。 

    * 1问题：找出没选过“黎明”老师的所有学生姓名。

      ```sql
      //1.在所有的数据库中统一使用单引号括起来，是标准，双引号可以在mysql中使用
      //2. window控制台插入的是gbk编码，不是utf-8
      ```

    * 2问题：列出 2 门以上（含 2 门）不及格学生姓名及平均成绩。

      ```sql
      
      ```

    * 3问题：即学过 1 号课程又学过 2 号课所有学生的姓名。

      ```sql
      
      ```

14. 列出所有员工及领导的姓名

    ```sql
    mysql> select e.ename, t.ename
        -> from emp e
        -> join emp t
        -> on e.mgr = t.empno;
    ```

15. 列出受雇日期早于其直接上级的所有员工的编号,姓名,部门名称

    ```sql
    1.受雇日期早于上级的受雇日期
    mysql> select e.empno, e.ename, d.dname
        -> from emp e
        -> join emp t
        -> on e.mgr=t.empno and e.hiredate < t.hiredate
        -> join dept d
        -> on e.deptno = d.deptno;
    //或者这样写：
    mysql> select e.empno, e.ename, d.dname
        -> from emp e
        -> join emp t
        -> on e.mgr=t.empno
        -> join dept d
        -> on e.deptno = d.deptno
        -> where e.hiredate < t.hiredate;
        
    ```

16. 列出部门名称和这些部门的员工信息,同时列出那些没有员工的部门.

    ```sql
    //要列出没有员工的部门
    mysql> select d.dname, e.*
        -> from emp e
        -> right join dept d
        -> on e.deptno = d.deptno;
    ```

17. 列出至少有5个员工的所有部门

    ```sql
    mysql> select deptno,  count(deptno) from emp group by deptno having count(deptno) >= 5;
    ```

18. 列出薪金比"SMITH"多的所有员工信息.

    ```sql
    mysql> select * from emp where sal > (select sal from emp where ename = 'SMITH');
    ```

19. 列出所有"CLERK"(办事员)的姓名及其部门名称,部门的人数.

    ```sql
    //clerk的姓名和部门名称
    mysql> select e.ename, d.dname,e.deptno
        -> from emp e
        -> join dept d
        -> on e.deptno = d.deptno
        -> where e.job='CLERK';
    +--------+------------+
    | ename  | dname      |
    +--------+------------+
    | MILLER | ACCOUNTING |
    | SMITH  | RESEARCH   |
    | ADAMS  | RESEARCH   |
    | JAMES  | SALES      |
    +--------+------------+
    mysql> select a.ename, a.dname, b.cc
        -> from (select e.ename, d.deptno,d.dname from emp e join dept d on e.deptno = d.deptno where e.job='CLERK') a
        -> join (select deptno, count(deptno)as cc from emp group by deptno) b
        -> on a.deptno = b.deptno;
    +--------+------------+----+
    | ename  | dname      | cc |
    +--------+------------+----+
    | MILLER | ACCOUNTING |  3 |
    | SMITH  | RESEARCH   |  5 |
    | ADAMS  | RESEARCH   |  5 |
    | JAMES  | SALES      |  6 |
    +--------+------------+----+    

20. 列出最低薪金大于1500的各种工作及从事此工作的全部雇员人数.

    ```sql
    mysql> 
    +-----------+--------------+
    | job       | count(empno) |
    +-----------+--------------+
    | ANALYST   |            2 |
    | MANAGER   |            3 |
    | PRESIDENT |            1 |
    | SALESMAN  |            1 |
    +-----------+--------------+
    //做错了 ?最低薪资是指工作的薪资， 一个工作可能有多个人对应，他们的薪资不多，找出最低薪资》1500
    
    mysql> select job ,count(*) from emp group by job having min(sal) > 1500;
    +-----------+----------+
    | job       | count(*) |
    +-----------+----------+
    | ANALYST   |        2 |
    | MANAGER   |        3 |
    | PRESIDENT |        1 |
    +-----------+----------+
    ```

21. 列出在部门"SALES"<销售部>工作的员工的姓名,假定不知道销售部的部门编号.

    ```sql
    mysql> select ename from emp where deptno = (select deptno from dept where dname = 'SALES');
    +--------+
    | ename  |
    +--------+
    | ALLEN  |
    | WARD   |
    | MARTIN |
    | BLAKE  |
    | TURNER |
    | JAMES  |
    +--------+
    ```

22. 列出薪金高于公司平均薪金的所有员工,所在部门,上级领导,雇员的工资等级.

    ```sql
    //1.算出公司平均薪资
     select avg(sal) from emp;
    
    mysql> select e.ename, d.dname, t.ename, s.grade
        -> from emp e
        -> join dept d
        -> on e.deptno = d.deptno
        -> join salgrade s
        -> on e.sal between s.losal and s.hisal
        -> left join emp t
        -> on e.mgr = t.empno
        -> where e.sal >(select avg(sal) from emp);
    +-------+------------+-------+-------+
    | ename | dname      | ename | grade |
    +-------+------------+-------+-------+
    | JONES | RESEARCH   | KING  |     4 |
    | BLAKE | SALES      | KING  |     4 |
    | CLARK | ACCOUNTING | KING  |     4 |
    | SCOTT | RESEARCH   | JONES |     4 |
    | KING  | ACCOUNTING | NULL  |     5 |
    | FORD  | RESEARCH   | JONES |     4 |
    +-------+------------+-------+-------+
    
    ```

23. 列出与"SCOTT"从事相同工作的所有员工及部门名称.

    ```sql
    //scott的工作岗位
    mysql> select e.ename, d.dname
        -> from emp e
        -> join dept d
        -> on e.deptno = d.deptno
        -> where e.job = (select job from emp where ename = 'SCOTT') and e.ename != 'SCOTT';
    +-------+----------+
    | ename | dname    |
    +-------+----------+
    | FORD  | RESEARCH |
    +-------+----------+
    ```

24. 列出薪金等于部门30中员工的薪金的其他员工的姓名和薪金.

    ```sql
    mysql> select distinct sal from emp where deptno = 30;
    +---------+
    | sal     |
    +---------+
    | 1600.00 |
    | 1250.00 |
    | 2850.00 |
    | 1500.00 |
    |  950.00 |
    +---------+
    //这个题目的题意表达不清楚
    ```

25. 列出薪金高于在部门30工作的所有员工的薪金的员工姓名和薪金.部门名称.

    ```sql
    mysql> select e.ename, e.sal, d.dname
        -> from emp e
        -> join dept d
        -> on e.deptno = d.deptno
        -> where e.sal >(select max(n.sal) from (select distinct sal from emp where deptno = 30) n );
    +-------+---------+------------+
    | ename | sal     | dname      |
    +-------+---------+------------+
    | KING  | 5000.00 | ACCOUNTING |
    | JONES | 2975.00 | RESEARCH   |
    | SCOTT | 3000.00 | RESEARCH   |
    | FORD  | 3000.00 | RESEARCH   |
    +-------+---------+------------+
    ```

26. 列出在每个部门工作的员工数量,平均工资和平均服务期限.

    ```sql
    //平均服务期限？--》入职年份-现在的年份
    mysql> select d.dname, count(e.deptno),avg(e.sal), ifnull(avg(timestampdiff(YEAR, hiredate, now())),0) as avgyear
        -> from emp e
        -> join dept d
        -> on e.deptno = d.deptno
        -> group by
        -> e.deptno;
    +------------+-----------------+-------------+---------+
    | dname      | count(e.deptno) | avg(e.sal)  | avgyear |
    +------------+-----------------+-------------+---------+
    | ACCOUNTING |               3 | 2916.666667 | 40.0000 |
    | RESEARCH   |               5 | 2175.000000 | 38.0000 |
    | SALES      |               6 | 1566.666667 | 40.3333 |
    +------------+-----------------+-------------+---------+
    ```

27. 列出所有员工的姓名、部门名称和工资

    ```sql
    mysql> select e.ename, e.sal, d.dname
        -> from emp e
        -> join dept d
        -> on e.deptno = d.deptno;
    +--------+---------+------------+
    | ename  | sal     | dname      |
    +--------+---------+------------+
    | CLARK  | 2450.00 | ACCOUNTING |
    | KING   | 5000.00 | ACCOUNTING |
    | MILLER | 1300.00 | ACCOUNTING |
    | SMITH  |  800.00 | RESEARCH   |
    | JONES  | 2975.00 | RESEARCH   |
    | SCOTT  | 3000.00 | RESEARCH   |
    | ADAMS  | 1100.00 | RESEARCH   |
    | FORD   | 3000.00 | RESEARCH   |
    | ALLEN  | 1600.00 | SALES      |
    | WARD   | 1250.00 | SALES      |
    | MARTIN | 1250.00 | SALES      |
    | BLAKE  | 2850.00 | SALES      |
    | TURNER | 1500.00 | SALES      |
    | JAMES  |  950.00 | SALES      |
    +--------+---------+------------+
    ```

28. 列出所有部门的详细信息和人数

    ```sql
    mysql> select d.*, ifnull(t.num,0)
        -> from dept d
        -> left join (select deptno, count(deptno) as num from emp group by deptno) t
        -> on d.deptno = t.deptno;
    +--------+------------+----------+-----------------+
    | DEPTNO | DNAME      | LOC      | ifnull(t.num,0) |
    +--------+------------+----------+-----------------+
    |     10 | ACCOUNTING | NEW YORK |               3 |
    |     20 | RESEARCH   | DALLAS   |               5 |
    |     30 | SALES      | CHICAGO  |               6 |
    |     40 | OPERATIONS | BOSTON   |               0 |
    +--------+------------+----------+-----------------+ 
    
    //做法二：
    select 
    	d.deptno,d.dname,d.loc,count(e.ename)
    from
    	emp e
    right join
    	dept d
    on
    	e.deptno = d.deptno
    group by
    	d.deptno,d.dname,d.loc;
    ```

29. 列出各种工作的最低工资及从事此工作的雇员姓名

    ```sql
    1.求出所有工作的最低工资
    mysql> select ename,job, min(sal) from emp group by job;
    +-------+-----------+----------+
    | ename | job       | min(sal) |
    +-------+-----------+----------+
    | SCOTT | ANALYST   |  3000.00 |
    | SMITH | CLERK     |   800.00 |
    | JONES | MANAGER   |  2450.00 |
    | KING  | PRESIDENT |  5000.00 |
    | ALLEN | SALESMAN  |  1250.00 |
    +-------+-----------+----------+
    2.从事此工作的雇员信息
    mysql> select e.*
        -> from emp e
        -> join (select job, min(sal) as msal from emp group by job) t
        -> on e.job = t.job and e.sal = t.msal;
    +-------+--------+-----------+------+------------+---------+---------+--------+
    | EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
    +-------+--------+-----------+------+------------+---------+---------+--------+
    |  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
    |  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
    |  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
    |  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
    |  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
    |  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
    |  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
    +-------+--------+-----------+------+------------+---------+---------+--------+
    ```
    
30. 列出各个部门的MANAGER(领导)的最低薪金

    ```sql
    mysql> select t.deptno, min(t.sal) as minsal
        -> from emp e
        -> join emp t
        -> on e.mgr = t.empno
        -> group by t.deptno;
    +--------+---------+
    | deptno | minsal  |
    +--------+---------+
    |     10 | 2450.00 |
    |     20 | 2975.00 |
    |     30 | 2850.00 |
    +--------+---------+
    ```

31. 列出所有员工的年工资,按年薪从低到高排序

    ```sql
    mysql> select ename, (sal*12) as income from emp order by income;
    +--------+----------+
    | ename  | income   |
    +--------+----------+
    | SMITH  |  9600.00 |
    | JAMES  | 11400.00 |
    | ADAMS  | 13200.00 |
    | WARD   | 15000.00 |
    | MARTIN | 15000.00 |
    | MILLER | 15600.00 |
    | TURNER | 18000.00 |
    | ALLEN  | 19200.00 |
    | CLARK  | 29400.00 |
    | BLAKE  | 34200.00 |
    | JONES  | 35700.00 |
    | FORD   | 36000.00 |
    | SCOTT  | 36000.00 |
    | KING   | 60000.00 |
    +--------+----------+
    ```

32. 求出员工领导的薪水超过3000的员工名称与领导名称

    ```sql
    mysql> select e.ename, t.ename
        -> from emp e
        -> join emp t
        -> on e.mgr = t.empno
        -> where t.sal > 3000;
    +-------+-------+
    | ename | ename |
    +-------+-------+
    | JONES | KING  |
    | BLAKE | KING  |
    | CLARK | KING  |
    +-------+-------+
    ```

33. 求出部门名称中,带'S'字符的部门员工的工资合计、部门人数.

    ```sql
    mysql> select d.dname, ifnull(sum(e.sal),0), count(e.deptno)
        -> from dept d
        -> left join emp e
        -> on e.deptno = d.deptno
        -> group by e.deptno
        -> having d.dname like '%S%';
    +------------+------------+-----------------+
    | dname      | sum(e.sal) | count(e.deptno) |
    +------------+------------+-----------------+
    | OPERATIONS |       0    |               0 |
    | RESEARCH   |   10875.00 |               5 |
    | SALES      |    9400.00 |               6 |
    +------------+------------+-----------------+
    ```

34. 给任职日期超过30年的员工加薪10%.

    ```sql
    update emp set sal = 1.1*sal where timestampDiff(YEA,hiredate,now())>30;
    ```
    
    
    
    

