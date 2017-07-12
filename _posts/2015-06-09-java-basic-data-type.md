---
layout: post
title:  Java数据类型
date:   2015-06-09 00:00:00 +0800
categories: Java
tag: Java
---

* content
{:toc}

### 一、Java基本数据类型 ###
  基本类型，或者内置类型，是Java中不同于类额特殊类型。基本类型共有八种，它们分别都有相对应的包装类。<br/>
　　**基本类型可以分为三类：**
　　*  字符类型char
　　*  布尔类型boolean
　　*  数值类型byte、short、int、long、float、double
　　<font color="red">数值类型又可以分为整数类型byte、short、int、long和浮点数类型float、double</font>
　　
　　Java中还存在另外一种基本类型void，它也有对应的包装类java.lang.Void，不过我们无法直接对它们进行操作。
　　
　　基本类型存储在栈中，因此它们的存取速度要快于存储在堆终端额对应包装类的实例对象。
　　所有基本类型（包括void）的包装类都使用了final修饰，因此我们无法继承它们扩展新的类，也无法重写它们的任何方法。

### 二、Java数据类型转换 ###

#### 1、Java基本数据类型转换 ####
　　Java语言是一种强类型的语言。强类型的语言有以下几个要求：<br/>
　　(1) 变量或常量必须有类型：要求声明变量或常量时必须声明类型，而且只能在声明以后才能使用。<br/>
　　(2) 赋值时类型必须一致：值得类型必须和变量或常量的类型完全一致。<br/>
　　(3) 运算时类型必须一致：参与运算的数据类型必须一致才能运算。<br/>

　　**Java语言中的数据类型转换有两种：**<br/>
　　自动类型转换：编译器自动完成类型转换，不需要再程序中国编写代码。<br/>
　　强制类型转换：强制编译器进行类型转换，必须在程序中编写代码。<br/>
　　
　　这两种类型转换的规则、适用场合以及使用时注意事项：<br/>
　
　　**1、自动类型转换**
　　隐式类型转换，是指不需要书写代码，由系统自动完成的类型转换，由JVM自动完成。
　　**转换规则：从存储范围小的类型到存储范围大的类型。**
　　具体规则为： byte -> short(char) -> int -> long -> float -> double

　　**注意问题：**
　　a、引用类型能够自动转换为父类
　　a、基本类型和它们的包装类型是能够互相转换的
　　c、在整数之间进行类型转换的时候数值不会发生变化，但是当将整数类型特别是比较大的整数类型转换成小数类型的时候，由于存储精度的不同，可能会存在数据精度的损失。
　　
　　**2、强制类型转换**
　　显式类型转换，是指必须书写代码才能完成的类型转换。该类型转换很可能存在精度的损失，所以必须书写相应的代码，并且能够忍受该种损失时才进行该类型的转换。
　　**转换规则：从存储范围大的类型到存储范围小的类型。**
　　**具体规则为：** double -> float -> long -> int -> short(char) -> byte
　　**语法格式为：**（转换到的类型）需要转换的值
　　**注意问题：**强制类型转换通常都会存储精度的损失，所以使用时需要谨慎。

#### 2、包装类过渡类型转换 ####
　　一般情况下，我们首先声明一个变量，然后生成一个对应的包装类，就可以利用包装类的各种方法进行类型转换。
　　1、把float类型转换为double类型：
```
　　float f1 =100.00f;
　　Float F1 = new Float(f1);
　　double d1 = f1.doubleValue();
```
　　2、把double类型转换为int类型
```
　　double d1 = 100.00;
　　Double D1 = new Double(d1);
　　int i1 = d1.intValue();
```
　　简单类型的变量转换为相应的包装类，可以利用包装类的构造函数。即Boolean(boolean value)、Character(char value)、Integer(int value)、Long(long value)、Float(float value)、Double(double value)。
　　而在各个包装类中，总有形如xxxValue()的方法，来得到其对应的简单类型数据。

#### 3、字符串与其他类型转换 ####
　　**其他类型向字符串转换**
　　a、调用类的串转换方法：X.toString()；
　　b、自动转换：X+"";
　　c、使用String的方法：String.valueOf(X)
　　
　　**字符串向其他类型转换**
　　a、先转换成相应的封装器实例，再调用对应的方法转换成其它类型（Double.valueOf("32.1").doubleValue()）
　　b、静态**parseXXX**方法
```
String s = "1";
byte b = Byte.parseByte(s);
short s = Short.parseShort(s);
int i = Integer.parseInt(s);
long l = Long.parseLong(s);
Float f = Float.parseFloat(s);
Double d = Double.parseDouble(s);
```
　　c、Character的getNumericValue(char c)方法

#### 4、Date类型与其他类型转换 ####
　　整型和Date类之间并不存在直接的对应关系，只是你可以使用int型为分别表示年、月、日、时、分、秒，这样就在两者之间建立了一个对应关系。
　　a、Date(int year, int month, int date)：以int类型表示年月日
　　b、Date(int year, int month, int date, int hrs, int min)：以int类型表示年月日时分
　　c、Date(int year, int month, int date, int hrs, int min, int sec)：以int类型表示年月日时分秒
　　将一个时间表示为距离格林尼治时间1970年1月1日0时0分0秒的毫秒数。Date类型有其对应的构造函数：Date(long date)。
　　得到Date的特定格式：
```
import java.text.SimpleDateFormat;
import java.util.*;

java.util.Date date = new java.util.Date();

//如果希望得到YYYYMMDD的格式
SimpleDateFormat sy1=new SimpleDateFormat("yyyyMMDD");
String dateFormat=sy1.format(date);

//如果希望分开得到年，月，日
SimpleDateFormat sy=new SimpleDateFormat("yyyy");
SimpleDateFormat sm=new SimpleDateFormat("MM");
SimpleDateFormat sd=new SimpleDateFormat("dd");
String syear=sy.format(date);
String smon=sm.format(date);
String sday=sd.format(date);
```

**总结：**
　　只有boolean不参与数据类型的转换
　　
#### 5、Java引用类型 ####
　　Java有5中引用类型（对象类型）：
　　**类 接口 数组 枚举 标注**
　　引用类型：底层结构和基本类型差别较大
　　JVM的内存空间：
　　(1) Heap堆空间：分配对象 new Student()
　　(2) Stack栈空间：临时变量 Student stu;
　　(3) Code代码区：类的定义，静态资源 Student.class
```
Student stu = new Student(); // new在内存的堆空间创建对象
stu.study();  // 把对象的地址付给stu引用变量
```
　　上例实现步骤：
　　a、JVM加载Student.class到Code区
　　b、new Student()在堆空间分配空间并创建一个Student实例
　　c、将此实例的地址赋值给引用stu，栈空间


----------
### 应该知道的 ###

#### 1、值传递和引用传递 ####

　　Java参数，不管是原始类型还是引用类型，传递的都是副本（有另外一种说法是传值，但是说传副本更好理解吧，传值通常是相对传址而言）。
　　如果参数类型是原始类型，那么传过来的就是这个参数的一个副本，也就是这个原始参数的值，这个跟之前所谈的传值是一样的。如果在函数中改变了副本的值不会改变原始的值。
　　如果参数类型是引用类型，那么传过来的就是这个引用参数的副本，这个副本存放的是参数的地址。如果在函数中没有改变这个副本的地址，而是改变了地址中的值，那么在函数内的改变会影响到传入的参数。如果在函数中改变了副本的地址，如new一个，那个副本就指向了一个新的地址，此时传入的参数还是指向原来的地址，所以不会改变参数的值。
　　<font color="red">
　　在Java里面只有基本类型和按照下面这种定义方式的String是按值传递，其他都是按引用传递。就是直接使用双引号定义字符串方式：String str="test";
　　</font>
　　值传递重要特点：传递的是值得拷贝，也就是传递后就互不相关了。
　　引用传递重要特点：传递前和传递后都指向同一个引用（也就是同一个内存空间）
　　
　　对象就是传引用
　　原始类型就是传值
　　String，Integer，Double等包装类型因为没有提供自身修改的函数，每次操作都是新生成一个对象，所以要特殊对待，可以认为是传值。在方法内对它们赋值都会导致原有类型的引用被指向方法内的栈地址，失去了对原类变量地址的指向。对赋值后的类型对象做任何操作，都不会影响原来对象。<br/>　
    ![](http://or9g8eqm7.bkt.clouddn.com/17-6-9/56015306.jpg)

　　**总结：**
　　1、对于基本类型，在方法体内对方法参数进行重新赋值，并不会改变原有变量的值。
　　2、对于引用类型，在方法体内对方法参数进行重新赋值引用，并不会改变原有变量所持有的引用。
　　3、方法体内对参数进行运算，不影响原有变量的值。
　　4、方法体内对参数所指向对象的属性进行运算，将改变原有变量所指向对象的属性值。其次，对象和引用型变量被当作参数传递给方法时，在方法实体中，无法重新给原变量重新赋值，但是可以改变它所指向对象的属性。至于到底它是值传递还是引用传递，这并不重要，重要的是我们要清楚当一个引用被作为参数传递给一个方法时，在这个方法体内会产生什么变化，这才是最重要的。

----------
三、常见面试题
---

 1. short s1=1; s1=s1+1; 有什么错？ short s1=1; s1+=1; 有什么错？
　　对于short s1=1;s1=s1+1来说，在s1+1运算时会自动提升表达式的类型为int，那么将int赋予给short类型的变量s1会出现**类型转换错误**。
　　对于short s1=1;s1+=1来说 **+=**是Java语言规定的运算符，Java编译器会对它进行特殊处理，因此可以正确编译。

 2. char类型变量能不能存储一个中文的汉字，为什么？
　　char类型变量是用来储存Unicode编码的字符的，Unicode字符集包含了汉字，所以char类型当然可以存储汉字，还有一种特殊情况就是某个生僻字没有包含在Unicode编码字符集中，那么char类型就不能存储该生僻字。

 3. Integer和int的区别
 　　int是Java的八种内置的原始数据类型之一。Java为每个原始类型都提供了一个封装类，Integer是int的封装类。
　　int变量的默认值是0，Integer变量的默认值是null，这一点说明Integer可以区分出未赋值和值为0的区别。
　　Integer类内提供了一些关于整数操作的一些方法，例如整数的最大值和最小值等。

 4. switch语句能否作用在byte上，能否作用在long上，能否作用在String上？
 　　byte的存储范围小于int，可以向int类型进行隐式转换，所以switch可以作用在byte上
　　long的存储范围大于int，不能像int类型进行隐式转换，所以switch不可以作用在long上
　　String在JDK1.7版本之前不可以，1.7版本之后可以。

 5. 当一个对象被当作参数传递到一个方法后，此方法可改变这个对象的属性，并可返回变化后的结果，那么这里到底是值传递还是引用传递？
　　**值传递。** 在Java里面参数传递都是按值传递（<font color="red">按值传递是传递的值的拷贝，按引用传递其实传递的是引用的地址值，所以统称按值传递</font>）。当一个对象实例作为一个参数被传递到方法中，参数的值就是该对象的引用一个副本。指向同一个对象，对象的内容可以在别调用的方法中改变，但对象的引用（不是引用的副本）是永远不会改变的。
 6. 
[1]: http://g.hiphotos.baidu.com/zhidao/pic/item/d6ca7bcb0a46f21f00cce9d0f2246b600d33aebd.jpg