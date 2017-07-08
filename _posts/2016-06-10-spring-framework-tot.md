---
layout: post
title:  Spring框架总结
date:   2015-06-10 00:00:00 +0800
categories: Spring
tag: spring
---

* content
{:toc}

Spring简介
------

1、Spring是什么？

&emsp;&emsp;在了解Spring之前，我们来了解在J2EE框架下企业级开发采用EJB框架的一些不足：

+ EJB太笨重，而且Entity EJB不能脱离容器
+ 企业级服务使用困难
+ 开发的复杂度太高
+ 侵入式方案，EJB要使用特定的接口

　　因此，Spring应运而生。

　　**Spring是一个开源的用于简化采用Java语言开发企业级程序的一个分层的框架。**
	
关于程序的分层结构：

**1、Presentation layer（表示层）**
+ 表示逻辑（生成界面代码）
+ 接收请求
+ 处理业务层抛出的异常
+ 负责规则验证（数据格式，数据非空等）
+ 流程控制

**2、Service layer（服务层/业务层）**
+ 封装业务逻辑处理，并且对外暴露接口
+ 负责事务，安全等服务

**3、Persistence layer（持久层）**
+ 封装数据访问的逻辑，暴露接口
+ 提供方便的数据访问的方案（查询语言，API，映射机制等）

**4、Domain layer（域层）**
+ 业务对象以及业务关系的表示
+ 处理简单的业务逻辑
+ 域层的对象可以穿越表示层，业务层，持久层
　　
![此处输入图片的描述][1]


Spring的作用
------

为什么要使用Spring？

1、简化企业级开发
&emsp;&emsp;封装了大部分的企业级服务，提供了更好的访问这些服务的方式
&emsp;&emsp;提供了IOC，AOP功能的容器，方便编程

2、遵循Spring框架的应用程序，一定是设计良好的，针对接口编程，这样就简化了企业级程序的设计。

3、Spring的组成
+ **Spring Core：** 核心容器，BeanFactory提供了组件生命周期的管理，组件的创建，装配，销毁等功能
+ **Spring Context：** ApplicationContext，扩展核心容器，提供事件处理、国际化等功能。它提供了一些企业级服务的功能，提供了JNDI，EJB，RMI的支持。
+ **Spring AOP：** 提供切面支持
+ **Spring DAO：** 提供事务支持，JDBC，DAO支持
+ **Spring ORM：** 对流行的O/R Mapping封装或支持
+ **Spring Web：** 提供Web应用上下文，对Web开发提供功能上的支持，如请求，表单，异常等
+ **Spring Web MVC：** 全功能MVC框架，作用等同于Struts。

----------

Spring的IoC
------

### IoC的概念 ###
　　IoC，Inversion of Control，控制反转。
　　对象的协作关系由对象自己负责。
　　依赖注入：对象的协作关系由容器来建立。
### IoC的类型 ###
1、基于特定接口（侵入式方案）
2、基于set方法
3、基于构造器
### Spring容器 ###
　　Spring容器负责生成、组装、销毁组件，并负责事件处理、国际化等功能。
　　1、BeanFactory< interface>
　　　　核心容器，负责组件生成和装配
　　　　实现主要包括XmlBeanFactory
　　2、ApplicationContext
　　3、WebApplicationContext

### IoC的使用 ###

**Resource：** interface，用来包装资源

**XmlBeanFactory：** BeanFactory的一个实现，使用Resource对象来查找配置文件

**BeanFactory.getBean("beanId")：**取得以参数命名，或者Id等于参数值的一个Bean实例。
&emsp;&emsp;BeanFactory（容器）在默认情况下，会采用单例方式返回对象。容器只到调用getBean方法时，才会实例化某个对象。
(1) Spring可以采用XML或者.properties文件作配置
(2) 配置文件（xml）
　　根元素< beans>可以有多个< bean>子元素，每个< bean>代表一个需要装配的对象。
### 1、setter注入###

a、注入简单属性（String和八种基本类型）
```　　
<beans>
    <bean id=" beanId" class=" classpath" autowire=" " dependency-check=" " >
        <property name=" parameterName" >
            <value>parameterValue</value>
        </property>
    </bean>
</beans>
```
　　
　　对于基本类型，Spring容器会自动作类型转换，以便赋值。
　　
b、注入对象
```　　
<bean>
    <ref local=" beanId">
<bean>
```
+ 让Spring容器在当前配置文件中找到相应的Bean，并调用set方法，注入该Bean。
+ 将一个Bean的定义嵌套在另一个Bean中（可读性差），被嵌套的Bean不能采用getBean()返回
+ 采用<ref bean=" ">搜索多个配置文件来注入

　　c、注入集合类型
　　c.1、Set
　　　　Set中存放字符串，对象，集合，不允许重复
　　c.2、List
　　　　List中可以放入字符串，对象，List
　　c.3、Map
　　　　Map有< entity>子元素来存取key，value，key只能为String
　　c.4、Properties
　　　　Properties有< props>子元素
　　　　
### 2、Constructor注入 ###
```　　
<bean>
    <constructor-arg>
        <value> </value>
    </constructor-arg>
    <constructor-arg>
        <ref bean=" " />
    </constructor-arg>
</bean>
```

　　如果Bean属性不多，并且属性值必须要注入才能使用，则应该采用constructor注入，其他情况就要set方法注入。
　　*装配关系检查（Dependency checking）*
　　simple：检查简单类型
　　objects：检查对象类型
　　all：检查所有
```
<bean dependency-check="all"></bean>
```
　　
　　**自动装配（Autowring Properties）**
　　装配方式：byName，byType，autodetect，constructor
　　**1、autowire="byName"：按照名称匹配**
　　按照Bean的Id与属性的名称进行匹配
　　自动装配与手动装配可以结合使用，手动装配会覆盖自动装配。
　　**2、autowire="byType"：按照类型匹配**
　　要注入的属性类型与配置文件中的Bean类型匹配的个数超过一个，会出错。
　　**3、autowire="autodetect"**
　　先按照constructor，后按照byType。
　　**4、autowire="constructor"**
　　先去匹配构造器中参数类型，后与配置文件中的Bean类型匹配。

### 3、比较两种注入方式 ###
　　关于自动匹配：
　　优点：快速开发
　　缺点：依赖关系不清楚，易出错，不易维护。
　　自动匹配的应用场合：
　　(1) 构建系统原型
　　(2) 与依赖关系检查（Dependency-check）结合使用
　　(3) 自动装配与手动装配结合
### 4、特殊的IoC ###
　　**4.1、后处理Bean**
　　接口：org.springfrwmework.beans.factory.config.BeanPostProcessor
　　Spring已经实现该接口的BeanPostProcessor（不用再注册）
　　ApplicationContextAwareProcessor：
　　把应用上下文传递给所有实现了ApplicationContextAware接口的Bean
　　ApplicationContextAware接口使用举例，可参照时间监听机制
　　DefaultAdvisorAutoProxyCreator自动对Bean应用切面
　　
　　**4.2、Bean工厂后处理（只能在应用上下文中使用）**
　　接口：org.springframework.beans.factory.config.BeanFactoryPostProcessor
　　Spring内部接口实现：
　　org.springframework.beans.factory.config.PropertyPlaceholderConfigurer
　　属性编辑
　　org.springframework.beans.factory.config.CustomEditorConfigurer
### 5、事件处理 ###
　　事件监听
　　(1) 自定义事件，通过继承org.springframework.context.ApplicationEvent
　　(2) 自定义监听器，实现org.springframework.context.ApplicationListener，并注册
　　(3) 发布事件，为得到应用上下文，必须实现org.springframework.context.ApplicationContextAware接口


----------
## Spring的AOP ##
### 1、什么是AOP？ ###
　　Aspect-oriented programming，面向切面编程。
　　**定义：**将程序中的交叉业务逻辑提取出来，称之为切面。将这些切面动态织入到目标对象，然后生成一个代理对象的过程。
### 2、AOP核心概念 ###

 1. Aspect（切面）
   切面，是对交叉业务逻辑的统称。
 2. Joinpoint（连接点）
   连接点，指切面可以织入到目标对象的位置（方法，属性等）。
 3. Advice（通知）
   通知，指切面的具体实现。
 4. Pointcut（切入点）
   切入点，指通知应用到哪些类的哪些方法或属性之上的规则。
 5. Introduction（引入）
   引入，指动态地给一个对象增加方法或属性的一种特殊的通知。
 6. Weaving（织入）
   织入，指将通知插入到目标对象。
 7. Target（目标对象）
   目标对象，指需要织入切面的对象。
 8. Proxy（代理对象）
   代理对象，指切面织入目标对象之后形成的对象。
### 3、Spring AOP原理 ###
　　采用**动态代理**模式。
　　Spring AOP采用动态代理的过程：
　　(1) 将切面使用动态代理的方式织入到目标对象（被代理类），形成一个代理对象；
　　(2) 目标对象如果没有实现代理接口，那么Spring会采用CGLib来生成代理对象，该代理对象是目标对象的子类；
　　(3) 目标对象如果是final类，并且也没实现代理接口，就不能运用AOP。

### 4、Spring的通知 ###

 **1、 Spring的通知类型**
 
 　　**1.1、MethodBeforeAdvice**
 　　类全名：org.springframework.op.MethodBeforeAdvice
 　　在方法调用之前，做处理。
 　　不能够改变返回值
 　　不能够改变目标方法的流程，也不能中断流程的处理过程（除非抛出异常）
 　　**1.2、AfterReturningAdvice**
 　　类全名：org.springframework.aop.AfterReturningAdvice
 　　在方法调用之后，做处理。
 　　不能够改变返回值
 　　不能够改变目标方法的流程，也不能中断流程的处理过程（除非抛出异常）
 　　**1.3、MethodInterceptor**
 　　类全名：org.springframework.intercept.MethodIntercepteor
 　　在方法调用之前以及之后，做处理。
 　　可以改变返回值，也可以改变流程。
 　　**1.4、ThrowsAdvice**
 　　类全名：org.springframework.aop.ThrowsAdvice
 　　在方法抛出异常后，做处理。
 　　当该通知处理完异常后，会简单地将异常再次抛出给目标调用方法。
 　　
 **2. 配置过程**
 　　(1) 配置目标对象
 　　(2) 配置通知
 　　(3) 利用ProxyFactoryBean将通知织入到目标对象，形成一个动态代理对象
 　　(4) 客户端使用动态代理来访问目标对象的方法。
 　　
 　　在默认情况下，通知会应用到所有的方法之上。
 　　Pointcut：根据方法和类决定在什么地方织入通知
 　　Advisor：将Pointcut与Advice结合到一起
 　　
 　　**自定义切入点：**
 　　步骤：
 　　1、实现org.springframework.aop.ClassFileter
 　　2、实现org.springframework.aop.MethodMatcher
 　　3、实现org.springframework.aop.Pointcut
 　　4、实现org.springframework.aop.PointcutAdvisor
 　　注意：
 　　在此可定义
 　　private advice advice;
 　　private Pointcut pointcut;
 　　在配置文件中，将自定义的切入点与通知绑定到一起
 　　5、利用ProxyFactoryBean将advisor织入到目标对象
 　　ProxyFactoryBean的作用：
 　　依照配置信息，将切面应用到目标对象，生成动态代理对象。
 　　(1) Spring只支持方法连接点，不支持属性连接点。（原因是Spring AOP采用的是动态代理模式，而动态代理本身就是在方法调用前加入代码）
 　　(2) Spring只在运行时，将切面织入到目标对象。（有些AOP实现，可以在编译时织入切面到目标对象）
```
<bean id=" registerService" class="org.springframework.aop.framework.ProxyFactoryBean" >
　　　　<property name=" proxyInterfaces" > <!-- 目标对象实现的接口 -->
　　　　<!-- 如果没有定义接口，则所有方法使用CGLib -->
　　　　　　<value>aop.RegisterService</value>
　　　　</property>
　　　　<property name=" target"> <!-- 目标对象 -->
　　　　　　<ref local=" registerServiceTarget" />
　　　　</property>
　　　　<property name=" interceptorNames" > <!-- 需要织入到目标对象的切面 -->
　　　　　　<list>
　　　　　　　　<value>logAdvice</value>
　　　　　　　　<value>throwsAdvice</value>
　　　　　　</list>
　　　　</property>
　　</bean>
```

### 5、切入点（Pointcut） ###
**1、Pointcut<interface>**
　　切入点是指通知/切面可以应用到哪些类，哪些方法之上。
　　Pointcut API
　　Pointcut：org.springframework.aop.Pointcut，对某些类某些方法应用切面。
　　ClassFilter：org.springframework.aop.ClassFilter，用来过滤类（哪些类可以应用切面）
　　MethodMatcher：org.springframework.aop.MethodMatcher，用来过滤方法（类中哪些方法应用切面）
　　Advisor：org.springframework.aop.PointcutAdvisor，将Pointcut和Advice结合到一起
　　配置文件样例：
```
<beans>
　　<bean id="registerServiceTarget" class="aop5.RegisterService"/>
　　<bean id="logAdvice" class="aop5.LogAdvice"/>
　　<bean id="myPointcut" class="aop5.MyPointcut"/>
　　<bean id="myPointcutAdvisor" class="aop5.MyPointcutAdvisor">
　　　　<property name="advice">
　　　　　　<ref local="logAdvice"/>
　　　　</property>
　　　　<property name="pointcut">
　　　　　　<ref local="myPointcut"/>
　　　　</property>
　　</bean>
　　<bean id="registerService" class="org.springframework.aop.framework.ProxyFactoryBean">
　　　　<property name="target">
　　　　　　<ref local="registerServiceTarget"/>
　　　　</property>
　　　　<property name="interceptorNames">
　　　　　　<list>
　　　　　　　　<value>myPointcutAdvisor</value>
　　　　　　</list>
　　　　</property> 
　　</bean>
</beans>
```

**2、预定义切入点**

**2.1、静态切入点：**

　　**a、NameMatchMethodPointAdvisor**
　　org.springframework.aop.support.NameMatchMethodPointcutAdvisor
　　根据方法名称的特点进行匹配
　　核心xml：mappedName --> advice(ref)
　　配置文件样例：
```
<beans>
　　<bean id="registerServiceTarget" class="aop6.RegisterService" />
　　<bean id="logAdvice" class="aop6.LogAdvice" />
　　<bean id="namedPointcutAdvisor" class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
　　　　<property name="mappedName">
　　　　　　<value>methodOne</value>
　　　　</property>
　　　　<property name="advice">
　　　　　　<ref local="logAdvice"/>
　　　　</property>
　　</bean>
　　<bean id="registerService" class="org.springframework.aop.framework.ProxyFactoryBean">
　　　　<property name="target">
　　　　　　<ref local="registerServiceTarget" />
　　　　</property>
　　　　<property name="interceptorNames">
　　　　　　<list>
　　　　　　　　<value>namedPointcutAdvisor</value>
　　　　　　</list>
　　　　</property>
　　</bean>
</beans>
```
　　**b、RegexpMethodPointcutAdvisor**
　　org.springframework.aop.support.RegexpMethodPointcutAdvisor
　　根据正则表达式匹配方法名
　　核心xml：pattern --> advice(ref)
　　配置文件样例：
```
<beans >
　　<bean id="registerServiceTarget" class="aop6.RegisterService" />
　　<bean id="logAdvice" class="aop6.LogAdvice" />
　　<bean id="regexpAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
　　　　<property name="pattern">
　　　　　　<value>.*method.*</value>
　　　　</property>
　　　　<property name="advice">
　　　　　　<ref local="logAdvice"/>
　　　　</property>
　　</bean>

　　<bean id="registerService" class="org.springframework.aop.framework.ProxyFactoryBean">
　　　　<property name="target">
　　　　　　<ref local="registerServiceTarget" />
　　　　</property>
　　　　<property name="interceptorNames">
　　　　　　<list>
　　　　　　　　<value>regexpAdvisor</value>
　　　　　　</list>
　　　　</property>
　　</bean>
</beans>
```
**2.2、动态切入点：**　
　　org.springframework.aop.support.ControlFlowPointcut
　　
**2.3、切入点的交叉和合并：**
　　Pointcuts.union
　　配置文件样例：
```
<bean id="unionPointcut" class="aop7.UnionPointcut">
　　<property name="pointcuts">
　　　　<list>
　　　　　　<ref local="myPointcut"/>
　　　　　　<ref local="otherPointcut"/>
　　　　</list>
　　</property>
</bean>

<bean id="myPointcutAdvisor" class="aop7.MyPointcutAdvisor">
　　<property name="advice">
　　　　<ref local="logAdvice"/>
　　</property>
　　<property name="pointcut">
　　　　<ref local="unionPointcut"/>
　　</property>
</bean>
```
**2.4、Introduction**
　　一种特殊类型的Advice，为类动态增加方法和属性。
　　编程步骤：
　　(1) 实现org.springframework.aop.IntroductionInterceptor或继承org.springframework.aop.support.DelegatingIntroductionInterceptor
　　(2) 使用org.springframework.aop.support.DefaultIntroductionAdvisor
　　配置文件样例：
```
<bean id="myIntroInterceptor" class="aop8.MyIntroductionInterceptor"/>
<bean id="myIntroInterceptorAdvisor" class="org.springframework.aop.support.DefaultIntroductionAdvisor">
　　<constructor-arg>
　　　　<ref local="myIntroInterceptor"/>
　　</constructor-arg>
　　<constructor-arg>
　　　　<value>aop8.OtherBean</value>
　　</constructor-arg>
</bean>
```

### 6、自动代理 ###
　　Spring在生成代理对象的时候，默认情况下，会使用被代理对象的接口来生成代理对象。
　　如果被代理对象没有实现接口，此时，Spring会使用GCLib生成代理对象，此时该代理对象是被代理对象的子类。
**1、BeanNameAutoProxyCreator**
　　org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator
　　根据类的名称来为符合相应名称的类生成相应代理对象。
　　beanName（list），interceptorNames
　　配置文件样例：
```
<bean id="namedProxyCreator" class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
　　<property name="beanNames">
　　　　<list>
　　　　　　<value>someService</value>
　　　　　　<value>otherService</value>
　　　　</list>
　　</property>
　　<property name="interceptorNames">
　　　　<list>
　　　　　　<value>logAdvice</value>
　　　　</list>
　　</property>
</bean>
```
2、DefaultAdvisorAutoProxyCreator
　　org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator
　　自动将Advisor与匹配的Bean进行绑定
　　只能与Advisor配合使用
　　配置文件样例：
```
<bean id="autoProxyCreator" class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
```


----------
## Spring对持久层的支持 ##
　　Spring对持久层的支持：1、JDBC，2、O/R Mapping（Hibernate，TopLink等）
### 1、Spring对持久层支持采用的策略： ###
　　1、Spring对持久层“不发明重复的轮子”，即没有重新实现新的持久层方案，对现有持久层方案做封装，更利于使用。
　　2、采用DAO模式
　　3、提供了大量的模板类来简化编程（HibernateDaoSupport，JdbcTemplate等）
　　4、重新设计了一套完善的异常体系结构
　　  a、类型丰富，细化异常类型
　　  b、全都是运行时异常（RuntimeException）
### 2、Spring对JDBC的支持 ###
　　**1、配置数据源**
　　方式一：采用Spring内置的数据源，Spring内置实现DriverManagerDataSource
```
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
　　<property name="driverClassName">
　　　　<value>com.mysql.jdbc.Driver</value>
　　</property>
　　<property name="url">
　　　　<value>jdbc:mysql://localhost:3306/hibdb</value>
　　</property>
　　<property name="username">
　　　　<value>root</value>
　　</property>
　　<property name="password">
　　　　<value>windows</value>
　　</property>
</bean>
```
　　方案二：采用开源数据库产品如DBCP
　　DBCP提供的BasicDataSource

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
　　<property name="driverClassName">
　　　　<value>com.mysql.jdbc.Driver</value>
　　</property>
　　<property name="url">
　　　　<value>jdbc:mysql://localhost:3306/hibdb</value>
　　</property>
　　<property name="username">
　　　　<value>root</value>
　　</property>
　　<property name="password">
　　　　<value>windows</value>
　　</property>
</bean>
```
　　方案三：直接使用容器提供的数据源（如Tomcat，Weblogic，Sun Application Server）
　　JNDI数据源：（mysql5，tomcat5.5）
　　**Step1：**
　　在server.xml中：
```
<Resource name="jdbc/mydatasource" auth="Container" description="DB Connection" type="javax.sql.DataSource" username="root" password="windows" driverClassName="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/tarena" maxActive="5" />
```
　　**Step2：**
　　在context.xml中(conf\context.xml)：
```
<ResourceLink name="jdbc/mydatasource" global="jdbc/mydatasource" type="javax.sql.DataSourcer"/>
```
　　**Step3：**
　　在beans-config.xml中：
```
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
　　<property name="jndiName">
　　　　<value>java:comp/env/jdbc/mydatasource</value>
　　</property>
</bean>
```
　　**2、配置JdbcTemplate模板类（封装了绝大多数数据库操作）**
　　**3、配置DAO**
　　**4、配置Service**
　　
　　编程步骤：
　　step1: 配置数据源
　　step2：配置JdbcTemplate
```
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
　　<property name="dataSource">
　　　　<ref bean="dataSource" />
　　</property>
</bean>    
```
　　step3：配置DAO
```
<bean id="orderDao" class="lab5.OrderDAOImpl">
　　<property name="jt">
　　　　<ref bean="jdbcTemplate"/>
　　</property>
</bean>
```
　　注意：查询时，使用RowMapper。
### 3、Spring对Hibernate的支持 ###
　　Step1：配置数据源
　　Step2：配置SessionFactory
```
<bean id="mySessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
　　<property name="dataSource">
　　　　<ref bean="dataSource" />
　　</property>
　　<property name="mappingResources">
　　　　<list>
　　　　　　<value>lab6/Order.hbm.xml</value>
　　　　</list>
　　</property>
　　<property name="hibernateProperties">
　　　　<props>
　　　　　　<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
　　　　　　<prop key="hibernate.show_sql">true</prop>
　　　　</props>
　　</property>
</bean>
```
　　Step3：配置DAO
```
<bean id="orderDao" class="lab6.OrderDAOHibernateImpl">
　　<property name="sessionFactory">
　　　　<ref bean="mySessionFactory" />
　　</property>
</bean>
```
　　注意：以上配置是要求dao继承HibernateDaoSupport。
　　

----------
## Spring对事务的支持 ##
### 1、AOP事务的含义 ###
　　事务当作一个切面，动态织入到目标对象，形成一个代理对象。
### 2、Spring的事务机制 ###
　　Spring支持声明式事务。
　　Spring使用事务服务代理和事务管理器（如HibernateTransactionManager）来支持事务服务。
　　Spring对事务的边界多了一种嵌套事务（PROPAGATION_NESTED）。
　　**PROPAGATION_NESTED：**
　　如果客户端启动了事务T1，那么Bean启动一个前台在T1中的子事务T2；
　　如果客户端没有启动事务，那么Bean会启动一个新的事务，类似于REQUIRED_NEW。
### 3、Spring中事务的使用 ###

 1. Spring中使用Hibernate事务
 　　Step1：配置数据源
 　　Step2：配置SessionFactory
 　　Step3：配置事务管理器
 ```
 <bean id="myTransactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
 　　<property name="sessionFactory">
 　　 　　<ref bean="mySessionFactory" />
 　　</property>
</bean> 
 ```
 　　Step4：创建事务服务代理
 ```
<bean id="saleService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
 　　<property name="proxyInterfaces">
 　　 　　<value>lab7.SaleService</value>
 　　</property>
 　　<property name="transactionManager">
 　　 　　<ref bean="myTransactionManager" />
 　　</property>
 　　<property name="target">
 　　 　　<ref bean="saleServiceTarget" />
 　　</property>
 　　<property name="transactionAttributes">
 　　 　　<props>
 　　 　　 　　<prop key="*">PROPAGATION_REQUIRED</prop>
 　　 　　</props>
 　　</property>
</bean>
 ```
 
 　　事务属性描述格式：
 　　传播行为，隔离级别，只读事务（read-only），回滚规则
 　　
 　　在默认情况下，Spring的容器对于非受查异常（服务模块中抛出的非受查异常），会回滚事务。对于受查异常，会提交事务。
 　　如果即使发生了某种受查异常，也要回滚事务，可以用“ - 异常类型 ”来声明。同样，对于非受查异常，如果不要求回滚事务，可以用“ + 异常类型 ”来声明。
 　　
 　　如何简化事务配置？
 　　使用继承（抽象的Service类）、自动代理。
### 4、Spring事务与EJB事务 ###
 　　**1、EJB事务**
 　　EJB的CMT管理事务方式，只能设置事务边界（传播行为），对于隔离性是不能设置的，并且EJB不支持嵌套事务。
 　　**2、Spring事务**
 　　对于Spring来说，Spring的声明式事务可以设置事务边界（传播行为），设置隔离级别，设置只读事务，回滚规则（+：对于任何异常都提交，- ：对于任何异常都回滚）
```
<property name=”transactionAttributes”>
 　　<props>
 　　 　　<prop key=”*”>+异常类型1，-异常类型2</prop>
 　　<props>
</property>
```
 　　**PS:**Spring对嵌套事务的支持依赖于数据库底层对嵌套式事务的支持。


----------
## SSH整合 ###
### 1、SSH ###
 　　Struts（表示层） + Spring（业务层） + Hibernate（持久层）
 　　**Struts：**
 　　Struts是一个表示层框架，主要作用是界面展示，接收请求，分发请求。
 　　在MVC框架中，Struts属于VC层次，负责界面表现，负责MVC关系的分发。（View：沿用JSP，HTTP，Form，Tag，Resourse；Controller：ActionServlet，struts-config.xml，Action）
 　　**Hibernate：**
 　　Hibernate是一个持久层框架，它只负责与关系数据库的操作。
 　　**Spring：**
 　　Spring是一个业务层框架，是一个整合的框架，能够很好地黏合表示层与持久层。
 　　(1) Web分层架构中业务层为什么都选择Spring？
 　　Service层需要处理业务逻辑和交叉业务逻辑，处理事务，日志，安全等，而这些与Spring的IoC，AOP等不谋而合。
 　　(2) Web分层架构中，对于各层技术的采用应该遵循一个怎样的标准？
 　　a、选择发展成熟的技术
 　　+ 经过了项目实践证实可行性良好
 　　+ 文档完善
 　　+ 技术一直处于可持续发展更新
 　　b、Web应用中对于技术的选择有赖于开发人员的技术掌握情况
### 2、Spring与Struts整合 ###
 　　前提：
 　　必须在Web应用启动时，创建Spring的ApplicationContext实例
 　　方式：
 　　1、采用ContextLoaderListener来创建ApplicationContext：
```
<context-param>
 　　<param-name>contextConfigLocation</param-name>
 　　<param-value>
 　　 　　/WEB-INF/spring-config/applicationContext.xml
 　　</param-value>
</context-param>

<listener>
 　　<listener-class>
 　　 　　org.springframework.web.context.ContextLoaderListener
 　　</listener-class>
</listener>
```
 　　2、采用ContextLoaderPlugin来创建ApplicationContext
```
<plug-in className="org.springframework.web.struts.ContextLoaderPlugIn">
 　　<set-property property="contextConfigLocation" value="/WEB-INF/config/sale.xml" />
</plug-in>
```
 　　或者：
 　　通过listener装载Spring应用上下文
 　　
 　　方式一：通过Spring的ActionSupport类
 　　ActionSupport类：知道ApplicationContext的获得方式。
 　　步骤：
 　　(1) Action直接继承ActionSupport
 　　(2) 使用ApplicationContext ctx = getWebApplicationContext()；取得Spring上下文
 　　(3) 取得相应Bean
 　　注意：有可能需要替换commons-attributes-compiler.jar包。
 　　优点：见到那
 　　缺点：耦合高，违反IoC，无法使用多方法的Action
 　　
 　　方式二：通过Spring的DelegatingActionProxy类
 　　步骤：
 　　1、Action中，使用IoC获得服务
 　　2、配置struts-config.xml
```
<action path="/somepath" type="org.springframework.web.struts.DelegatingActionProxy"/>
```
 　　3、在Spring配置文件中
```
<bean name="/somepath" class="SomeAction">
 　　<property name="service"><ref bean=""/>
</bean>
```
 　　注意，要用bean name命名。
 　　/somepath: Action的path
 　　优点：不使用Spring API编写Action，利用了IoC装配，可以利用容器的scope="prototype"来保证每一个请求有一个单独的Action来处理，避免struts中Action的线程安全问题。
 　　缺点：struts配置文件中，所有path都映射到同一个代理类
 　　
 　　方式三：通过Spring的DelegatingRequestProcessor类
 　　步骤：
 　　1、Action中，使用IoC获得服务
 　　2、配置struts-config.xml
```
<controller processorClass="org.springframework.web.struts.DelegatingRequestProcessor" />
```
 　　3、在Spring配置文件中
```
<bean name="/somepath" class="SomeAction">
 　　<property name="service"><ref bean=""/>
</bean>
```
 　　**小结：**
 　　Spring和Struts整合方式只有两种：
 　　(1) 由Spring容器来管理Action（方式二，方式三）
 　　(2) Action处于容器之外（方式一）
 　　
 　　注意：
 　　中文问题：设置过滤器，设置页面编码，数据库编码
### 3、关于Spring和EJB ###
**1、Spring与EJB3.0的对比**
 　　Spring与EJB3.0之间的关系是竞争关系。
 　　(1) Spring是一个开源的框架，而EJB3.0是一个标准（标准意味着将得到广泛的支持以及良好的兼容性），并且，采用EJB3.0，项目的后期维护得到了保证。
 　　(2) Spring是一个轻量级框架，EJB3.0是一个重量级框架（完整的容器，包含所有的服务）。
 　　(3) Spring的IoC，AOP集成了大量的开源框架，可扩展性良好。EJB3.0的可扩展性则完全依赖于新的容器。
 　　(4) Spring对事物支持不如EJB3.0成熟，Spring对集群的兼容也不够。
 　　(5) Spring和EJB3.0都是一个企业级开发框架，都支持声明式事务。
 　　
**2、Spring的优势与劣势**
 　　**优势：**
 　　(1) 简化了企业级开发（对企业级服务进行了进一步的封装）
 　　(2) 采用Spring框架的程序意味着良好的分层架构设计，并保证是面向接口编程的
 　　(3) 用IoC，AOP容器，模块是可配置的，松耦合的，方便了后期维护
 　　
 　　**劣势：**
 　　(1) 配置负责，不方便维护
 　　(2) 容器大量使用反射等机制装配对象，影响性能，对于高并发的大型应用无能为力。


![](http://img181.poco.cn/mypoco/myphoto/20110121/13/55913548201101211309453196937047417_001.jpg)