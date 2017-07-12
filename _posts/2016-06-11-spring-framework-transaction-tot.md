---
layout: post
title:  Spring事务管理
date:   2016-06-11 00:00:00 +0800
categories: Spring
tag: [Spring, 事务管理]
---

* content
{:toc}

### 1、Spring事务属性分析 ###
#### 事务隔离级别 ####
&emsp;&emsp;隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefnition接口中定义了五个表示隔离级别的产量：
+ TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
+ TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
+ TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
+ TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
+ TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

#### 事务传播行为 ####
&emsp;&emsp;事务传播行为是指如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几下表示传播行为的常量：
+ TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
+ TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
+ TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
+ TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
+ TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
+ TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
+ TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

&emsp;&emsp;前面的六种事务传播行为是Spring从EJB中引入的，它们共享相同的概念。而PROPAGATION_NESTED是 Spring 所特有的。以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

#### 事务超时 ####
&emsp;&emsp;所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

#### 事务的只读属性 ####
&emsp;&emsp;事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。在 TransactionDefinition 中以 boolean 类型来表示该事务是否只读。

#### 事务的回滚规则 ####
&emsp;&emsp;通常情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常），则默认将回滚事务。如果没有抛出任何异常，或者抛出了已检查异常，则仍然提交事务。这通常也是大多数开发者希望的处理方式，也是 EJB 中的默认处理方式。但是，我们可以根据需要人为控制事务在抛出某些未检查异常时任然提交事务，或者在抛出某些已检查异常时回滚事务。

### 2、Spring事务管理API分析 ###
&emsp;&emsp;Spring 框架中，涉及到事务管理的 API 大约有100个左右，其中最重要的有三个：TransactionDefinition、PlatformTransactionManager、TransactionStatus。

&emsp;&emsp;所谓事务管理，其实就是“按照给定的事务规则来执行提交或者回滚操作”。“给定的事务规则”就是用 TransactionDefinition 表示的，“按照……来执行提交或者回滚操作”便是用 PlatformTransactionManager 来表示，而 TransactionStatus 用于表示一个运行着的事务的状态。打一个不恰当的比喻，TransactionDefinition 与 TransactionStatus 的关系就像程序和进程的关系。

#### TransactionDefinition ####
&emsp;&emsp;它用于定义一个事务。它包含了事务的静态属性，比如：事务传播行为、超时时间等等。Spring 为我们提供了一个默认的实现类：DefaultTransactionDefinition，该类适用于大多数情况。如果该类不能满足需求，可以通过实现 TransactionDefinition 接口来实现自己的事务定义。

#### PlatformTransactionManager ####
&emsp;&emsp;PlatformTransactionManager 用于执行具体的事务操作。接口定义如下所示：
```
Public interface PlatformTransactionManager{
  TransactionStatus getTransaction(TransactionDefinition definition)
   throws TransactionException;
   void commit(TransactionStatus status)throws TransactionException;
   void rollback(TransactionStatus status)throws TransactionException;
}
```
&emsp;&emsp;根据底层所使用的不同的持久化 API 或框架，PlatformTransactionManager 的主要实现类大致如下：
+ DataSourceTransactionManager：适用于使用JDBC和iBatis进行数据持久化操作的情况。
+ HibernateTransactionManager：适用于使用Hibernate进行数据持久化操作的情况。
+ JpaTransactionManager：适用于使用JPA进行数据持久化操作的情况。
+ 另外还有JtaTransactionManager 、JdoTransactionManager、JmsTransactionManager等等。

#### TransactionStatus #### 
&emsp;&emsp;PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。TransactionStatus 接口提供了一个简单的控制事务执行和查询事务状态的方法。
```
public  interface TransactionStatus{
   boolean isNewTransaction();
   void setRollbackOnly();
   boolean isRollbackOnly();
}
```

### 3、编程式事务管理 ###
&emsp;&emsp;在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法。通过 Spring 提供的事务管理 API，我们可以在代码中灵活控制事务的执行。在底层，Spring 仍然将事务操作委托给底层的持久化框架来执行。

#### 基于底层API的编程式事务管理 ####
&emsp;&emsp;根据PlatformTransactionManager、TransactionDefinition 和 TransactionStatus 三个核心接口，可以通过编程的方式来进行事务管理。
```
public class BankServiceImpl implements BankService {
    private BankDao bankDao;
    private TransactionDefinition txDefinition;
    private PlatformTransactionManager txManager;
    ......

    public boolean transfer(Long fromId， Long toId， double amount) {
        TransactionStatus txStatus = txManager.getTransaction(txDefinition);
        boolean result = false;
        try {
            result = bankDao.transfer(fromId， toId， amount);
            txManager.commit(txStatus);
        } catch (Exception e) {
            result = false;
            txManager.rollback(txStatus);
            System.out.println("Transfer Error!");
        }
        return result;
    }
}
```
配置：
```
<bean id="bankService" class="footmark.spring.core.tx.programmatic.origin.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
        <property name="txManager" ref="transactionManager"/>
        <property name="txDefinition">
        <bean class="org.springframework.transaction.support.DefaultTransactionDefinition">
            <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
        </bean>
    </property>
</bean>
```

#### 基于 TransactionTemplate 的编程式事务管理 ####
```
public class BankServiceImpl implements BankService {
    private BankDao bankDao;
    private TransactionTemplate transactionTemplate;
    ......

    public boolean transfer(final Long fromId， final Long toId， final double amount) {
        return (Boolean) transactionTemplate.execute(new TransactionCallback(){
            public Object doInTransaction(TransactionStatus status) {
                Object result;
                try {
                    result = bankDao.transfer(fromId， toId， amount);
                } catch (Exception e) {
                    status.setRollbackOnly();
                    result = false;
                    System.out.println("Transfer Error!");
                }
                return result;
            }
        });
    }
}
```
配置：
```
<bean id="bankService" class="footmark.spring.core.tx.programmatic.template.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="transactionTemplate" ref="transactionTemplate"/>
</bean>
```
&emsp;&emsp;TransactionTemplate 的 execute() 方法有一个 TransactionCallback 类型的参数，该接口中定义了一个 doInTransaction() 方法，通常我们以匿名内部类的方式实现 TransactionCallback 接口，并在其 doInTransaction() 方法中书写业务逻辑代码。这里可以使用默认的事务提交和回滚规则，这样在业务代码中就不需要显式调用任何事务管理的 API。doInTransaction() 方法有一个TransactionStatus 类型的参数，我们可以在方法的任何位置调用该参数的 setRollbackOnly() 方法将事务标识为回滚的，以执行事务回滚。

&emsp;&emsp;根据默认规则，如果在执行回调方法的过程中抛出了未检查异常，或者显式调用了TransacationStatus.setRollbackOnly() 方法，则回滚事务；如果事务执行完成或者抛出了 checked 类型的异常，则提交事务。

&emsp;&emsp;TransactionCallback 接口有一个子接口 TransactionCallbackWithoutResult，该接口中定义了一个 doInTransactionWithoutResult() 方法，TransactionCallbackWithoutResult 接口主要用于事务过程中不需要返回值的情况。当然，对于不需要返回值的情况，我们仍然可以使用 TransactionCallback 接口，并在方法中返回任意值即可。

### 4、声明式事务管理 ###
&emsp;&emsp;<strong>Spring 的声明式事务管理在底层是建立在 AOP 的基础之上的。</strong>其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

&emsp;&emsp;声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明（或通过等价的基于标注的方式），便可以将事务规则应用到业务逻辑中。因为事务管理本身就是一个典型的横切逻辑，正是 AOP 的用武之地。Spring 开发团队也意识到了这一点，为声明式事务提供了简单而强大的支持。

#### 基于 TransactionInterceptor  的声明式事务管理 ####
&emsp;&emsp;首先，我们配置了一个 TransactionInterceptor 来定义相关的事务规则，他有两个主要的属性：一个是 transactionManager，用来指定一个事务管理器，并将具体事务相关的操作委托给它；另一个是 Properties 类型的 transactionAttributes 属性，它主要用来定义事务规则，该属性的每一个键值对中，键指定的是方法名，方法名可以使用通配符，而值就表示相应方法的所应用的事务属性。

&emsp;&emsp;指定事务属性的取值有较复杂的规则，具体的书写规则如下：
> 传播行为 [，隔离级别] [，只读属性] [，超时属性] [不影响提交的异常] [，导致回滚的异常]

+ 传播行为是唯一必须设置的属性，其他都可以忽略，Spring为我们提供了合理的默认值。
+ 传播行为的取值必须以“PROPAGATION_”开头，具体包括：PROPAGATION_MANDATORY、PROPAGATION_NESTED、PROPAGATION_NEVER、PROPAGATION_NOT_SUPPORTED、PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_SUPPORTS，共七种取值。
+ 隔离级别的取值必须以“ISOLATION_”开头，具体包括：ISOLATION_DEFAULT、ISOLATION_READ_COMMITTED、ISOLATION_READ_UNCOMMITTED、ISOLATION_REPEATABLE_READ、ISOLATION_SERIALIZABLE，共五种取值。
+ 如果事务是只读的，那么我们可以指定只读属性，使用“readOnly”指定。否则我们不需要设置该属性。
+ 超时属性的取值必须以“TIMEOUT_”开头，后面跟一个int类型的值，表示超时时间，单位是秒。
+ 不影响提交的异常是指，即使事务中抛出了这些类型的异常，事务任然正常提交。必须在每一个异常的名字前面加上“+”。异常的名字可以是类名的一部分。比如“+RuntimeException”、“+tion”等等。
+ 导致回滚的异常是指，当事务中抛出这些类型的异常时，事务将回滚。必须在每一个异常的名字前面加上“-”。异常的名字可以是类名的全部或者部分，比如“-RuntimeException”、“-tion”等等。

#### 基于 TransactionProxyFactoryBean 的声明式事务管理 ####
&emsp;&emsp;Spring 为我们提供了 TransactionProxyFactoryBean，用于将TransactionInterceptor 和 ProxyFactoryBean 的配置合二为一。
```
<beans......>
    ......
    <bean id="bankServiceTarget" class="footmark.spring.core.tx.declare.classic.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
    </bean>
    <bean id="bankService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <property name="target" ref="bankServiceTarget"/>
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>
    ......
</beans>

```

#### 基于 <tx> 命名空间的声明式事务管理 ####
```
<beans......>
    ......
    <bean id="bankService" class="footmark.spring.core.tx.declare.namespace.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
    </bean>

    <tx:advice id="bankAdvice" transaction-manager="transactionManager">

    <aop:config>
        <aop:pointcut id="bankPointcut" expression="execution(**.transfer(..))"/>
        <aop:advisor advice-ref="bankAdvice" pointcut-ref="bankPointcut"/>
    </aop:config>
    ......
</beans>
```

#### 基于 @Transactional 的声明式事务管理 ####
&emsp;&emsp;@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。
```
@Transactional(propagation = Propagation.REQUIRED)
    public boolean transfer(Long fromId， Long toId， double amount) {
    return bankDao.transfer(fromId， toId， amount);
}
```
&emsp;&emsp;Spring 使用 BeanPostProcessor 来处理 Bean 中的标注，因此我们需要在配置文件中作如下声明来激活该后处理 Bean
> <tx:annotation-driven transaction-manager="transactionManager"/>
PS: 虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 小组建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

### 5、Spring事务配置 ###

&emsp;&emsp;Spring配置文件中关于事物配置总是由三个组成部分，分别是DataSource、TransactionManager和代理机制这三个部分，无论哪种配置方式，一般变化的只是代理机制这部分。

&emsp;&emsp;DataSource、TransactionManager这两部分只是会根据数据访问方式有所变化，比如使用Hibernate进行数据访问时，DataSource实际为SessionFactory，TransactionManager的实现为HibernateTransactionManager。

&emsp;&emsp;具体如下图：

&emsp;&emsp;![](http://or9g8eqm7.bkt.clouddn.com/17-7-12/17461997.jpg)

根据代理机制的不同，总结了五种Spring事务的配置方式，配置文件如下：

#### 第一种方式：每个Bean都有一个代理 ####
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">

    <bean id="sessionFactory"  
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">  
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />  
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean>  

    <!-- 定义事务管理器（声明式的事务） -->  
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    
    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    
    <bean id="userDao"  
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">  
           <!-- 配置事务管理器 -->  
           <property name="transactionManager" ref="transactionManager" />     
        <property name="target" ref="userDaoTarget" />  
         <property name="proxyInterfaces" value="com.bluesky.spring.dao.GeneratorDao" />
        <!-- 配置事务属性 -->  
        <property name="transactionAttributes">  
            <props>  
                <prop key="*">PROPAGATION_REQUIRED</prop>
            </props>  
        </property>  
    </bean>  
</beans>
```

#### 第二种方式：所有Bean共享一个代理基类 ####
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">

    <bean id="sessionFactory"  
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">  
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />  
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean>  

    <!-- 定义事务管理器（声明式的事务） -->  
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    
    <bean id="transactionBase"  
            class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean"  
            lazy-init="true" abstract="true">  
        <!-- 配置事务管理器 -->  
        <property name="transactionManager" ref="transactionManager" />  
        <!-- 配置事务属性 -->  
        <property name="transactionAttributes">  
            <props>  
                <prop key="*">PROPAGATION_REQUIRED</prop>  
            </props>  
        </property>  
    </bean>    
   
    <!-- 配置DAO -->
    <bean id="userDaoTarget" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    
    <bean id="userDao" parent="transactionBase" >  
        <property name="target" ref="userDaoTarget" />   
    </bean>
</beans>
```

#### 第三种方式：使用拦截器 ####
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">

    <bean id="sessionFactory"  
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">  
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />  
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean>  

    <!-- 定义事务管理器（声明式的事务） -->  
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean> 
   
    <bean id="transactionInterceptor"  
        class="org.springframework.transaction.interceptor.TransactionInterceptor">  
        <property name="transactionManager" ref="transactionManager" />  
        <!-- 配置事务属性 -->  
        <property name="transactionAttributes">  
            <props>  
                <prop key="*">PROPAGATION_REQUIRED</prop>  
            </props>  
        </property>  
    </bean>
      
    <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">  
        <property name="beanNames">  
            <list>  
                <value>*Dao</value>
            </list>  
        </property>  
        <property name="interceptorNames">  
            <list>  
                <value>transactionInterceptor</value>  
            </list>  
        </property>  
    </bean>  
  
    <!-- 配置DAO -->
    <bean id="userDao" class="com.bluesky.spring.dao.UserDaoImpl">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
</beans>
```

#### 第四种方式：使用tx标签配置的拦截器 ####
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">

    <context:annotation-config />
    <context:component-scan base-package="com.bluesky" />

    <bean id="sessionFactory"  
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">  
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />  
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean>  

    <!-- 定义事务管理器（声明式的事务） -->  
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>
    
    <aop:config>
        <aop:pointcut id="interceptorPointCuts"
            expression="execution(* com.bluesky.spring.dao.*.*(..))" />
        <aop:advisor advice-ref="txAdvice"
            pointcut-ref="interceptorPointCuts" />        
    </aop:config>      
</beans>
```

#### 第五种方式：全注解 ####
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">

    <context:annotation-config />
    <context:component-scan base-package="com.bluesky" />

    <tx:annotation-driven transaction-manager="transactionManager"/>

    <bean id="sessionFactory"  
            class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">  
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />  
        <property name="configurationClass" value="org.hibernate.cfg.AnnotationConfiguration" />
    </bean>  

    <!-- 定义事务管理器（声明式的事务） -->  
    <bean id="transactionManager"
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    
</beans>
```

此时在Dao/Service/Action上需加上@Transactional注解，如下：
```
package com.bluesky.spring.dao;

import java.util.List;

import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.orm.hibernate3.support.HibernateDaoSupport;
import org.springframework.stereotype.Component;

import com.bluesky.spring.domain.User;

@Transactional
@Component("userDao")
public class UserDaoImpl extends HibernateDaoSupport implements UserDao {

    public List<User> listUsers() {
        return this.getSession().createQuery("from User").list();
    }
    
    
}
```

### 6、总结 ###
+ 基于 TransactionDefinition、PlatformTransactionManager、TransactionStatus 编程式事务管理是 Spring 提供的最原始的方式，通常我们不会这么写，但是了解这种方式对理解 Spring 事务管理的本质有很大作用。
+ 基于 TransactionTemplate 的编程式事务管理是对上一种方式的封装，使得编码更简单、清晰。
+ 基于 TransactionInterceptor 的声明式事务是 Spring 声明式事务的基础，通常也不建议使用这种方式，但是与前面一样，了解这种方式对理解 Spring 声明式事务有很大作用。
+ 基于 TransactionProxyFactoryBean 的声明式事务是上中方式的改进版本，简化的配置文件的书写，这是 Spring 早期推荐的声明式事务管理方式，但是在 Spring 2.0 中已经不推荐了。
+ 基于 <tx> 和 <aop> 命名空间的声明式事务管理是目前推荐的方式，其最大特点是与 Spring AOP 结合紧密，可以充分利用切点表达式的强大支持，使得管理事务更加灵活。
+ 基于 @Transactional 的方式将声明式事务管理简化到了极致。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置，然后在需要实施事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不必其他方式逊色。