# 谢延  201810414224   软工2班

# 实验一：SQL语句的执行计划分析与优化指导

## 实验目的
    分析SQL执行计划，执行SQL语句的优化指导。理解分析SQL语句的执行计划的重要作用。

## 实验内容
- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。
- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。

### 查询语句

- 查询一

```sql
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY d.department_name;
```
#### 运行结果
![运行结果](https://img-blog.csdnimg.cn/20210316140117231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMjY2NA==,size_16,color_FFFFFF,t_70)

 #### 执行计划
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031620444537.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMjY2NA==,size_16,color_FFFFFF,t_70)

 #### 分析
 该查询语句通过员工表employees和部门表departments来查询部门的总人数和平均工资，并按照部门名'IT'和'Sales'进行分组查询。总人数直接使用了count(员工表id)得到员工人数，平均工资使用avg(员工表salary)算出。使用了多表联查从部门中找出了目标部门然后再通过其department_id在员工表中找出该部门所有的员工。
 可以通过创建多个索引改进此语句的执行计划，也可以考虑改进物理方案设计的访问指导或者创建推荐的索引。原理是创建推荐的索引可以显著地改进此语句的执行计划。但由于使用典型的SQL工作量运行“访问指导”可能比单个语句更加可取，所以通过这种方法可以获得全面的索引建议，包括计算索引维护的开销、附加的空间消耗和提升查询效率。
 
- 查询二

```sql
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY d.department_name
HAVING d.department_name in ('IT','Sales');
```

#### 运行结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316140258196.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMjY2NA==,size_16,color_FFFFFF,t_70)

#### 执行计划
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316203959150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMjY2NA==,size_16,color_FFFFFF,t_70)


#### 分析
 该查询语句同样是通过员工表employees和部门表departments来查询部门的总人数和平均工资，并按照部门名'IT'和'Sales'进行分组查询。判断部门ID和员工ID是否对应，由having确认部门名字是IT和sales来查询部门总人数和平 均工资。
 由于使用了WHERE和HAVING进行了两次过滤，结果更加精准，所以该查询语句比第一条查询语句要好一点，目前没有优化建议。

### 优化代码

```sql
set autotrace on

SELECT d.department_name,count(e.job_id) as "部门总人数",avg(e.salary) as "平均工资"
FROM  hr.departments d,hr.employees e
WHERE e.department_id=d.department_id and d.department_id in 
(SELECT department_id from hr.departments WHERE department_name in ('IT','Sales')) 
group by d.department_name;
```
#### 执行计划
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316204357827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMjY2NA==,size_16,color_FFFFFF,t_70)

#### 运行结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316192150470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzcyMjY2NA==,size_16,color_FFFFFF,t_70)


分析：该查询语句在语句一的基础上进行了优化，将查询的条件部门名'IT'和'Sales'换成对应部门的id进行查询，使用部门名查询对应的部门id作为子查询而得到的结果作为外层查询条件，这样来查询结果的准确性更高，查询效率也更高。