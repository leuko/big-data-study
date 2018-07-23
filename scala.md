# Scala学习笔记

## 变量和常量的声明和使用

1. 变量声明后可以更改，但常量不允许
2. 后期在代码编写中，最常用的类型是 常量(val)
3. scala可以自动推断出变量或常量的类型,也可以显示的来指定，比如：val v6:Int=100
4. scala可以无缝衔接java的类库，此外，scala也有自己的类库方法
5. scala有一种隐式转换机制。比如声明一个String类型，初始时是java的类型。当调用的方法不是java类库方法时，`scala会隐式转换成scala的类型，比如：String(java)->StringOps(scala)`

```scala
object Demo01 {
  println("Welcome to the Scala worksheet")       //> Welcome to the Scala worksheet
  
  //--声明变量
  var v1=100                                      //> v1  : Int = 100
  
  //--声明常量
  val v2=200                                      //> v2  : Int = 200
  
  val v3="hello"                                  //> v3  : String = hello
  
  val v4=2.0                                      //> v4  : Double = 2.0
  
  val v5=2.0f                                     //> v5  : Float = 2.0
  
  val v6:Int=100                                  //> v6  : Int = 100
  
  //--针对字符串做repeat操作
  v3.*(2)                                         //> res0: String = hellohello
  v3.split("l")                                   //> res1: Array[String] = Array(he, "", o)
  
  //--去重
  v3.distinct                                     //> res2: String = helo
  //--去除头n个元素
  v3.drop(1)                                      //> res3: String = ello
  //--去除尾部n个元素
 	v3.dropRight(2)                           //> res4: String = hel
 	
 	//--提取头n个元素
 	v3.take(1)                                //> res5: String = h
 	
 	//--提取尾部n个元素
 	v3.takeRight(2)                           //> res6: String = lo
 	
 	//--foreach 遍历函数，一般用于打印
 	//--foreach不仅可以应用String，其他类型都适用
 	v3.foreach{x=>println(x)}                 //> h
                                                  //| e
                                                  //| l
                                                  //| l
                                                  //| o
  //--filter 根据指定的原则做过滤，其他类型都适用
  v3.filter { x =>x!='l' }                        //> res7: String = heo
  
  
  v3.contains("l")                                //> res8: Boolean = true
  v3.exists { x => x=='l' }                       //> res9: Boolean = true
  //--返回头1个元素。相当于take(1)
  v3.head                                         //> res10: Char = h
  
  //--返回尾部最后一个元素。
  v3.last                                         //> res11: Char = o
  
  v3.mkString(",")                                //> res12: String = h,e,l,l,o
  
  //--反转
  v3.reverse                                      //> res13: String = olleh
  
  val v7="100"                                    //> v7  : String = 100
  v7.toInt                                        //> res14: Int = 100
  v7.toDouble                                     //> res15: Double = 100.0
  v7.toFloat                                      //> res16: Float = 100.0
}
```





## 面向对象

1. scala的面向对象比java更彻底，都是对象和方法。没有基本数据类型, 此外，针对Int，也会通过隐式转换，变为scala的对象类型

> Int->RichInt
>
> Double—>RichDouble
>
> ……RichFloat
>
> ……

1. scala在做运算时，如果以方法形式调用，则以方法调用顺序来执行，比如：`val v4=2.+(3).*(5)=25`如果以操作符的形式来调用，则以操作符优先级的顺序调用。优先级顺序同java

```scala
object Demo02 {
  println("Welcome to the Scala worksheet")       //> Welcome to the Scala worksheet
  
  val v1=1                                        //> v1  : Int = 1
  
  v1.+(3)                                         //> res0: Int = 4
  
  //--生成指定范围的区间数据，常用于循环时使用
  v1.to(5)                                        //> res1: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5)
  //--生成区间并指定步长
  v1.to(5,2)                                      //> res2: scala.collection.immutable.Range.Inclusive = Range(1, 3, 5)
  
  //--生成区间，但不含尾
  v1.until(5)                                     //> res3: scala.collection.immutable.Range = Range(1, 2, 3, 4)
  v1.until(5,2)                                   //> res4: scala.collection.immutable.Range = Range(1, 3)
  
  //--map方法可以针对集合类型操作，改变集合中元素的类型
 	v1.to(5).map { x => x.toDouble }          //> res5: scala.collection.immutable.IndexedSeq[Double] = Vector(1.0, 2.0, 3.0, 
                                                  //| 4.0, 5.0)
 	
 	v1.toDouble                               //> res6: Double = 1.0
 	v1.toFloat                                //> res7: Float = 1.0
 	v1.toString                               //> res8: String = 1
 	
 	val v3:BigInt=2                           //> v3  : BigInt = 2
 	v3.pow(10)                                //> res9: scala.math.BigInt = 1024
 	
 	val v4=2.+(3).*(5)                        //> v4  : Int = 25
 	val v5=2+3*5                              //> v5  : Int = 17
 	//--前缀操作符，4种：+  -  ！   ~
 	//--需要前面加空格,为了避免歧义，可以通过unary_来使用
 	val v6= -2                                //> v6  : Int = -2
 	val v7= +2                                //> v7  : Int = 2
 	val v8= !false                            //> v8  : Boolean = true
 	val v9= ~0XFF                             //> v9  : Int = -256

	val v10=2.unary_-                         //> v10  : Int = -2
	val v11=2.unary_+                         //> v11  : Int = 2
	val v12=false.unary_!                     //> v12  : Boolean = true
	val v13=0XFF.unary_~                      //> v13  : Int = -256
	
	val v14="hello\t1711"                     //> v14  : String = hello	1711
	
	//--三引号的使用
	val v15="""hello\t1711"""                 //> v15  : String = hello\t1711
}
```



## 条件判断

1. scala的if else语句是有返回值的
2. scala的语法规则（通用）:会将方法体的最后一行代码当做返回值返回。
3. scala的语法规则（通用）:scala每行后面不需要加 ;
4. 如果一行中需要些多条语句，则需要用 ; 隔开
5. 如果scala的方法体里只有一行代码(通用)，则方法体{}可以省略
6. 如果scala调用的方法，只有一个参数，则.()可以省略
7. match case 匹配机制类似于java 的switch
8. match case语句是有返回值的

```scala
object Demo03 {
  println("Welcome to the Scala worksheet")       //> Welcome to the Scala worksheet
  val a=5                                         //> a  : Int = 5
  
  val result=if(a>5){
  	
  	"big"
  }else{
  
  	"small"
  }                                               //> result  : String = small
  
  //val r2=if(a>5)"big";else"small"                  //> r2  : String = small
  
  val a1=Array(1,2,3,4,5)                         //> a1  : Array[Int] = Array(1, 2, 3, 4, 5)
  
  var index=0                                     //> index  : Int = 0
  
  while(index<a1.length){
  	println(a1(index))
  	index+=1
  }                                               //> 1
                                                  //| 2
                                                  //| 3
                                                  //| 4
                                                  //| 5
   for(i<-a1){
   	println(i)                                //> 1
                                                  //| 2
                                                  //| 3
                                                  //| 4
                                                  //| 5
   }
   
   for(i<- 1.to(5)){
   	println(i)                                //> 1
                                                  //| 2
                                                  //| 3
                                                  //| 4
                                                  //| 5
   }
   for(i<- 1 to 5 by 2)println(i)                 //> 1
                                                  //| 3
                                                  //| 5
   //--scala的for循环里可以写条件判断语句。
   //--多个条件之间用 ;隔开
   for(i<- 1 to 5;if i>3;if i%2==0 )println(i)    //> 4
   
   //--要求打印 99乘法表
   //1*1=1
   //1*2=2   2*2=4
   //1*3=3   2*3=6  3*3=9
   //提示：①for的循环嵌套  ②print 和println函数
   for(a<- 1 to 9){
   		for(b<-1 to a){
   			print(b+"*"+a+"="+b*a+"\t")
   		}
   		println()
   }                                              //> 1*1=1	
                                                  //| 1*2=2	2*2=4	
                                                  //| 1*3=3	2*3=6	3*3=9	
                                                  //| 1*4=4	2*4=8	3*4=12	4*4=16	
                                                  //| 1*5=5	2*5=10	3*5=15	4*5=20	5*5=25	
                                                  //| 1*6=6	2*6=12	3*6=18	4*6=24	5*6=30	6*6=36	
                                                  //| 1*7=7	2*7=14	3*7=21	4*7=28	5*7=35	6*7=42	7*7=49	
                                                  //| 1*8=8	2*8=16	3*8=24	4*8=32	5*8=40	6*8=48	7*8=56	8*8=64	
                                                  //| 1*9=9	2*9=18	3*9=27	4*9=36	5*9=45	6*9=54	7*9=63	8*9=72	9*9=81	
                                                  //| 
   for(a<- 1 to 9;b<- 1 to a;val s=if(b==a)"\r\n";else"\t")print(b+"*"+a+"="+b*a+s)
                                                  //> 1*1=1
                                                  //| 1*2=2	2*2=4
                                                  //| 1*3=3	2*3=6	3*3=9
                                                  //| 1*4=4	2*4=8	3*4=12	4*4=16
                                                  //| 1*5=5	2*5=10	3*5=15	4*5=20	5*5=25
                                                  //| 1*6=6	2*6=12	3*6=18	4*6=24	5*6=30	6*6=36
                                                  //| 1*7=7	2*7=14	3*7=21	4*7=28	5*7=35	6*7=42	7*7=49
                                                  //| 1*8=8	2*8=16	3*8=24	4*8=32	5*8=40	6*8=48	7*8=56	8*8=64
                                                  //| 1*9=9	2*9=18	3*9=27	4*9=36	5*9=45	6*9=54	7*9=63	8*9=72	9*9=81
   for(a<- 1 to 9;b<- 1 to a;val s=if(b==a)"\r\n";else"\t")print(s"$b*$a=${b*a}$s")
                                                  //> 1*1=1
                                                  //| 1*2=2	2*2=4
                                                  //| 1*3=3	2*3=6	3*3=9
                                                  //| 1*4=4	2*4=8	3*4=12	4*4=16
                                                  //| 1*5=5	2*5=10	3*5=15	4*5=20	5*5=25
                                                  //| 1*6=6	2*6=12	3*6=18	4*6=24	5*6=30	6*6=36
                                                  //| 1*7=7	2*7=14	3*7=21	4*7=28	5*7=35	6*7=42	7*7=49
                                                  //| 1*8=8	2*8=16	3*8=24	4*8=32	5*8=40	6*8=48	7*8=56	8*8=64
                                                  //| 1*9=9	2*9=18	3*9=27	4*9=36	5*9=45	6*9=54	7*9=63	8*9=72	9*9=81
   //--for yield可以返回一个新的集合
   //--scala的集合类型涵盖：数组，链表，Set，Map，Range，Tuple等
   //--for yild for循环遍历的是什么类型，返回的就是什么类型
   val a2=for(i<-a1)yield{i*2}                    //> a2  : Array[Int] = Array(2, 4, 6, 8, 10)
   
   val l1=List(1,2,3)                             //> l1  : List[Int] = List(1, 2, 3)
   
   val l2=for(i<-l1)yield{i+1}                    //> l2  : List[Int] = List(2, 3, 4)
   
   val m1=Map("k1"->1,"k2"->2)                    //> m1  : scala.collection.immutable.Map[String,Int] = Map(k1 -> 1, k2 -> 2)
   
   for(i<-m1)println(i)                           //> (k1,1)
                                                  //| (k2,2)
   
   for((m,n)<-m1)println(n)                       //> 1
                                                  //| 2
    
    val v1="bbb"                                    //> v1  : String = bbb
  
    // match
  val result=v1 match{
  	case "aaa"=>{println("1");"1"}
  	case "bbb"=>{println("2");"2"}
  	case _=>{println("3");"3"}
  }
    
    //Exception
    try {
      throw new RuntimeException("error");
    }catch {
      case t: NullPointerException =>{t.printStackTrace();println("null")}
      case t: Exception=>{t.printStackTrace();println("other")}
    }finally {
      println("end")
   } 
}
```





## 函数

1. scala 通过def 关键字里声明一个函数
2. Unit 是函数的一种返回值类型，等同于java的void
3. scala可以自动根据方法体的返回值，推断出函数的返回值类型。

`注意：如果想让scala能够自动推断，必须有 = 号`

1. 如果函数没有 = ，则一律返回值类型都是 Unit类型
2. scala函数默认的访问权限是public。此外，可以加private或protected



## 匿名函数

1. 匿名函数没有函数名
2. 匿名函数的作用是配合高阶函数来使用，匿名函数可以作为函数的参数进行传递。
3. 如果函数的方法体只有一行代码，则方法体{}可以省略
4. 如果匿名函数的参数类型可以被推断出来，则类型可以省略
5. 如果匿名函数的参数列表只有一个，则()可以省略
6. 高阶函数可以将函数作为参数进行传递
7. 可以通过_(占位符)代替参数化简



## 集合类型

①数组 ②链表  ③set ④map ⑤tuple

1. 数组Array分定长和变长两种类型。后期常用的类型是定长(Array)
2. scala的泛型的声明使用 []来定义的。不同于java的<>
3. scala通过下标操作Array,使用(index)来操作，不同于java的[index]
4. Tuple 元组用()来声明元组。元组是最灵活的一种数据结构
5. 元组的取值 用 ._index (index的取值从1开始)



## 懒值，变长参数，柯里化

1. lazy懒值，在声明时不会马上赋值，只有当调用时才会赋值
2. lazy只用于 常量(val),不能用于变量(var)
3. scala支持变长参数（等价于java的可变参数）。形式是 参数类型*。
4. 变长参数类型本质上是一个数组，所以可以通过相关的api进行操作
5. 变长参数需要位于参数列表的最后
6. 柯里化技术可以将接受多个参数的函数转变为接受单一参数的函数
7. 柯里化技术可以结合高阶函数，允许用户自建控制结构

```scala
object Demo03 {
  println("Welcome to the Scala worksheet")       //> Welcome to the Scala worksheet
  
  val v1=100                                      //> v1  : Int = 100
  
  lazy val v2=100                                 //> v2: => Int
  
  //--定义变长参数
  def f1(num:Int*)={}                             //> f1: (num: Int*)Unit
  
  f1(1,2,3,4)
  
  def f2(num:Int*)={
  	for(i<-num)println(i)
  }                                               //> f2: (num: Int*)Unit
  
  f2(1,2,3,4)                                     //> 1
                                                  //| 2
                                                  //| 3
                                                  //| 4
 	
 	def f3(a:Int,b:Int)={a+b}                 //> f3: (a: Int, b: Int)Int
 	
 	def f31(a:Int)(b:Int)={a+b}               //> f31: (a: Int)(b: Int)Int
  f3(2,3)                                         //> res0: Int = 5
  f31(2)(3)                                       //> res1: Int = 5
  
  def f4(a:Int,b:Int,c:Int)={a+b+c}               //> f4: (a: Int, b: Int, c: Int)Int
 	
 	def f41(a:Int)(b:Int,c:Int)={a+b+c}       //> f41: (a: Int)(b: Int, c: Int)Int
 	def f42(a:Int,b:Int)(c:Int)={a+b+c}       //> f42: (a: Int, b: Int)(c: Int)Int
 	def f43(a:Int)(b:Int)(c:Int)={a+b+c}      //> f43: (a: Int)(b: Int)(c: Int)Int
 	
 	def f5(a:Int,b:Int,f:(Int,Int)=>Int)={f(a,b)}
                                                  //> f5: (a: Int, b: Int, f: (Int, Int) => Int)Int
 	def f51(a:Int,b:Int)(f:(Int,Int)=>Int)={f(a,b)}
                                                  //> f51: (a: Int, b: Int)(f: (Int, Int) => Int)Int
 	
 	def f52(a:Int)(b:Int,f:(Int,Int)=>Int)={f(a,b)}
                                                  //> f52: (a: Int)(b: Int, f: (Int, Int) => Int)Int
 	
 	def f53(a:Int)(b:Int)(f:(Int,Int)=>Int)={f(a,b)}
                                                  //> f53: (a: Int)(b: Int)(f: (Int, Int) => Int)Int
  f51(2,3)(_+_)                                   //> res2: Int = 5
}
```



## 重要的高阶函数

```scala
object Demo02 {
  println("Welcome to the Scala worksheet")       //> Welcome to the Scala worksheet
  
  val l1=List(1,2,3,4,5,6)                        //> l1  : List[Int] = List(1, 2, 3, 4, 5, 6)
  
  //--按照指定原则拆分。返回的是一个二元Tuple
  l1.partition { x => x%2==0 }                    //> res0: (List[Int], List[Int]) = (List(2, 4, 6),List(1, 3, 5))
  
  val l2=List("hadoop","world","hello","hello")   //> l2  : List[String] = List(hadoop, world, hello, hello)
  //--word->(word,1)
  
  l2.map { x => (x,1) }                           //> res1: List[(String, Int)] = List((hadoop,1), (world,1), (hello,1), (hello,1)
                                                  //| )

  val l3=List("hello world","hello hadoop")       //> l3  : List[String] = List(hello world, hello hadoop)
  
  l3.map { x => x.split(" ") }                    //> res2: List[Array[String]] = List(Array(hello, world), Array(hello, hadoop))
                                                  //| 
  //--扁平化map,会取出集合的元素。注意：此方法会概念集合中的元素个数
  //--一般引用场景：读取文件后，处理文件，将每行数据按指定分隔符切分
  l3.flatMap { x => x.split(" ") }                //> res3: List[String] = List(hello, world, hello, hadoop)
  
  //--要求，操作l3,做如下的转换:line->(word,1)
  l3.flatMap { x => x.split(" ") }.map { x =>(x,1) }
                                                  //> res4: List[(String, Int)] = List((hello,1), (world,1), (hello,1), (hadoop,1)
                                                  //| )
  //--过滤
  val l4=List(1,2,3,4,5)                          //> l4  : List[Int] = List(1, 2, 3, 4, 5)
  l4.filter { x => x>3 }                          //> res5: List[Int] = List(4, 5)
  
  //--等价于reduceLeft
  l4.reduce{_+_}                                  //> res6: Int = 15
  
  
  val l5=List(("bj",1),("sh",2),("bj",3),("sh",4),("sz",5))
                                                  //> l5  : List[(String, Int)] = List((bj,1), (sh,2), (bj,3), (sh,4), (sz,5))
  //--按照指定原则做分组，返回的是Map类型。Map的key是分组键，value是对应的List集合
  l5.groupBy{x=>x._1}                             //> res7: scala.collection.immutable.Map[String,List[(String, Int)]] = Map(bj ->
                                                  //|  List((bj,1), (bj,3)), sz -> List((sz,5)), sh -> List((sh,2), (sh,4)))
  l5.groupBy{case(addr,count)=>addr}              //> res8: scala.collection.immutable.Map[String,List[(String, Int)]] = Map(bj ->
                                                  //|  List((bj,1), (bj,3)), sz -> List((sz,5)), sh -> List((sh,2), (sh,4)))
  val m1=Map("rose"->23,"tom"->25,"jary"->30)     //> m1  : scala.collection.immutable.Map[String,Int] = Map(rose -> 23, tom -> 25
                                                  //| , jary -> 30)
  //--此方法是针对Map类型的value做操作。此方法只有Map类型有
  m1.mapValues { x => x+10 }                      //> res9: scala.collection.immutable.Map[String,Int] = Map(rose -> 33, tom -> 35
                                                  //| , jary -> 40)
  
  val l6=List((2,"aaa"),(1,"bbb"),(4,"ddd"),(3,"ccc"))
                                                  //> l6  : List[(Int, String)] = List((2,aaa), (1,bbb), (4,ddd), (3,ccc))
  
  l6.sortBy{x=> -x._1}                            //> res10: List[(Int, String)] = List((4,ddd), (3,ccc), (2,aaa), (1,bbb))
  l6.sortBy{x=>x._2}                              //> res11: List[(Int, String)] = List((2,aaa), (1,bbb), (3,ccc), (4,ddd))
  l6.sortBy{case(num,str)=>num}                   //> res12: List[(Int, String)] = List((1,bbb), (2,aaa), (3,ccc), (4,ddd))
  
  
  val l7=List("hello hadoop","hello world","hello spark",
  						"hello hadoop","hello hive")
                                                  //> l7  : List[String] = List(hello hadoop, hello world, hello spark, hello had
                                                  //| oop, hello hive)
  //--要求统计出每个单词出现的频次。
  //--最后的结果形式:(hello,5) (hadoop,2)……
  l7.flatMap { line =>line.split(" ") }.groupBy { word =>word }
  						.mapValues { list => list.size }.foreach{println(_)}
                                                  //> (world,1)
                                                  //| (hadoop,2)
                                                  //| (spark,1)
                                                  //| (hive,1)
                                                  //| (hello,5)
  //
  l7.flatMap { line =>line.split(" ") }.map { word => (word,1) }
  						.groupBy{x=>x._1}
  						.mapValues{list=>list.map{x=>x._2}.reduce(_+_)}.foreach{println(_)}
                                                  //> (world,1)
                                                  //| (hadoop,2)
                                                  //| (spark,1)
                                                  //| (hive,1)
                                                  //| (hello,5)
  //--要求 统计单词频次，然后返回频次最高的前2项结果
  //--返回的结果应该是：(hello,5),(hadoop,2)
  
  l7.flatMap { _.split(" ") }.map {(_,1)}.groupBy{_._1}
  						.mapValues{_.map{_._2}.reduce(_+_)}
  						.toList.sortBy{-_._2}.take(2)
                                                  //> res13: List[(String, Int)] = List((hello,5), (hadoop,2))
}
```

