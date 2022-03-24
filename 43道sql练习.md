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

   

