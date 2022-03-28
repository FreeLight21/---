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

   

