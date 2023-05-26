# Scala学习笔记
@[toc]
### 一.scala相关
 1.`_.charAt(0)`表示首字母

 2.符号使用：

	循环<-
	=>拉姆达表达式
	->map

3.`：`后面跟数据类型

4.`list.insert`（指定位置，多个数据）指定添加

5.不可变命名语法

	val set = mutable.set(,,,)
	val list2 = ListBuffer(12, 53, 75)
	val arr2 = ArrayBuffer(23, 57, 92)

6.rdd数组中有数组，遍历方式为

	glomRDD.collect().foreach(data=>println(data.mkString(",")))

7.rdd不会回行，使用`collect().mkString(",")`

8.一般使用`不可变`的RDD，做更改转换为新的一个RDD


### 二.最简化原则：
	val mapRDD: RDD[Int] = rdd.map((num:Int)=>{num*2})
    val mapRDD: RDD[Int] = rdd.map((num:Int)=>num*2)
    val mapRDD: RDD[Int] = rdd.map(num=>num*2)
    val mapRDD: RDD[Int] = rdd.map(num=>num*2)
    val mapRDD: RDD[Int] = rdd.map(_*2)

### 三.创建用伴生对象：
	val listname = List(数据)
	val listname = ListBuffer(数据)
	val listname = Array(数据)   
	val listname = ArrayBuffer(数据)   

### 四.通用语法
1.按照集合类型生成字符串

	println(set)、println(list)

2.输出元素`自定义分隔符：`

	println(list.mkString("-"))

3.判断`包含`

	1.list.contains(23)
	2.set.contains(23)

### 五.修改、增加元素
1.list(3)=2

2.array和list通用增加`+：`，`：+`

3.list可用`+=`添加数据，如果在前面加就是666 +=：list

4.list1`++=`list2可以在list1后面追加list2

5.`++=:`是反过来追加

6.list加list用`：：：`或者`++`，但是要赋值给list3 val list3 = list1 ++ list2

### 六.添加数组
1.array和list增加array和list都用`++`就可以

2.`array.insertAll(插入位置, newArr)`插入数组

3.`array.prependAll(newArr)`在前面加数组

4.`array.prepend`前面加数据

5.`array.append`后面加数据

### 七.删除方式
1.`list.remove`(删除位置)

2.list `-=` 删除元素

### 八.打印方式
1.打印用`array.foreach(println)`

2.或者`println(array.mkString(“，”))`

3.或者`for(i <- list.indices) println(list(i))`

4.也可以`for(elem <- list) println(elem)`

5.迭代器 `for (elem <- list.iterator) println(elem)`

### 九.双重循环
1.双重循环用一个for加if
2.for循环用` i <- 1 to 10`用`；`隔开，如果写成 `j = 10 - i `表明`j`是一个`引入变量`
3.也可以写成

		for{
		i <- 1 to 10
		j= 10 - i 
		}



### 十一.Set
#### (1)`不可变`Set：
1.`val set01=Set(数据,数据)`,可以用来数据去重
2.添加元素:`set02=set01.+(20)`,set01是不可变类型的，需要用set02来接受,也可以写成`set1 + 20`
 3.`set是无序的`
 4.合并set用:`set2 ++set3 `
5.删除某个元素用:`set02 - 元素`

#### (2)`可变`Set：
1.定义方式：

	val set = mutable.set(,,,)
2.不能直接用＋来添加数据，要用新的set2来获取，可以调用`+=`添加元素或者用`set.add（）`
3.删除元素用:`-=`或者`set.remove`,set.remove底层调用的`-=`
4.合并set用`++`、`++=`


### 十二.Map
1.Map类型

	(val map1:Map(String,Int,自定类型)) = Map()
 2.可以直接打印查看:
 
	val map1=Map("a"->13, "b"->25,"Hello"->3)
	println(map1)
3.如果使用遍历：

	map1.foreach(println)
会得到一下形式

	(a,13)
	(b,25)
	(hello,3)
4.`取所有key，value`

	for(key <- map1.keys){
	println(s"$key--->${map.get(key)}")
	}
5.访问`某一个key的value`

	printlv(map1.get("a").get)
	//最好用map1.getOrElse("c",0),如果c没有值就返回0不会报异常
	//简化：println(map1("a"))

6.可变map，map都是`无序`的

	val map1 =mutable.Map("a" ->13)
7.添加1:

	map1.put("c",5)
	map1.put("d",9)
 8.添加2:
 	
 	map +=（"e"，7）
9.删除:
	
	map.remove("c"指定key就可以),或者map1 -=“d”
10.修改:
	
	map1.updete（“c”，5）如果不存在就是插入，如果存在就是修改
 11.合并用:`++`赋值给第三个map，原来的map不变，追加合并用`++=`并且做`覆盖`

### 十三.Queue
#### 1.`可变`队列
	val queue = new mutable.Queue[String] ()
1.1入队

	queue.enqueue("a","b")
	println(queue)
1.2出队

	queue.dequeue
#### 2.`不可变`队列
 	val queue2=Queue("a","b")
 需要用一个新队列去接受
 	
	val queue3 =queue2.enqueue("d")


### 十四.元组
 1.元组是将`多个无关`的数据封装为一个`整体`，最大`22`个元素，可以理解为一个`容器`， 里面存着各种`相同或不同类型`的数据

 2.声名元组`(相同或不同的数据类型，，，，)，`
 3.`创建`元组:
 
	 val tuple =（"hello",100,'a',true）,println(tuple)
4.`访问`数据：

	println（tuple._1）
	println（tuple._2）
	println（tuple._3）
	println（tuple._4）
	println（tuple.productElement(0)）
 5.`遍历`元组：
 
	for(elem <- tuple.productIterator)
	println(elem)
 6.`可嵌套`：
 
	val mulTuple = ("hello",(23,"scala"))
	
访问`"scala"`

	println(mulTuple._2._2)

### 十五.特定衍生集合操作

（1）获取集合的头`(头、尾的称法是针对list)`

 	println(list1.head)

（2）获取集合的尾`（不是头的就是尾）`

 	println(list1.tail)

（3）集合`最后一个`数据

 	println(list1.last)

4）集合`初始`数据`（不包含最后一个）

 	println(list1.init)

（5）反转

 	println(list1.reverse)

（6）取前（后）n 个元素

	println(list1.take(3))
 	println(list1.takeRight(3))

（7）去掉前（后）n 个元素

 	println(list1.drop(3))
 	println(list1.dropRight(3))

（8）并集

	 println(list1.union(list2))

（9）交集

 	println(list1.intersect(list2))

（10）差集

	 println(list1.diff(list2))

（11）拉链 
注:如果两个集合的元素`个数不相等`，那么会`将同等数量`的数据进行拉链，`多余的数据省略不用`。

 	println(list1.zip(list2))

（12）滑窗

 	list1.sliding(2, 5).foreach(println)

### 十六.常用计算函数
 1.求和 `list.sum`
 
    var sum = 0
    for (elem <- list){
      sum += elem
    }
    println(sum)

    println(list.sum)

 2.求乘积 `list.product`
 
    println(list.product)

3.最大值 `list.max list.maxBy`

    println(list.max)
    println(list2.maxBy( (tuple: (String, Int)) => tuple._2 ))
    println(list2.maxBy( _._2 ))

4.最小值 `list.min list.minBy`

    println(list.min)
    println(list2.minBy(_._2))

 5.排序
 5.1 `sorted`
 
    val sortedList = list.sorted
    println(sortedList)
	//从大到小逆序排序
    println(list.sorted.reverse)
    // 传入隐式参数
    println(list.sorted(Ordering[Int].reverse))
    println(list2.sorted)

 5.2 `sortBy`
 
    println(list2.sortBy(_._2))
    println(list2.sortBy(_._2)(Ordering[Int].reverse))

 5.3 `sortWith`
 
    println(list.sortWith( (a: Int, b: Int) => {a < b} ))
    println(list.sortWith( _ < _ ))
    println(list.sortWith( _ > _))



### 十七.常用方法
 1.过滤 `filter`
 
    // 选取偶数
    val evenList = list.filter( (elem: Int) => {elem % 2 == 0} )
    println(evenList)

    // 选取奇数
    println(list.filter( _ % 2 == 1 ))


2.映射 map

    // 把集合中每个数乘2
    println(list.map(_ * 2))
    println(list.map( x => x * x))
   

 3.扁平化 `flat`
 
    val nestedList: List[List[Int]] = List(List(1,2,3),List(4,5),List(6,7,8,9))

    val flatList = nestedList(0) ::: nestedList(1) ::: nestedList(2)
    println(flatList)

    val flatList2 = nestedList.flatten
    println(flatList2)

   
 4.扁平映射 `flatmap`(一般直接用flatmap就可以)
 
    // 将一组字符串进行分词，并保存成单词的列表
    val strings: List[String] = List("hello world", "hello scala", "hello java", "we study")
    val splitList: List[Array[String]] = strings.map( _.split(" ") )    // 分词
    val flattenList = splitList.flatten    // 打散扁平化

    println(flattenList)

    val flatmapList = strings.flatMap(_.split(" "))
    println(flatmapList)


5.分组 `groupBy`

    // 分成奇偶两组
    val groupMap: Map[Int, List[Int]] = list.groupBy( _ % 2)
    val groupMap2: Map[String, List[Int]] = list.groupBy( data => if (data % 2 == 0) "偶数" else "奇数")

    println(groupMap)
    println(groupMap2)

    // 给定一组词汇，按照单词的首字母进行分组
    val wordList = List("china", "america", "alice", "canada", "cary", "bob", "japan")
    println( wordList.groupBy( _.charAt(0) ) )