---
layout: post
title:  Java反射机制
date:   2015-06-09 00:00:00 +0800
categories: Java
tag: 反射
---

* content
{:toc}

## 1、基本概念 ##
　　Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能成为Java反射机制（<font color="red">程序访问、检测和修改它自身状态或行为的一种能力</font>）。
　　**Java反射机制主要用途：**
　　+ 运行时判断任意一个对象所属的类
　　+ 运行时构造任意一个类的对象
　　+ 运行时判断任意一个类所具有的成员变量和方法
　　+ 运行时调用任意一个对象的方法
　　*反射机制允许程序在运行时通过反射的API获取类中的描述，方法，并且允许我们在运行时改变fields内容或者去调用methods*
```
import java.lang.reflect.Method;

public class ForNameTest {
　　public static void main(String[] args) throws Exception {
　　　　// 获得Class
　　　　Class<?> cls = Class.forName(args[0]);
　　　　// 通过Class获得所对应对象的方法
　　　　Method[] methods = cls.getMethods();
　　　　// 输出每个方法名
　　　　for (Method method : methods) {
　　　　　　System.out.println(method);
　　　　}
　　}
}
```
## 2、Java Reflection APIs ##
　　在JDK中，主要由以下类来实现Java反射机制，这些类都位于java.lang.reflect包中
　　1、Class类：代表一个类。（注：这个Class类进行继承了Object，比较特别）
　　2、Field类：代表类的成员变量（成员变量也称为类的属性）
　　3、Method类：代表类的方法
　　4、Constructor类：代表类的构造方法
　　5、Array类：提供了动态创建数组，以及访问数组的元素的静态方法
　　
　　**使用Java反射机制遵循的步骤：**
　　1、获得操作类的Class对象
　　2、通过获得的Class对象去取得操作类的方法或属性名
　　3、操作取得的方法或属性
　　
　　Java运行的时候，某个类无论生成多少个对象，它们都会对应同一个Class对象，它表示正在运行程序中的类和接口。如何取得操作类的Class对象，常用的有三种方式：
　　a、调用Class的静态方法forName，如：Class.forName("java.lang.Class")；
　　b、使用类的.class语法，如：Class<?> cls = String.class；
　　c、调用对象的getClass方法，如：String str = "abc"; Class<?> cls = str.getClass();
　　
　　**实例：执行某对象的某个方法**
```
import java.lang.reflect.Method;

public class ReflectionTest {
　　public static void main(String[] args) throws Exception {
　　　　Display disPlay = new DisPlay();
　　　　// 获得Class
　　　　Class<?> cls = disPlay.getClass();
　　　　// 通过Class获得DisPlay类的show方法
　　　　Method method = cls.getMethod("show", String.class);
　　　　// 调用show方法
　　　　method.invoke(disPlay, "Kevin");
　　}
}

class DisPlay {
　　public void show(String name) {
　　　　System.out.println("Hello: " + name);
　　}
}
```
　　**实例：给某个类的属性赋值**
```
package com.wanggc.reflection;

import java.lang.reflect.Field;

/**
 * Java 反射之属性练习。
 * 
 * @author Wanggc
 */
public class ReflectionTest {
    public static void main(String[] args) throws Exception {
        // 建立学生对象
        Student student = new Student();
        // 为学生对象赋值
        student.setStuName("Wanggc");
        student.setStuAge(24);
        // 建立拷贝目标对象
        Student destStudent = new Student();
        // 拷贝学生对象
        copyBean(student, destStudent);
        // 输出拷贝结果
        System.out.println(destStudent.getStuName() + ":" + destStudent.getStuAge());
    }

    /**
     * 拷贝学生对象信息。
     * 
     * @param from
     *            拷贝源对象
     * @param dest
     *            拷贝目标对象
     * @throws Exception
     *             例外
     */
　　private static void copyBean(Object from, Object dest) throws Exception {
        // 取得拷贝源对象的Class对象
        Class<?> fromClass = from.getClass();
        // 取得拷贝源对象的属性列表
        Field[] fromFields = fromClass.getDeclaredFields();
        // 取得拷贝目标对象的Class对象
        Class<?> destClass = dest.getClass();
        Field destField = null;
        for (Field fromField : fromFields) {
            // 取得拷贝源对象的属性名字
            String name = fromField.getName();
            // 取得拷贝目标对象的相同名称的属性
            destField = destClass.getDeclaredField(name);
            // 设置属性的可访问性
            fromField.setAccessible(true);
            destField.setAccessible(true);
            // 将拷贝源对象的属性的值赋给拷贝目标对象相应的属性
            destField.set(dest, fromField.get(from));
        }
    }
}

/**
 * 学生类。
 */
class Student {
    /** 姓名 */
    private String stuName;
    /** 年龄 */
    private int stuAge;

    /**
     * 获取学生姓名。
     * 
     * @return 学生姓名
     */
    public String getStuName() {
        return stuName;
    }

    /**
     * 设置学生姓名
     * 
     * @param stuName
     *            学生姓名
     */
    public void setStuName(String stuName) {
        this.stuName = stuName;
    }

    /**
     * 获取学生年龄
     * 
     * @return 学生年龄
     */
    public int getStuAge() {
        return stuAge;
    }

    /**
     * 设置学生年龄
     * 
     * @param stuAge
     *            学生年龄
     */
    public void setStuAge(int stuAge) {
        this.stuAge = stuAge;
    }
}
```
　　**实例：在运行时创建类的一个对象**
```
import java.lang.reflect.Field;

/**
 * Java 反射之属性练习。
 * 
 * @author Wanggc
 */
public class ReflectionTest {
    public static void main(String[] args) throws Exception {
        // 建立学生对象
        Student student = new Student();
        // 为学生对象赋值
        student.setStuName("Wanggc");
        student.setStuAge(24);
        // 建立拷贝目标对象
        Student destStudent = (Student) copyBean(student);
        // 输出拷贝结果
        System.out.println(destStudent.getStuName() + ":" + destStudent.getStuAge());
    }

    /**
     * 拷贝学生对象信息。
     * 
     * @param from
     *            拷贝源对象
     * @param dest
     *            拷贝目标对象
     * @throws Exception
     *             例外
     */
    private static Object copyBean(Object from) throws Exception {
        // 取得拷贝源对象的Class对象
        Class<?> fromClass = from.getClass();
        // 取得拷贝源对象的属性列表
        Field[] fromFields = fromClass.getDeclaredFields();
        // 取得拷贝目标对象的Class对象
        Object ints = fromClass.newInstance();
        for (Field fromField : fromFields) {
            // 设置属性的可访问性
            fromField.setAccessible(true);
            // 将拷贝源对象的属性的值赋给拷贝目标对象相应的属性
            fromField.set(ints, fromField.get(from));
        }

        return ints;
    }
}

/**
 * 学生类。
 */
class Student {
    /** 姓名 */
    private String stuName;
    /** 年龄 */
    private int stuAge;

    /**
     * 获取学生姓名。
     * 
     * @return 学生姓名
     */
    public String getStuName() {
        return stuName;
    }

    /**
     * 设置学生姓名
     * 
     * @param stuName
     *            学生姓名
     */
    public void setStuName(String stuName) {
        this.stuName = stuName;
    }

    /**
     * 获取学生年龄
     * 
     * @return 学生年龄
     */
    public int getStuAge() {
        return stuAge;
    }

    /**
     * 设置学生年龄
     * 
     * @param stuAge
     *            学生年龄
     */
    public void setStuAge(int stuAge) {
        this.stuAge = stuAge;
    }
}
```

**补充：**在获得类的方法/属性/构造函数时，会有getXXX和getDeclaredXXX两种对应的方法。之间的区别在于前者返回的是访问权限为public的方法和属性，包括父类中的；而后者返回的是所有访问权限的方法和属性，不包括父类的。
## 3、Class类是Reflect API中的核心类 ##
　　(1) getName()：获得类的完整名字
　　(2) getFields()：获得类的public类型的属性。
　　(3) getDeclaredFields()：获得类的所有属性。
　　(4) getMethods()：获得类的public类型的方法。
　　(5) getDeclaredMethods()：获得类的所有方法。
　　(6) getMethod(String name, Class[] parameterTypes)：获得类的特定方法，name参数指定方法的名字parameterTypes参数指定方法的参数类型。
　　(7) getConstructors()：获得类的public类型的构造方法。
　　(8) getConstructor(Class[] parameterTypes)：获得类的特定构造方法，parameterTypes参数指定构造方法的参数类型。
　　(9) newInstance()：通过类的不带参数的构造方法创建这个类的一个对象。
　　
　　先看上面的(8)和(9)其中都能生存对象，但是因为构造函数有无参合有参构造函数两种，所以我们分两种情况考虑
　　**情况一：无参的构造函数来生成对象**
　　a、首先我们去获取Class对象，然后直接通过Class对象去调用newInstance()方法就可以
```
Class<?> tclass = Reflection2.class;
Object reflection2 = classType.newInstance();
```
　　b、首先我们也是去获取Class对象,然后去去调用getConstructor（）得到Constructor对象，接着直接调用newInstance()即可
```
Class<?> classType = Reflection2.class;
// Object reflection2 = classType.newInstance();
Constructor<?> constructor = classType.getConstructor(new Class[] {});
Object reflection2 = constructor.newInstance(new Object[] {});
```
　　**情况二:现在是有参构造函数，那我们只有一种方法来通过反射生成对象:** 
```
Class<?> tClass = Person.class;  
Constructor cons = classType.getConstructor(new Class[]{String.class, int.class});   
Object obj = cons.newInstance(new Object[]{“zhangsan”, 19});
```

###通过反射机制调用对象的私有方法，访问对象的私有变量###
　　要实现这种功能，我们需要用到AccessibleObject类中的public void setAccessible(boolean flag)方法：
　　使用这个方法，把参数flag设置成true，然后我们的field或者method就可以绕过Java语言的语法访问的检查
　　具体使用如下：
　　1、使用反射去访问私有方法
```
package com.jiangqq.reflection;

public class Test01 {
	private String getName(String name) {
		return "This i:" + name;
	}
}

package com.jiangqq.reflection;

import java.lang.reflect.Method;

public class TestPrivate01 {
	public static void main(String[] args) throws Exception {
		Test01 p = new Test01();
		Class<?> classType = p.getClass();
		Method method = classType.getDeclaredMethod("getName",
				new Class[] { String.class });
		method.setAccessible(true);
		Object object = method.invoke(p, new Object[] { "tom" });
		System.out.println((String)object);
	}
}
```
　　2、使用反射机制去访问私有变量
```
package com.jiangqq.reflection;

public class Test02 {
  private String name="张三";
  private String getName()
  {
	  return name;
  }
}


package com.jiangqq.reflection;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class TestPrivate02 {
	public static void main(String[] args) throws Exception {
		Test02 p = new Test02();
		Class<?> classType = p.getClass();
		Field field = classType.getDeclaredField("name");
		//设置true,使用可以绕过Java语言规范的检查
		field.setAccessible(true);
		//对变量进行设置值
		field.set(p, "李四");
		Method method = classType.getDeclaredMethod("getName", new Class[] {});
		method.setAccessible(true);
		Object object = method.invoke(p, new Object[] {});
		System.out.println((String) object);
	}
}

```

