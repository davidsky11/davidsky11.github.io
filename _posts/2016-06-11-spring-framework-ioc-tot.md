---
layout: post
title:  Spring IOC深入
date:   2016-06-11 08:08:00 +0800
categories: Spring
tag: [Spring, IOC]
---

* content
{:toc}

### IoC概述 ###
&emsp;&emsp;IoC是Spring容器的内核，AOP、声明式事务等功能都依赖于此功能，它涉及代码解耦、设计模式、代码优化等问题的考量。    
&emsp;&emsp;IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI(Dependency Injection，依赖注入)来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了Spring我们就只需要告诉Spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，Spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖Connection才能正常运行，而这个Connection是由Spring注入到A中的，依赖注入的名字就这么来的。那么DI是如何实现的呢？Java 1.3之后一个重要特征就是反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象中的方法、改变对象的属性，Spring就是通过反射来实现注入的。   

#### IoC思想 ####
&emsp;&emsp;Spring容器来实现这些相互依赖对象的创建、协调工作。对象只需要关心业务逻辑本身就可以了。   

#### IoC的注入类型 ####
> + 构造器注入：通过调用类的构造函数，将接口实现的类通过构造函数变量传入。
+ 属性注入：通过setter方法完成调用类所需依赖的注入，更加灵活方便。
+ 接口注入：将调用类所有依赖注入的方法抽取到一个接口中，调用类通过实现该接口提供相应的注入方法。

#### IoC的注入方式 ####
&emsp;&emsp;Spring作为一个容器，通过配置文件或者注解描述类和类之间的依赖关系，自动完成类的初始化和依赖注入的工作。

### Java反射机制 ###
&emsp;&emsp;Java语言允许通过程序化的方式间接对Class的对象实例操作，class文件有类加载器加载后，在JVM中将形成一份描述Class结构的元信息对象，通过该元信息对象可以获知class的结构信息，如构造函数、属性和方法等。

* ClassLoader： *    
&emsp;&emsp;类加载器就是寻找类的字节码文件并构造出类在JVM内部表示的对象组件，主要工作由classLoader及其子类负责，ClassLoader是一个重要的Java运行时系统组件，它负责在运行时查找和装入class字节码文件。

* ClassLoader工作机制： *     
装载：查找和导入class文件   
链接：执行校验，准备和解析步骤    
初始化：对类的静态变量、静态代码块执行初始化工作   

* Java反射机制： *   
&emsp;&emsp;Class反射对象描述类语义结构，可以从Class对象中获取构造函数，成员变量，方法等类元素的反射对象，并以编程的方式通过这些反射对象对目标类对象进行操作。这些反射类对象在java.reflet包中定义，下面是最主要的三个反射类：   
Constructor    
Method    
Field    

* Java反射机制与IoC的关系： *    
&emsp;&emsp;在Spring中，通过IoC可以将实现类、参数信息等配置在其对应的配置文件中，当需要更改实现类或参数信息时，只需要修改配置文件即可，还可以对某对象所需要的其他对象进行注入，这种注入方式都是在配置文件中做的。    
&emsp;&emsp;Spring的IoC的实现原理利用的就是Java的反射机制，Spring的工厂类会帮助我们完成配置文件的读取利用反射机制注入对象等工作，我们还可以通过Bean的名称获取对象的对象。    


### 资源访问工具类 ###
&emsp;&emsp;JDK所提供的资源访问类并不能很好的满足各种底层资源的访问需求，因此，Spring设计了一个Resource接口，它为应用提供了更强大的访问底层资源的能力：

> 主要方法：   
`boolean exists()`   
`boolean isOpen()`    
`URL getURL()`   
`File getFile()`    
`InputStream getInputStream()`   
主要实现类：   
ByteArrayResoure   
ClassPathResource   
FileSystemResource   
InputStreamResource   
ServletContextResource   
UrlResource   

&emsp;&emsp;为了访问不同的资源，必须使用相应的Resource实现类，这是比较麻烦的，Spring提供了一个强大的加载资源的机制，能够自动识别不同的资源类型

资源类型地址前缀：   
classpath classpath:    
File file:    
http http://    
ftp ftp://    

无前缀：   
Ant风格的匹配符：   
? 匹配文件名中的一个字符    
\* 匹配文件名中的任意字符    
\*\* 匹配多层路径   

Ant风格的资源路径示例：  
Classpath: com/spring-\*.xml     
File: /home/kv/spring/\*.xml    
Classpath: com/\*\*/context.xml    
classpath: org/springframework/\*\*/\*.xml      

### BeanFactory和ApplicationContext ###
&emsp;&emsp;BeanFactory是Spring框架的最核心的接口，它提供了高级IoC的配置机制。   
&emsp;&emsp;ApplicationContext建立在BeanFactory基础之上，提供了更多面向应用的功能，它提供了国际化支持和框架事件体系，更易于创建实际应用，一般称BeanFactory为IoC容器，而称ApplicationContext为应用上下文。   
&emsp;&emsp;BeanFactory是一个类工程，可以创建并管理各种类的对象，Spring称这些创建和管理的java对象为Bean。在Spring中,Java对象的范围更加宽泛。   

#### BeanFactory体系结构 ####
XmlBeanFactroy    
ListableBeanFactory   
HierarhicalBeanFactroy   
ConfigurableBeanFactory   
AutowireCapableBeanFactory   
SingletonBeanFactory   
BeanDefinitionRegistry   

Spring IoC容器的执行步骤：   
1、资源定位，即首先要找到applicationContext.xml文件   
2、BeanDefinition的载入，把xml文件中的数据统一加载到BeanDefinition中，以便后续处理  
3、向IoC容器中注入BeanDefinition数据    
4、将BeanDefinitioni中的数据进行依赖注入   

#### ApplicationContext ####
&emsp;&emsp;ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的功能。在BeanFactory中，很多功能需要以编程的方式实现，而在ApplicationContext中则可以通过配置文件的方式实现。   
> ApplicationContext实现类：    
ClassPathXmlApplicationContext   
FileSystemXmlApplicationcontext   
ConfigurableApplicationContext   

### Bean的生命周期 ###
&emsp;&emsp;Spring容器中的Bean拥有明确的生命周期，由多个特定的生命阶段组成，每个生命阶段都允许外接对Bean施加控制。

![](http://or9g8eqm7.bkt.clouddn.com/17-7-30/51771076.jpg)

![](http://or9g8eqm7.bkt.clouddn.com/17-7-30/65374505.jpg)