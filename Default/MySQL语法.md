## MySQL语法：

1. **基本样式**：括号里面表示可以选择的，`select (a as b),(sum(d)),(distinct c),(max(e))... from 表名 (where +条件....) (group by ...) (order by...)(limit a,b)`

2. **修改数据**`update 表名 set 字段名=值`

3. **插入数据**`insert into 表名(字段1，字段2...) value(值1，值2...)`

4. **常用逻辑判断**：`<>不等于,and,or,>,<,>=,<=`

5. **常用函数**`min(),sum(),max(),count(),distinct`

6. 查询多个字段之间用“，”隔开，如果要用a as b可以直接写成a b，表名也可以这样起别名，别名可以方便自己使用，也可以让查询出来的结果的字段名显示成这个别名，比如说你给usrid取别名为id，那么查询出来的字段名就叫id而不是userid。

7. 查询出来的**结果可以作为查询条件**，称为**子查询**，比如说一个语句查出一堆数字，就可以用a in(该语句)来判断a是否在查询结果中，查询出来的结果也可以作为一张表使用

8. 匹配字符串要用"" ''，比如查询某个字段等于男，就可以用select name from user where sex='男'，表示从user表中查询性别为男的姓名

9. 几种连接方式`join ...on..,union,left join,right join`

10. `select a as "b",c d,e `表示查询a称为b，c称为d，e，as可以省略," "可以省略，也可以写成' '

11. `IFNULL(num1,num2)`表示如果num1=null，就返回num2，可以用IFNULL(a,0)，来把a为Null的地方替换为0

12. `distinct num` 去重num

13. `is Null,is not Null`用于判断是否为Null值，Null表示该字段为空，0表示该字段值=0,两者意义不一样

14. `if(a,b,a)`可以用来颠倒a和b的值，如果a那就执行b，否则执行a，也可以通过"buy"字段来判断扣钱还是加钱

15. `upper`函数将所有字母字符转换为大写字母，对应LOWER

16. `concat（a,b）`连接a，b两个字符串，形成一个单一的字符串

17. `LEFT(a,b)`返回从字符串a开始的指定字符数b。LEFT不填充字符串;如果指定的字符数大于字符串中的字符数，则LEFT返回该字符串。如果传递给任何一个参数一个NULL值，左返回NULL。

18. `like "%2022%" 包含2022的 like "2022%"`以2022开头的

19. `in(),not in()`判断是否在括号中

20. `order by a DESC，b   `写多个条件表示先按a降序排列，如果相同就再按b升序排列，order by默认升序，加DESC改成降序

 21. `case when then
            when then
    else  `

22.`limit a,b `把数据分页一页多少个，显示第几页

23.`count()`计数

24.`DATEDIFF(start，end) `计算日期之差

25.日期范围可以用`between '2019-06-28' and '2019-07-27，>,<`也可以用
26.时间戳也可以调用`max,min`函数
27.`limit 1`，只显示第一个
