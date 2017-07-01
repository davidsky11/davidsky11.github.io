---
layout: post
title:  SpringMVC框架整理
date:   2016-06-06 00:00:00 +0800
categories: Spring
tag: 总结
---

* content
{:toc}

### MVC概述 ###
&emsp;&emsp;Model（模型） - View（视图） - Controller（控制器）
+ Model（模型）：
	业务模型：业务流程和业务逻辑
	数据模型：页面显示的数据 - 数据库保存的数据对应的JavaBean
+ View（视图）：
	和用户直接交互的页面、程序界面
+ Controller（控制器）：
	调度器控制整个网站的转发逻辑

### SpringMVC概述 ###
&emsp;&emsp;SpringMVC是基于MVC理念的表现层框架。

### SpringMVC运行流程 ###
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/59314165.jpg)

### SpringMVC的运行逻辑 ###
+ 客户端请求提交到DispatcherServlet（SpringMVC前端控制器）
+ 由DispatcherServlet控制器查询一个或多个HandlerMapping，找到处理请求的Controller，根据HandlerMapping提供的信息，再去找到真正解析和调用业务逻辑方法的适配器
+ DispatcherServlet将请求提交到Controller（也称为Handler），也就是使用适配器去执行目标方法 mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
+ Controller调用业务逻辑处理后，返回ModelAndView（ModelAndView包含了要去向视图的信息，以及其他信息）
+ DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图ViewResoler进行视图解析，调用render解析出去哪个页面
+ 视图负责将结果显示到客户端，渲染  RequestDispatcher rd    rd.forward(requestToExpose, response);

&emsp;&emsp;总结：
<font color="darkblue" size="5" face="黑体">请求到达 --> 找到能够执行目标方法的适配器 --> 执行目标方法 --> 返回视图信息以及数据信息（MV） --> 根据返回信息渲染视图 --> 展示视图</font>
	
### 相关包 ###
	1、基础包
	commons-logging
	net.sf.cglib
	org.aopalliance
	org.aspectj.weaver
	spring-aop
	spring-aspects
	spring-bean
	spring-context
	spring-core
	spring-expression
	spring-web
	spring-webmvc
	
	2、在web.xml配置前端控制器
	```
	<!-- SpringMVC的前端控制器，它是一个真实的Servlet继承于HttpServlet -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.Dispatcherservlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<!-- 这个location对应SpringMVC配置文件 -->
			<param-value>classpath: xx/springmvc.xml</param-value>
		</init-param>
		<!-- 这个是启动的优先级，数值越小，优先级越高 -->
		<load-on-startup>1<load-on-startup>
	</servlet>

	<!-- 配置拦截哪些请求 -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<!-- 这里配置成‘/’，可以拦截所有请求同时又覆盖了DefaultServlet的配置，这样所有的请求都由我们的前端控制器拦截 -->
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	```

	3、SpringMVC配置文件
	```
	<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/context
			http://www.springframework.org/schema/context/spring-context-4.0.xsd" rel="nofollow" target="_blank" >

		<!-- 开启包的自动扫描，将加了controller注解的handle类注入 -->
		< context:component-scan base-package="com.springmvc.handle"></context:component-scan>
		<!-- 配置视图解析器，用于渲染mv -->
		<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
			<!-- 给返回地址配置一个前缀 -->
			<property name="prefix" value="/"></property>
			<!-- 配置一个后缀 -->
			<property name="suffix" value=".jsp"></property>
			<!--/success.jsp(前缀+返回结果+后缀共同构成转发地址)-->
		</bean>

	</beans>
	```

@RequestMapping
	1、<font color="red">在类上标注</font>，表示该类中的所有handle方法都以这个映射地址开始。<font color="red">相对于Web应用的根目录</font>
	2、<font color="red">在方法上标注</font>，若类上没有@RequestMapping，直接表示请求地址，若类上有，类上映射 + 方法上映射共同构成请求机构。<font color="red">相对于标注在类上的URL</font>

	作用：DispatcherServlet截获请求后，就通过控制器上@RequestMapping提供的映射信息确定请求对应的处理方法。

	属性：
	value：设置请求地址，在类上标注和在方法上标注/表示的含义不同
	method：设置某个方法处理什么样的请求，method是个数组表示可以处理多个符合要求的请求。默认可以处理任意请求
	params：设置请求参数（可以设置多个请求参数）（支持简单的表达式）
		当配置了params时，必须在请求的时候传入配置的参数，不匹配或者不传都会报错404
		当设置多个请求参数的时候必须都设置，否则报错404
		结论：params设置的请求参数必须精确匹配，否则报错
		params = !user 表示请求参数不能设置为user，其他的都可以
		params = user!=1 表示请求参数值不能等于1

	含有params的代码：
	```
	/**
	 * 这个请求地址中必须有password，user可以有可以没有，但是若有user的值不能是1，不能有username
         * 还可以有其他请求
         * @return
         */
	@RequestMapping(value="/test5", params={"user!=1", "password", "!username"})
	public String test5() {
		System.out.println("接受到请求");
		return "success";
	}
	```

	headers设置请求头：
	发送请求的请求头必须包含headers中的内容，不同的浏览器请求头不同，可以用于区分不同的浏览器
	```
	/**
	 * headers (设置只有某一浏览器可以访问)
	 * @return
	 */
	@RequestMapping(value="/testheader", headers="User-Agent=Mozilla/5.0 (Windows NT 10.0; WOW64; rv:49.0) Gecko/20100101 Firefox/49.0")
	public String test6() {
		System.out.println("接受到请求");
		return "success";
	}
	```
	设置@RequestMapping的条件后，在发送请求的时候，必须满足上边的所有规则，否则报错，什么也不写表示默认规则。

	return String 转发地址，在springmvc配置文件中配置的视图解析器还会对这个地址进行包装。


## Restful编程风格 ##
	### 简介 ###

	REST：即Representational State Transfer。<font color="red">（资源）表现层状态转化。</font>，是目前最流行的一种互联网软件架构。它结构清晰、符合标准、易于理解、拓展方便，所以正得到越来越多网站的采用。
	<font color="red">资源（Resources）：</font>网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的存在。可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，因此URI即为每一个资源的独一无二的识别符。
	<font color="red">表现层（Representation）：</font>把资源具体呈现出来的形式，叫做它的表现层（Representation）。比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式。
	<font color="red">状态转化（State Transfer）：</font>每发出一个请求，就代表了客户端和服务器的一次交互过程。HTTP协议，是一个无状态协议，即所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生“状态转化”（State Transfer）。而这种转化是建立在表现层之上的，所有就是“表现层状态转化”。具体说，就是HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：GET用来获取资源，POST用来新建资源，PUT用来更新资源，DELETE用来删除资源。

	### @PathVariable ###
	带占位符的URL是Spring3.0新增的功能。
	通过@PathVariable可以将URL中占位符参数绑定到控制器处理方法的入参：URL中的{xxx}占位符可以通过@PathVariable("xxx") 绑定到操作方法的入参中。
	这样在@RequestMapping配置的映射地址设置占位符，同时使用@PathVariable注解可以获取到

	### HiddenHttpMethodFilter ###
	浏览器form表单只支持GET和POST请求，而DELETE、PUT等method并不支持，Spring3.0添加了一个过滤器，可以将这些请求转换为标准的http方法，使得GET、POST、PUT和DELETE请求。
	原生的jsp页面无法发送DELETE请求和PUT请求，要想发送需要在web.xml中进行配置
	```
	<!-- 配置配置org.springframework.web.filter.HiddenHttpMethodFilter来支持页面发起restful请求 PUT、DELETE -->
	<filter>
		<filter-name>HiddenHttpMethodFilter</filter-name>
		<filter-class>org.springframework.web.filter. HiddenHttpMethodFilter </filter-class>
	</filter>
	<filter-mapping>
		<filter-name>HiddenHttpMethodFilter</filter-name>
		<servlet-name>springDispatcherServlet</servlet-name>
	</filter-mapping>
	```

	一个简易的Restful风格的程序
	```
	<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>Insert title here</title>
	</head>
	<body>
		<h1>订单管理</h1>
		<!-- get请求获取订单 -->
		<a href="/myspringmvc01 /order/1">获取一个订单</a>
		<br>
		<!-- post请求添加订单 -->
		<form action="/myspringmvc01/order" method="post" >
			<input type="submit" value="新增一个订单">
		</form>
		<br>
		<!-- PUT请求修改订单 -->
		<form action="/myspringmvc01/order/1" method="post" >
			<input type="hidden" name="_method" value="PUT" >
			<input type="submit" value="修改一个订单">
		</form>
		<br>
		<!-- DELETE请求删除订单 -->
		<form action="/myspringmvc01/order/1" method="post" >
			<input type="hidden" name="_method" value="DELETE" >
			<input type="submit" value="删除一个订单">
		</form>
	</body>
	</html>
	```

	处理请求的handler方法
	```
	import org.springframework.stereotype.Controller;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;

	@Controller
	public class RestFulPathHandle {
		/**
		 *  订单处理
		 *  order 1 GET 获取订单号为1的订单
		 *  order 1 DELETE 删除订单号为1的订单
		 *  order 1 PUT 更新订单号为1的订单
		 *  order   POST 新增订单
		 * RestFul 利用Http协议中的多种请求方式实现状态转换
		 * 通过判断提交的请求类型来选择执行不同的处理请求的方法
		 * {路径，请求参数}只能表示一层路径，必须得有
		 * 利用@PathVariable注解从url中获取值给参数赋值
		 */
		@RequestMapping(value="/order/ {id}",method=RequestMethod.GET)
		public String getOrder(@PathVariable(" id")String id) {
			System.out.println("获取订单号为" + id + "的订单");
			return "success";
		}
    
		@RequestMapping( value="/order" ,method=RequestMethod.POST)
		public String saveOrder() {
			System.out.println("新增一个订单");
			return "success";
		}
    
		@RequestMapping( value="/order/{id}" ,method=RequestMethod.PUT)
		public String updateOrder(@PathVariable( "id" )String id) {
			System.out.println("更新订单号为" + id + "的订单");
			return "success";
		}

		@RequestMapping( value="/order/{id}" ,method=RequestMethod.DELETE)
		public String deleteOrder(@PathVariable( "id" )String id) {
			System.out.println("删除订单号为" + id + "的订单");
			return "success";
		}
	}
	```