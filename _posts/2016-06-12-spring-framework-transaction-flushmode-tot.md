---
layout: post
title:  Spring事务FlushMode属性
date:   2016-06-12 00:00:00 +0800
categories: Spring
tag: [Spring, Hibernate, 事务管理]
---

* content
{:toc}

### 1、hibernate的api ###
- <http://tool.oschina.net/apidocs/apidoc?api=hibernate-3.6.10/>   
- <http://tool.oschina.net/apidocs/apidoc?api=hibernate-4.1.4/>  
--> 3.6的 FlushMode 属性是一个Class类，而4.1更换成了Enum。

### 2、FlushMode属性(策略) ###

&emsp;&emsp;FlushMode 代表一个flushing（刷新）的策略，它将通过改变session的状态和执行SQL的状态来处理线程中的数据。
+ **NEVER**： 已经废弃，被 MANUAL 取代
+ **MANUAL**：Session永远只会在应用程序调用`Session.flush()`方法时才会刷新
+ **AUTO**：  在确保查询从不会返回脏数据的情况下，在查询前刷新 Session
+ **COMMIT**：Session在提交事务时刷新
+ **ALWAYS**：在查询前刷新 Session

说明:   
&emsp;&emsp;1、Session：Session接口是Hibernate向程序提供操作数据库的最主要接口，是单线程对象，它提供了基本的保存、更新、删除和查询方法。它有一个缓存，保存了持久化对象，当清理缓存时，按照这些持久化对象同步更新数据库。根据Session的定义，可以知道刷新Session也就刷新了数据库。  
&emsp;&emsp;2、调用`Session.flush()`方法，不管FlushMode被设置成任何策略，都将会刷新Session，使其中的缓存数据同步到数据库。  
&emsp;&emsp;3、ALWAYS和AUTO的区别：  
&emsp;&emsp;从API看出，AUTO不会像ALWAYS那样总是刷新Session，那它在何时才会刷新呢？当hibernate缓存中的对象被改动之后，会被标记为脏数据（即与数据库不同步了）。当session设置为FlushMode.AUTO时，hibernate在进行查询的时候会判断缓存中的数据是否为脏数据，是则刷数据库，不是则不刷，而always是直接刷新，不进行任何判断。很显然auto比always要高效得多。   
&emsp;&emsp;4、设置FlushMode.MANUAL，在操作过程中hibernate会将事务设置为readonly，所以在增加/删除/修改操作过程中会出现如下错误：
`org.springframework.dao.InvalidDataAccessApiUsageException: Write operations are not allowed in read-only mode (FlushMode.NEVER/MANUAL): Turn your Session into FlushMode.COMMIT/AUTO or remove 'readOnly' marker from transaction definition.`

Session的刷新策略
<table>
	<tr>
		<td></td>
		<td>调用Session的查询方法时</td>
		<td>调用Session.commit()时</td>
		<td>调用Session.flush()时</td>
	</tr>
	<tr>
		<td>ALWAYS</td>
		<td>刷新</td>
		<td>刷新</td>
		<td>刷新</td>
	</tr>
	<tr>
		<td>AUTO</td>
		<td>当缓存被标记为脏数据时，刷新</td>
		<td>刷新</td>
		<td>刷新</td>
	</tr>
	<tr>
		<td>COMMIT</td>
		<td>不刷新</td>
		<td>刷新</td>
		<td>刷新</td>
	</tr>
	<tr>
		<td>MANUAL</td>
		<td>不刷新</td>
		<td>不刷新</td>
		<td>刷新</td>
	</tr>
</table>

--> FlushMode的四个属性实际上代表了 hibernate 处理 Session 中的持久化对象的缓存的四种策略，这四种策略最大的不同就是刷新 Session，将其中的缓存的持久化对象同步更新数据库的时间点不同。

### 3、OpenSessionInViewFilter的FlushMode属性设置 ###
&emsp;&emsp;假设在OpenSessionInViewFilter设置FlushMode.MANUAL。若OpenSessionInViewFilter在`getSession`的时候,会把获取回来的session的flush mode设为`FlushMode.MANUAL`。然后把该sessionFactory绑定到`TransactionSynchronizationManager`，使request的整个过程都使用同一个session，在请求过后再解除该sessionFactory的绑定，最后`closeSessionIfNecessary`根据该session是否已和transaction绑定来决定是否关闭session。在这个过程中，若HibernateTemplate发现自当前session有不是readOnly的transaction，就会获取到FlushMode.AUTO Session，使方法拥有写权限。也即是，如果有不是readOnly的transaction就可以由Flush.NEVER转为Flush.AUTO,拥有insert,update,delete操作权限，如果没有transaction，并且没有另外人为地设flush model的话，则doFilter的整个过程都是Flush.MANUAL。所以受transaction保护的方法有写权限，没受保护的则没有。

### 4、解决因FlushMode设置出现不能进行增、删、改的异常 ###
1、配置事务，spring会读取事务中的各种配置来覆盖hibernate的session中的FlushMode。
```
<!-- 配置Spring的事务处理 -->
<!-- 创建事务管理器-->
<bean id="txManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>

<!-- 配置事务管理器应用的范围 -->
<!-- 配置AOP，Spring是通过AOP来进行事务管理的 -->
<aop:config>
    <!-- 设置pointCut表示哪些方法要加入事务处理 -->
　　<!-- 以下的事务是声明在DAO中，但是通常都会在Service来处理多个业务对象逻辑的关系，注入删除，更新等，此时如果在执行了一个步骤之后抛出异常
　　就会导致数据不完整，所以事务不应该在DAO层处理，而应该在service，这也就是Spring所提供的一个非常方便的工具，声明式事务 -->
    <aop:pointcut id="managerOperation" expression="execution(* com.test.service.*.*(..))"/>
    <!-- 通过advisor来确定具体要加入事务控制的方法 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="managerOperation" />
</aop:config>

<!-- 配置Advice(事务的传播特性) 即：配置哪些方法要加入事务控制 -->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <!-- 让所有的方法都加入事务管理，为了提高效率，可以把一些查询之类的方法设置为只读的事务 -->
        <tx:method name="*"  propagation="REQUIRED" read-only="true" />
	<!-- 以下方法都是可能设计修改的方法，就无法设置为只读 -->
        <tx:method name="save*" propagation="REQUIRED" />
        <tx:method name="add*"  propagation="REQUIRED" />
        <tx:method name="del*"  propagation="REQUIRED" />
        <tx:method name="update*" propagation="REQUIRED" isolation="REPEATABLE_READ"/>
    </tx:attributes>
</tx:advice>  
```

2、直接修改OpenSessionInViewFilter过滤器的配置，配置过滤器的时候设置openSession org.springframework.orm.hibernate3.support.OpenSessionInViewFilter中flushMode为AUTO/COMMIT 。  
注意：如果这样设置，则要么结合上面1，要么结合下面3，因为不管怎样，对于Hibernate来说总是要配置事务，否则无法操作持久化对象同步到数据库。
```
<filter>
    <filter-name>openSessionInViewFilter</filter-name>
    <filter-class>org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class>
    <init-param>
        <param-name>singleSession</param-name>
        <param-value>true</param-value><!-- 默认true -->
    </init-param>
    <init-param>
        <param-name>flushMode</param-name>
        <param-value>AUTO</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>openSessionInViewFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

3、先编程式修改FlushMode，比如`session.setFlushMode(FlushMode.AUTO)`
```
@Override
public void saveOrUpdate(T t) {
    Session session = null;
    Transaction tr = null;
    try {
        session = HibernateSessionFactory.getSession();
        tr = session.beginTransaction();
        session.saveOrUpdate(t);
        tr.commit();
        HibernateSessionFactory.closeSesssion();
    } catch (Exception e) {
        e.printStackTrace();
        tr.rollback();
    }
}
```

### 5、OpenSessionInView分析  ###
&emsp;&emsp;对于配置了Filter的处理。每次都会拦截到一个请求，在后台处理之前就会开启一个事物并打开一个数据库连接（线程安全的），后台处理完毕后会关闭当前的连接。  
&emsp;&emsp;原理图如下：
&emsp;&emsp;![OpenSessionInView原理图](http://or9g8eqm7.bkt.clouddn.com/17-7-12/51189021.jpg)

&emsp;&emsp;OpenSessionInView 性能问题:  
&emsp;&emsp;由于使用了OpenSessionInView模式，Session的生命周期变得非常长。虽然解决了Lazy Load的问题，但是带来的问题就是Hibernate的一级缓存，也就是Session级别的缓存的生命周期会变得非常长，那么如果你在你的Service层做大批量的数据操作时，其实这些数据会在缓存中保留一份，这是非常耗费内存的。还有一个数据库连接的问题，存在的原因在于由于数据库的Connection是和Session绑在一起的，所以，Connection也会得不到及时的释放。因而当系统出现业务非常繁忙，而计算量又非常大的时候，往往数据连接池的连接数会不够。
