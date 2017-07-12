---
layout: post
title:  Java泛型
date:   2015-06-10 00:00:00 +0800
categories: Java
tag: [Java, 泛型]
---

* content
{:toc}

> Java泛型（generics）是JDK5中引入的一个新特性，允许在定义类/接口/方法的时候使用类型参数（type parameter）。声明的类型参数在使用时用具体的类型来替换。

### 1.为什么使用泛型 ###
&emsp;&emsp;泛型的好处在于编写重用性更好的代码，用来指定容器要持有什么类型的对象，由编译器保证类型的正确性，不需要强制类型转换。

### 2.基本使用 ###
#### 2.1.泛型术语 ####
+ List 泛型类型，参数化类型，原始类型
+ List中的E 类型变量或类型参数
+ List中的String 实际类型参数

#### 2.2.泛型类 ####
&emsp;&emsp;创建类时，指明想持有的对象，将其置于尖括号之内。常见使用情景**元组类库**：是一个单一对象，将一组对象直接打包存储于其中。这个容器对象允许读取其中的元素，但是不允许向其中存放新的对象。
```
public class TwoTuple<A, B> {
    public final A first;
    public final B second;
    public TwoTuple(A a, B b) {  
        first = a;
        second = b;
    }
}
```
&emsp;&emsp;泛型接口定义类似于泛型类。

#### 2.3.泛型方法 ####
&emsp;&emsp;泛型方法使得该方法能够独立于类而产生变化，建议在使用泛型方法可以取代整个类泛型的时候选择泛型方法。定义泛型方法，只需要泛型参数列表置于返回值之前。
```
public class Generators{
    public static  Collection fill(Collection coll, Generator gne, int n){
        for(int i = 0; i < n;i++){
            coll.add(gen.next());
        }
        return coll;
    }
}
```
**泛型方法使用注意点：**
&emsp;&emsp;定义方法所用的泛型参数需要在修饰符之后添加，如public static，如果有多个泛型参数，可以如此定义<K, V>。类型参数推断，区别于泛型类在使用时候必须指定类型参数值，泛型方法则不必，编译器可以为我们找出具体类型，可以像调用普通方法一样调用`fill()`方法。

### 3.泛型擦除 ###
> Java的泛型是在编译器层面实现的，生成的字节码中是不包含泛型中的类型信息的。

&emsp;&emsp;在JAVA的虚拟机中并不存在泛型，泛型只是为了完善java体系，增加程序员编程的便捷性以及安全性而创建的一种机制，在JAVA虚拟机中对应泛型的都是确定的类型。   
&emsp;&emsp;在编写泛型代码后，java虚拟中会把这些泛型参数类型都擦除，用相应的确定类型来代替，代替的这一动作叫做**类型擦除**，而用于替代的类型称为原始类型，**在类型擦除过程中，一般使用第一个限定的类型来替换**，若无限定则使用Object。

**有界类擦除前：**
```
public class Eraser {    
    public static > A max(Collection xs) { 
        Iterator xi = xs.iterator(); 
	A w = xi.next(); 
	while (xi.hasNext()) { 
	    A x = xi.next(); 
	    if (w.compareTo(x) < 0) w = x; 
	} 
	return w; 
    } 
}
```

**有界类擦除后：**
```
public class Eraser {    
    public static Comparable max(Collection xs) {        
        Iterator xi = xs.iterator();        
        Comparable w = (Comparable)xi.next();        
        while (xi.hasNext()) {            
            Comparable x = (Comparable)xi.next();            
            if (w.compareTo(x) < 0)                
                w = x;        
            }        
        return w;    
    }
}
```

类型参数的边界>A，A必须为Comparable的子类，按照类型擦除的过程，先将所有的类型参数转换为最左边界Comparable，然后去掉参数类型A，得到最终的擦除后结果。

**多边界擦除**   
规则：使用其最左限制的擦除，即用第一个边界的类型变量来替换。  
如果类Pair声明如下：
    public class Pair { }
那么原始类型就是Comparable

### 4.边界与通配符 ###
&emsp;&emsp;边界使得你可以用于泛型的参数类型上设置限制条件，潜在效果你可以按照自己边界类型调用方法。

#### 4.1.边界定义 ####
&emsp;&emsp;这表示T应该是BoundingType的子类型，T和BoundingType可以是类，也可以是接口。另外，此处的extends表示的是子类型，不等同于继承。

#### 4.2.边界限定通配符 ####
<? extends T> 表示类型的上界，表示参数化类型的可能是T或是T的子类。    
<? super T> 表示类型下界，表示参数化类型是此类型的超类型（父类型），直至Object    
<?> 表示类型无界，常见使用多个泛型参数允许其中某些为任何类型，例如：Map<String,?>    

如果要从集合中读取类型T的数据，并且不能写入，可以使用 ? extends 通配符；(Producer Extends)   
如果要从集合中写入类型T的数据，并且不需要读取，可以使用 ? super 通配符；(Consumer Super)   
如果既要存又要取，那么就不要使用无界通配符。 
```
// demo1使用上界写入会编译错误
List<? extends Fruit> frutis = new ArrayList();
fruits.add(new Apple());// complie error
fruits.add(new Fruit());// complie error
// demo2使用下界写入正常
List<? super Apple> frutis = new ArrayList();
fruits.add(new Apple());
```

### 5.注意问题 ###
不能用基本类型实例化类型参数   
不能实例化类型变量，例如new T()，T.class等   
List.class,String[].class,int.class都是允许的，但就List.class和List<?>.class则不合法   
和 之间没有继承关系，所以List= new ArrayList;会报编译错误的，上界和下界为了解决该问题引入。   
类型安全和异构容器： 一个集合如果可以预测返回值的类型，则是类型安全的；如果这个集合的键并不是同一个类型的，则称之为异构的。通过包装一个Map，将其中的key保存为Class的，value保存为Object的，可以实现一个异构容器。