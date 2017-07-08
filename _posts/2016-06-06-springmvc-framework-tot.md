---
layout: post
title:  SpringMVC框架整理
date:   2016-06-06 00:00:00 +0800
categories: SpringMVC
tag: 框架
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
	jstl
	standard
	taglibs-standard-impl
	taglibs-standard-spec
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
### 请求参数传入（请求方法签名处理） ###
&emsp;&emsp;Spring MVC通过分析处理方法的签名，将HTTP请求信息绑定到处理方法的入参中。Spring MVC对控制器处理方法签名的限制是宽松的，几乎可以按喜欢的任何方式对方法进行签名。必要时可以对方法及方法入参标注相应的注解（@PathVariable、@RequestParam、@RequestHeader等）、Spring MVC框架会将HTTP请求的信息绑定到相应的方法入参中，并根据方法的返回值类型做出相应的后续处理。

#### 获取请求参数的方式 ####
1、直接在方法的参数列表处，声明一个（保存请求参数值的）参数（入参）
&emsp;&emsp;若不适用@RequestParam注解，参数名就是请求参数的键（相当于getParameter("键")），key=参数名
&emsp;&emsp;在发送请求的时候，不传入任何值，或者键不匹配，key(参数名)=null(value=null)，键匹配不传值，值为空串。可以传入多个键值对。

&emsp;&emsp;使用@RequestParam注解可以传入多个键值对，属性：
value：默认空串（设置键）
required：默认true（必须传参数与这个键匹配的键值对，不传，或者不匹配都会400）
defaultValue：默认没有，用于设置默认值（键的默认值）

2、使用@PathVariable，@RequestHeader，@CookieValue对入参进行修饰
&emsp;&emsp;请求头包含了若干个属性，服务器可据此获知客户端的信息，通过@RequestHeader即可将请求头中的属性值绑定到处理方法的入参中
@RequestHeader修饰入参（将请求头中的某些值和入参进行绑定）
```
/**
 * 入参与请求头中的键值对（任意的）进行绑定
 * @RequestHeader   value=key(将入参（参数列表处设置的参数）和请求头中key对应的值绑定)
 * 这些注解的属性和@RequestParam注解属性一致，含义一致
 */
@RequestMapping(value="/testheader")
public String testHeader(@RequestHeader(value="User-Agent")String agent) {
	System.out.println("Agent=" + agent);
	return "success";
}

@RequestMapping(value="/testheader2")
public String testHeader2(@RequestHeader(value="Accept-Language")String language) {
	System.out.println("Accept-Language=" + language);
	return "success";
}
```

@CookieValue修饰入参（将Cookie值与入参进行绑定）
```
/**
 * 使用@RequestHeader获取Cookie，在第一次请求的时候会报500错，因为第一次请求时没有Cookie
 *  Missing header 'Cookie' of type [java.lang.String]找不见Cookie
 * 获取Cookie使用@CookieValue注解获取(设置defaultValue=""就不会出现500错)，Cookie值为空串
 *          还可以获取JSESSIONID
 * @param cookie
 * @return
 */
@RequestMapping(value="/testcookie")
public String testCookie(@CookieValue(value="Cookie", defaultValue="")String cookie) {
	System.out.println("Cookie=" + cookie);
	return "success";
}

@RequestMapping(value="/testcookie2")
public String testCookie2(@CookieValue(value="JSESSIONID",defaultValue="")String JSESSIONID) {
	System.out.println("JSESSIONID=" + JSESSIONID);
	return "success";
}
```

### 使用POJO对象绑定请求参数值 ###
&emsp;&emsp;使用表单提交数据，Spring MVC会按请求参数名和POJO属性名进行自动匹配，自动为该对象填充属性值。支持级联属性。
&emsp;&emsp;处理请求的fangfa
```
/**
 * SpringMVC框架也可以自动处理POJO(JavaBean对象)
 * 只需要让表单的属性和POJO属性一一对象，然后在方法参数列表设置一个对象引用保存这个对象，
 * 入参设置为一个对象SpringMVC框架在处理这个方法的时候会帮你自动封装，级联属性也可以自动封装
 *
 * 根据表单给的属性值，自动封装成对象
 */
@RequestMapping(value="/testpojo")
public String testPOJO(Employee employee) {
        System.out.println(employee);
        return "success";
}
```

### 解决POST中文乱码问题 ###
请求乱码：
&emsp;&emsp;post请求（原生解决方式）在服务器第一次接收到请求的时候（在filter中设置）
	```
	request.setCharacterEncoding("utf-8")
	```
&emsp;&emsp;get请求  服务器的server.xml中设置
	```
	URIEncoding="UTF-8" (8080端口号设置处设置)
	```
响应乱码：
&emsp;&emsp;response响应的时候乱码（将数据从服务器发送到浏览器的时候乱码）
	```
	response.setContentType("text/html;charset=UTF-8");
	```

&emsp;&emsp;Spring MVC框架的乱码问题解决方式：
+ get请求和原生的解决方式一致
+ post请求

web.xml中配置CharacterEncodingFilter
```
<filter>
     <filter-name>characterEncodingFilter</filter-name>
     <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
     <!-- 配置编码参数 -->
     <init-param>
     <!-- 配置请求编码设置 -->
          <param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
     </init-param>
     <init-param>
     <!-- 配置响应编码设置 设置的时候根据请求编码的参数设置-->
          <param-name>forceEncoding</param-name>
          <param-value>true</param-value>
     </init-param>
</filter>
<filter-mapping>
     <filter-name>characterEncodingFilter</filter-name>
     <!-- 对所有请求和.jsp进行拦截 -->
     <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 使用原生的ServletAPI作为入参 ###
+ HttpServletRequest
+ HttpServletResponse
+ HttpSession
+ java.security.Principal
+ Locale
+ InputStream
+ OutputStream
+ Reader
+ Writer

```
/**
 * 响应的编码测试
 * 使用SpringMVC框架的一个好处就是在想要用的时候直接声明一个变量保存就好
 * 想要使用原生的api只需要在参数列表设置就行
 * @param employee
 * @return
 */
@RequestMapping(value="/testpon")
public void testResponse(HttpServletRequest request, HttpServletResponse response) {
	System.out.println("使用原生的api不需要设置返回值");
        System.out.println("param:" + request.getParameter("user"));
        try {
            /*
             * 只有使用原生api，CharacterEncodingFilter会出现乱码，原因，没有设置text/html
             * 但是设置了响应和请求的解码格式
             *
             */
            response.setContentType("text/html");
            response.getWriter().write("你好");
        } catch (IOException e) {
            e.printStackTrace();
        }
}
```

### 响应数据传出（处理模型数据） ###
&emsp;&emsp;从服务器向页面发送响应
原生：web通过request，session，application域对象传递，然后在页面获取
1、返回ModelAndView对象 框架自动保存在request域中可以直接在页面获取
2、方法的参数列表，设置一个Map、ModelMap（实际是一个map）、Model（Spring定义的接口）类型的应用，添加到这三个类型参数中的对象会被自动保存到request域中，它们三个在SpringMVC框架中最后都被包装成BindingAwareModelMap

返回MV对象
``` 
@RequestMapping("/testresponse")
public ModelAndView testResponse() {
	//创建对象
        Employee employee1 = new Employee(1, "小红", 21, new Address("大同", "南郊区"));
        Employee employee2 = new Employee(2, "猪八戒", 2000, new Address("盘丝洞", "南郊区"));
        //创建一个ModelAndView对象
        ModelAndView mv = new ModelAndView();
        //设置视图路径，返回的String
        mv.setViewName("success");
        //添加对象到ModelAndView中
        mv.addObject("emp1", employee1);
        mv.addObject("emp2", employee2);
        return mv;
}
```

设置Map/ModelMap/Model
```
@RequestMapping("/testres01")
public String testRes01(Map<String, Object> map) {
        Employee employee1 = new Employee(1, "小红1", 21, new Address("大同", "南郊区"));
        Employee employee2 = new Employee(2, "猪八戒1", 2000, new Address("盘丝洞", "南郊区"));

        map.put("emp1", employee1);
        map.put("emp2", employee2);
        return "success";
}

@RequestMapping("/testres02")
public String testRes02(ModelMap modelMap) {
        Employee employee1 = new Employee(1, "小红2", 21, new Address("大同", "南郊区"));
        Employee employee2 = new Employee(2, "猪八戒2", 2000, new Address("盘丝洞", "南郊区"));

        modelMap.addAttribute("emp1", employee1);
        modelMap.addAttribute("emp2", employee2);

        return "success";
}

@RequestMapping("/testres03")
public String testRes02(Model model) {
        Employee employee1 = new Employee(1, "小红3", 21, new Address("大同", "南郊区"));
        Employee employee2 = new Employee(2, "猪八戒3", 2000, new Address("盘丝洞", "南郊区"));
        model.addAttribute("emp1", employee1);
        model.addAttribute("emp2", employee2);
        System.out.println(model.getClass());
        return "success";
}
```

3、如何将对象放入session域中？
&emsp;&emsp;两种方式，原生api和@SessionAttributes注解

@SessionAttributes注解只能加到类上
```
@SessionAttributes(types=Employee.class)
//value和type都可以设置多个，value将指定的对象放到session域中，type将某一类型全部放到session域中
//@SessionAttributes(value="emp1")
@Controller
public class EmployeeHandle {
 将对象放到session域中
    /**
     * 给session域中保存对象
     *  方式 ①使用原生api
     *     ②使用SessionAttributes注解 这个注解只能加到类上
     *          value表示key (表示添加到session域中的key，
     *          具体实现是，保存到request域中的时候再给session中保存一份
     *          type 表示类型，指定类型添加
     *      value,type都可以指定多个
     *      加上注解后，还需要将其添加到request中，才能添加到session
     */
    @RequestMapping("testres04")
    //只需要在参数列表设置参数就可以获取到session
    public String testRes03(HttpSession session){
        Employee employee1 = new Employee(1, "小红3", 21, new Address("大同", "南郊区"));
        Employee employee2 = new Employee(2, "猪八戒3", 2000, new Address("盘丝洞", "南郊区"));

        session.setAttribute("emp1", employee1);
        session.setAttribute("emp2", employee2);
        return "success";
    }
    @RequestMapping("testres05")
    public String testRes04(ModelMap modelMap) {
        Employee employee1 = new Employee(1, "小红3", 21, new Address("大同", "南郊区"));
        Employee employee2 = new Employee(2, "猪八戒3", 2000, new Address("盘丝洞", "南郊区"));

        modelMap.addAttribute("emp1", employee1);
        modelMap.addAttribute("emp2", employee2);
        return "success";
    }
```

### @ModelAttribute ###
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/67439550.jpg)
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/48635661.jpg)

在方法定义上使用@ModelAttribute注解：Spring MVC在调用目标处理方法前，会先逐个调用在方法级上标注了@ModelAttribute的方法。

在方法的入参前使用@ModelAttribute注解：可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数绑定到对象中，再传入入参将方法入参对象添加到模型中。

## 视图和视图解析器 ##
不论控制器返回一个String,ModelAndView,View都会转换为ModelAndView对象，由视图解析器解析视图，然后，进行页面的跳转。

请求处理方法执行完成后，最终返回一个 ModelAndView 对象。对于那些返回 String，View 或 ModeMap 等类型的处理方法，Spring MVC 也会在内部将它们装配成一个 ModelAndView 对象，它包含了逻辑名和模型对象的视图。

Spring MVC 借助视图解析器（ViewResolver）得到最终的视图对象（View），最终的视图可以是 JSP ，也可能是 Excel、JFreeChart等各种表现形式的视图

视图的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户。

为了实现视图模型和具体实现技术的解耦，Spring 在 org.springframework.web.servlet 包中定义了一个高度抽象的 View 接口： 视图对象由视图解析器负责实例化。由于视图是无状态的，所以他们不会有线程安全的问题

### 视图解析器 ###
SpringMVC 为逻辑视图名的解析提供了不同的策略， 可以在 Spring WEB 上下文中配置一种或多种解析策略，并指定他们之间的先后顺序。每一种映射策略对应一个具体的视图解析器实现类。

视图解析器的作用：将逻辑视图解析为一个具体的视图对象。

所有的视图解析器都必须实现 ViewResolver 接口。

常用的视图解析器实现类
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/13287921.jpg)


JSP 是最常见的视图技术，可以使用 InternalResourceViewResolver 作为视图解析器，helloworld使用的就是InternalResourceViewResolver
```
<!-- 配置视图解析器，用于渲染mv -->
<bean id="viewResolver" class="org.springframework.web.servlet.view. InternalResourceViewResolver ">
        <!-- 给返回地址配置一个前缀 -->
        <property name="prefix" value="/"></property>
        <!-- 配置一个后缀 -->
        <property name="suffix" value=".jsp"></property>
        <!--/success.jsp(前缀+返回结果+后缀共同构成转发地址)-->
</bean>
```

若项目中使用了 JSTL，则 SpringMVC 会自动把视图由 InternalResourceView 转为 JstlView 若使用 JSTL 的 fmt 标签则需要在 SpringMVC 的配置文件中配置国际化资源文件
```
<bean id="messageSource" class="org.springframework.context.support. ResourceBundleMessageSource ">
     <!-- 指定国际化文件的基本名 -->
    <property name="basename" value="i18n"></property>
</bean>
```

若希望直接响应通过 SpringMVC 渲染的页面，可以使用 mvc:view-controller 标签实现 （不用编写处理请求的handle方法，但是SpringMVC渲染页面）
```
<!-- 使用穿越火线，不过handle直接到指定页面  直接使用SpringMVC渲染视图  path，请求路径，view-name视图名字-->
<mvc:view-controller view-name="i18n" path="/i18n"/>
```

### Excel视图 ###
若希望使用 Excel 展示数据列表，仅需要扩展 SpringMVC 提供的 AbstractExcelView 或 AbstractJExcel View 即可。实现 buildExcelDocument() 方法，在方法中使用模型数据对象构建 Excel 文档就可以了。

AbstractExcelView 基于 POI API，而 AbstractJExcelView 是基于 JExcelAPI 的。

视图对象需要配置 IOC 容器中的一个 Bean，使用 BeanNameViewResolver 作为视图解析器即可
若希望直接在浏览器中直接下载 Excel 文档，则可以 设置响应头 Content-Disposition 的值为 attachment;filename=xxx.xls

### 重定向 ###
一般情况下，控制器方法返回字符串类型的值会被当成逻辑视图名处理
如果返回的字符串中带 forward: 或 redirect: 前缀时，SpringMVC 会对他们进行特殊处理：将 forward: 和 redirect: 当成指示符，其后的字符串作为 URL 来处理
redirect:success.jsp：会完成一个到 success.jsp 的重定向的操作
forward:success.jsp：会完成一个到 success.jsp 的转发操作



### 数据绑定流程（数据转换，数据格式化，数据校验） ###
1. Spring MVC 主框架将 ServletRequest  对象及目标方法的入参实例传递给 WebDataBinderFactory 实例，以创建 DataBinder 实例对象
2. DataBinder 调用装配在 Spring MVC 上下文中的 ConversionService 组件进行数据类型转换、数据格式化工作。将 Servlet 中的请求信息填充到入参对象中
3. 调用 Validator 组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果 BindingData 对象
4. Spring MVC 抽取 BindingResult 中的入参对象和校验错误对象，将它们赋给处理方法的响应入参

Spring MVC 通过反射机制对目标处理方法进行解析，将请求消息绑定到处理方法的入参中。数据绑定的核心部件是 DataBinder，运行机制如下：
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/43990005.jpg)

使用binderFactory创建一个绑定器，WebDataBinder
```
WebDataBinder binder = binderFactory.createBinder(request, attribute, name);
if (binder.getTarget() != null) {
    //绑定数据
    bindRequestParameters(binder, request);
    //校验数据
    validateIfApplicable(binder, parameter);
    //错误信息
    if (binder.getBindingResult().hasErrors()) {
	if (isBindExceptionRequired(binder, parameter)) {
	    throw new BindException(binder.getBindingResult());
	}
    }
}
```

## 数据绑定流程 组件 ##
1、数据类型转换组件（了解它可以自动逸类型转换器）
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/7860791.jpg)
ConversionService 是 Spring 类型转换体系的核心接口。
可以利用 ConversionServiceFactoryBean 在 Spring 的 IOC 容器中定义一个 ConversionService.
Spring 将自动识别出 IOC 容器中的 ConversionService，并在 Bean 属性配置及 Spring  MVC 处理方法入参绑定等场合使用它进行数据的转换
可通过 ConversionServiceFactoryBean 的 converters 属性注册自定义的类型转换器

2、验证组件（在服务器端对用户信息的合法性进行验证）
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/65974546.jpg)

3、错误消息保存组件（用于保存数据校验后的错误信息，然后在页面就可以进行获取）
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/16351693.jpg)

### ConversionService ###
conversionService在工作的时候，里面装配了很多的xxxConvertor，用于数据在不同类型之间的转换。

其中的一个数据类型转换器
```
final class StringToCharacterConverter implements Converter<String, Character> {

    @Override
    public Character convert(String source) {
        if (source.length() == 0) {
            return null;
        }
        if (source.length() > 1) {
            throw new IllegalArgumentException(
                    "Can only convert a [String] with length of 1 to a [Character]; string value '" + source + "'  has length of " + source.length());
        }
        return source.charAt(0);
    }

}
```

### 自定义类型转换器 ###
数据绑定核心思想：
     1、WebDataBinder用来将请求过来的数据和指定的对象进行绑定
     2、WebDataBinder里面注册了
               1）有负责数据转换/格式化工作的ConversionService组件。
                         这个组件在工作的时候使用他里面各种转换器和格式化器进行工作
               2）还注解了一堆验证组件Validators
               3）还将数据绑定的结果（失败信息）放在BindingResult

在熟知了数据绑定的流程我们可以自定义类型转换器并让其工作：

需求：
使用表单提交value="1-tom-email-男-dd",提交后框架可以帮助我们自动包装成对象。框架默认并没有符合我们需求的类型转换器，我们需要自定义一个

表单
```
<form action="${pageContext.request.contextPath }/save" method="post">
    <input type="text" name="emp" value="1-tom-email-男-dd">
    <input type="submit" value="提交">
</form>
```

1、编写一个自定义类型转换器实现Converter<S,T>
	需要实现public interface Converter<S,T> 接口
自定义类型转换器
```
package com.atguigu.convert;
import org.springframework.core.convert.converter.Converter;
import com.atguigu.entities.Employee;

/**
 * 创建自定义的类型转换器，将我们的字符串转换成对象
 * @author LENOVO
 *
 */
public class EmployeeConvert implements Converter<String, Employee> {

    @Override
    public Employee convert(String src) {
        System.out.println(src);
        if(src!=null&&!"".equals(src)) {
            //将其利用split根据分割线拆分成数组
            String[] split = src.split("-");
            Employee employee = new Employee(Integer.parseInt(split[0]), split[1], split[2], 0, null);
            return employee;
        };
        return null;
    }
}
```

2、在ConversionService组件的创建工厂中注册自定义转换器        
```
<bean id="conversionServiceFactoryBean" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <!--在ConversionService组件中创建工厂中注册自定义的转换器-->
                <bean class="com.atguigu.convert.EmployeeConvert"></bean>
            </set>
        </property>
</bean>
```

3、告诉SpringMVC用这个工厂创建ConversionService 组件
```
<mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
```

### 数据格式化 ###
在SpringMVC框架中格式化器和类型转换器都在ConversionService 组件中，我们在数据类型转换的时候，还可以根据指定的格式进行格式化并转换
但是我们自定义的类型转换器，不具有格式化功能。

原因：
ConversionServiceFactoryBean造出的ConversionService组件是DefaultConversionService，不具有格式化功能
而默认配置的ConversionService组件是DefaultFormattingConversionService

配置既可以实现类型转换（含自定义类型转换）和数据格式功能
```
<!--配置一个既有格式化功能又能数据类型转换的创建工厂  -->
<bean id="formattingConversionServiceFactoryBean" class="org.springframework.format.support. FormattingConversionServiceFactoryBean">
<property name="converters">
    <set>
	<bean class="com.atguigu.convert.EmployeeConvert"></bean>
    </set>
</property>
</bean>
<!--告诉SpringMVC用这个工厂创建ConversionService组件-->
<mvc:annotation-driven conversion-service="formattingConversionServiceFactoryBean"/>
```

提供给测试所有的handle
```
/**
 * 自定义类型转化器的测试
 * @param employee
 * @return
 */
@RequestMapping(value="/save",method=RequestMethod.POST)
public String save(@RequestParam("emp") Employee employee) {
	System.out.println(employee);
	return "input";
}
```

## annotation-driven细节 ##
```
< mvc:annotation-driven />（单独配置它，无法访问静态资源）
< mvc:default-servlet-handler />（把这个配上就有可以访问静态资源了）
因此当我们使用SpringMVC框架做表现层开发时这两个配置是标配
```

### 数据校验 ###
JSR 303 是 Java 为 Bean 数据合法性校验提供的标准框架，它已经包含在 JavaEE 6.0 中 . JSR 303 通过在 Bean 属性上标注类似于 @NotNull、@Max 等标准的注解指定校验规则，并通过标准的验证接口对 Bean 进行验证。

常用的校验注解：
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/71638549.jpg)

Hibernate Validator 是 JSR 303 的一个参考实现，除支持所有标准的校验注解外，它还支持以下的扩展注解
扩展注解
![](http://or9g8eqm7.bkt.clouddn.com/17-7-2/90606682.jpg)

Spring 4.0 拥有自己独立的数据校验框架，同时支持 JSR 303 标准的校验框架。Spring 在进行数据绑定时，可同时调用校验框架完成数据校验工作。

在 Spring MVC 中，可直接通过注解驱动的方式进行数据校验

Spring 的 LocalValidatorFactroyBean 既实现了 Spring 的 Validator 接口，也实现了 JSR 303 的 Validator 接口。只要在 Spring 容器中定义了一个 LocalValidatorFactoryBean，即可将其注入到需要数据校验的 Bean 中。

<mvc:annotation-driven/> 会默认装配好一个 LocalValidatorFactoryBean，通过在处理方法的入参上标注 @valid 注解即可让 Spring MVC 在完成数据绑定后执行数据校验的工作

Spring 本身并没有提供 JSR303 的实现，所以必须将 JSR303 的实现者的 jar 包放到类路径下。

<strong>数据校验的方式： </strong>
在已经标注了 JSR303 注解的表单/命令对象前 标注一个 @Valid，Spring MVC 框架在将请求参数绑定到该入参对象后，就会调用校验框架根据注解声明的校验规则实施校验
Spring MVC 是通过对处理方法签名的规约来保存校验结果的：前一个表单/命令对象的校验结果保存到随后的入参中，这个保存校验结果的入参必须是 BindingResult 或 Errors 类型，这两个类都位于 org.springframework.validation 包中
需校验的 Bean 对象和其绑定结果对象或错误对象时成对出现的，它们之间不允许声明其他的入参 Errors 接口提供了获取错误信息的方法，如 getErrorCount() 或 getFieldErrors(String field) BindingResult 扩展了 Errors 接口

<strong>如何获取校验结果：</strong>

在目标方法中获取校验结果
在表单/命令对象类的属性中标注校验注解，在处理方法对应的入参前添加 @Valid，Spring MVC 就会实施校验并将校验结果保存在被校验入参对象之后的 BindingResult 或 Errors 入参中。

常用方法：
      FieldError   getFieldError(String field)  
     List<FieldError>  getFieldErrors()
      Object getFieldValue(String field)
     Int getErrorCount()

<strong>如何在页面上显示错误：</strong>
Spring MVC 除了会将表单/命令对象的校验结果保存到对应的 BindingResult 或 Errors 对象中外，
     还会将所有校验结果保存到"隐含模型” 即使处理方法的签名中没有对应于表单/命令对象的结果入参，校验结果也会保存在 “隐含对象” 中。
      隐含模型中的所有数据最终将通过 HttpServletRequest 的属性列表暴露给 JSP 视图对象，因此在 JSP 中可以获取错误信息 在 JSP 页面上可通过 <form:errors path=“userName”> 显示错误消息

## 文件下载 ##

1、导包
	commons-fileupload
	commons-io

2、页面
```
<a href="download" rel="nofollow" target="_blank" >下载</a>
```

3、处理文件下载的handle
```
/**
 * 使用SpringMVC实现下载
 */
@RequestMapping("/download")
public ResponseEntity<byte[]> download() {
	//下载的东西D:\一会看的东西.txt
	HttpHeaders headers = new HttpHeaders();
	//添加响应头
	headers.add("Content-Disposition", "attachment;filename=abc.txt");
	byte[] byteArray = null;
	try {
		byteArray = IOUtils.toByteArray(new FileInputStream("D:/一会看的东西.txt"));
	} catch (Exception e) {
		e.printStackTrace();
	}
	//body响应体，headers响应头，status响应状态码
	ResponseEntity<byte[]> responseEntity = new ResponseEntity<byte[]>(byteArray, headers, HttpStatus.OK);
	return responseEntity;
}
```

## 文件上传 ##

1、导包
	commons-fileupload
	commons-io

2、配置文件上传的解析器
```
<!-- 配置一个文件上传的解析器 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 可以指定一些配置信息 -->
        <property name="defaultEncoding" value="utf-8"></property>
        <property name="maxUploadSize" value="#{1024*1024*20}"></property>
</bean>
```

3、处理文件上传请求
```
@RequestMapping("/upload")
public String upload(MultipartFile file) {
        System.out.println("获取到请求");
        //上传到D:/upload
        try {
            file.transferTo(new File("D:/upload/" + file.getOriginalFilename()));
        } catch (IllegalStateException | IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return "index";
}
```

4、扩展-多文件上传
```
/**
 * 多文件上传要指定file
 * @param file
 * @return
 */
@RequestMapping("/upload")
public String upload(@RequestParam("file") MultipartFile[] file) {
        System.out.println("获取到请求");
        //上传到D:/upload
        try {
            for (MultipartFile multipartFile : file) {

                multipartFile.transferTo(new File("D:/upload/" + multipartFile.getOriginalFilename()));
            }
        } catch (IllegalStateException | IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return "index";
}
```

补充：
@InitBinder
     由 @InitBinder 标识的方法，可以对 WebDataBinder 对象进行初始化。WebDataBinder 是 DataBinder 的子类，用于完成由表单字段到 JavaBean 属性的绑定
     @InitBinder方法不能有返回值，它必须声明为void。
     @InitBinder方法的参数通常是是 WebDataBinder

例如：设置一下不允许绑定的字段
```
/**
 * 在绑定器执行执行可以给绑定器指定一些规则，例如设置某些属性不需要绑定
 */
@InitBinder
public void initBinder(WebDataBinder binder) {
        //设置不允许绑定的字段
        binder.setDisallowedFields("lastName");
}
```

## 拦截器 ##

作用：
	在目标方法执行前后，进行一些预处理工作、进行一些扫尾工作

SpringMVC中拦截器的接口： HandlerInterceptor

该接口主要方法： 

preHandle
```
//在目标方法执行之前会被执行
boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
```

postHandle
```
//在目标方法执行之后会执行
void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)throws Exception;
```

afterCompletion
```
//请求响应完成后执行
void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception;
```

原理：
```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = processedRequest != request;

                //Determine handler for the current request.
                //动态资源使用RequestMappingHandlerMapping处理映射
                //静态资源 RequestMappingHandlerMapping是没有用的，
                //SimpleUrlHandlerMapping来处理静态资源和处理器的映射
                //所有的静态资源默认都是使用DefaultServletHttpRequestHandler来处理
                mappedHandler = getHandler(processedRequest);
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }

                // Determine handler adapter for the current request.
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (logger.isDebugEnabled()) {
                        String requestUri = urlPathHelper.getRequestUri(request);
                        logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
                    }
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }
               
               //执行拦截器的preHandle方法
                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

                try {
                    // Actually invoke the handler.
                    //适配器执行目标方法
                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                }
                finally {
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }
                }

                applyDefaultViewName(request, mv);

                //执行postHandle方法
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
             //渲染模型
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {

            //如果出现异常触发AfterCompletion
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Error err) {
            triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                return;
            }
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
}
```

①获取Handler的时候拿到的是Handler的执行链
```
mappedHandler = getHandler(processedRequest);

protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        for (HandlerMapping hm : this.handlerMappings) {
            if (logger.isTraceEnabled()) {
                logger.trace(
                        "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
            }
            HandlerExecutionChain handler = hm.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
        return null;
}
```

applyPreHandle：按照拦截器定义的顺序正序执行
```
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        if (getInterceptors() != null) {
            //正序遍历
            for (int i = 0; i < getInterceptors().length; i++) {
                HandlerInterceptor interceptor = getInterceptors()[i];
                if (!interceptor.preHandle(request, response, this.handler)) {
                    triggerAfterCompletion(request, response, null);
                    return false;
                }
                this.interceptorIndex = i;
            }
        }
        return true;
}
```

postHandle--postHandle是按照拦截器的配置顺序倒序执行
```
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
        if (getInterceptors() == null) {
            return;
        }
        for (int i = getInterceptors().length - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = getInterceptors()[i];
            interceptor.postHandle(request, response, this.handler, mv);
        }
}
```

拦截器的执行过程：
	拦截器...preHandle（正序执行）
	目标方法...
	拦截器...postHandle（逆序执行）
	jsp页面--页面渲染完成
	拦截器...afterCompletion

拦截器收尾工作算法：
```
for (int i = this.interceptorIndex; i >= 0; i--) {
	HandlerInterceptor interceptor = getInterceptors()[i];
	try {
                interceptor.afterCompletion(request, response, this.handler, ex);
	}
	catch (Throwable ex2) {
		logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
	}
}
```

### 多拦截器：处理流程 ###
流程分析（当前置发生异常）：
①前置处理一但返回false后面都不会被执行
②当第二个拦截器异常的时候

执行过程：
	拦截器...preHandle
	SecondInterceptor...preHandle
	拦截器...afterCompletion

总结：
  1）执行成功(preHandle -->true)的拦截器会执行最终的清理工作afterCompletion
  2）执行失败(preHandle -->false)的拦截器不会执行他自己的清理工作afterCompletion
  3）不管任何一个拦截器执行失败。整个拦截器的前置执行就失败了，目标方法就不会执行
  ```
  if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
  }
  ```

出现异常源码分析：

三个拦截器-前置返回true还是false的时候
          1、第一个（默认）执行成功-this.interceptorIndex = 0;
          2、第二个（First）执行成功-this.interceptorIndex = 1;
          3、第三个（Seconde）执行失败

进入清理方法
               清理方法在执行的时候也是逆序执行  

```
for (int i = this. interceptorIndex; i >= 0; i--) {
	//第一次进来  1  拿到第二个（First）拦截器  执行清理方法
	//第二次进来  0  拿到第一个(默认)拦截器  执行清理方法
	//第三个执行失败的拦截器的清理方法就被跳过了
	HandlerInterceptor interceptor = getInterceptors()[i];
          
	try {
		interceptor.afterCompletion(request, response, this.handler, ex);
	}
	catch (Throwable ex2) {
		logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
	}
}
```

当后置方法发生异常：
	 1）后置的方法postHandle，前置执行失败是不会被执行的
	2）目标方法执行出现异常，后置postHandle不会被执行
	3）一切都正常的情况下渲染时候完成后才会执行清理工作
	```
	processDispatchResult-->视图渲染完成后-->
	mappedHandler.triggerAfterCompletion(request, response, null);
	```
	 4）出现异常
		①目标方法出异常-后置不执行
		②目标方法出异常-还是会最终执行processDispatchResult渲染异常视图
		③无论何处发生异常都会执行最终的清理工作

	 主方法中的try-catch代码
	 ```
	try {
		// Actually invoke the handler.
		mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
	}
	finally {
		if (asyncManager.isConcurrentHandlingStarted()) {
			return;
		}
	}

	applyDefaultViewName(request, mv);
	mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
	 ```

### 自定义拦截器 ###
Spring MVC也可以使用拦截器对请求进行拦截处理，用户可以自定义拦截器来实现特定的功能。
     ①自定义的拦截器必须实现HandlerInterceptor接口 
     
	preHandle()：这个方法在业务处理器处理请求之前被调用，在该方法中对用户请求 request 进行处理。如果程序员决定该拦截器对请求进行拦截处理后还要调用其他的拦截器，或者是业务处理器去进行处理，则返回true；如果程序员决定不需要再调用其他的组件去处理请求，则返回false。
	
	postHandle()：这个方法在业务处理器处理完请求后，但是DispatcherServlet 向客户端返回响应前被调用，在该方法中对用户请求request进行处理。       
	
	afterCompletion()：这个方法在 DispatcherServlet 完全处理完请求后被调用，可以在该方法中进行一些资源清理的操作。

## 异常 ##

异常解析的三个类

ExceptionHandlerExceptionResolver在异常发生后，在handler里面获取全局配置@ExceptionHandler注解的方法，去运行。
```
@ExceptionHandler(value= {Exception.class})
public ModelAndView gtError2(Exception ex) {
        System.out.println("刚才的异常："+ex);
        ModelAndView mv = new ModelAndView("error");
        mv.addObject("ex", ex);
        return mv;
}
```

ResponseStatusExceptionResolver将异常处理为错误的响应状态码，发给浏览器。浏览器就能收到请求错误
使用@ResponseStatus注解标注在类上：
```
@ResponseStatus(reason="发生了我的异常..haha",value=HttpStatus.FORBIDDEN)
public class MyException extends RuntimeException{

}
```

在方法上标注，这个方法运行完成后总会返回一个错误页面，一般不标注在处理方法上
```
@ResponseStatus(value=HttpStatus.BAD_REQUEST,reason="错误的请求")
@RequestMapping(value="/testException")
public String testException(@RequestParam("a")int a) {

        try {
            System.out.println("计算结果+"+(10/a));
        } catch (Exception e) {
            // TODO Auto-generated catch block
           throw new MyException();
        }
        return "index";
}
```

DefaultHandlerExceptionResolver 处理SpringMVC发生的自己定义的异常，是使用它处理

默认的异常-MVC自己定义的异常不会受 ExceptionHandlerExceptionResolver的影响

#### 异常处理源码位置 ####
```
DispatcherServlet--->processDispatchResult--->
出现异常进行处理--->
mv = processHandlerException(request, response, handler, exception);
```

挨个处理器尝试解析异常，如果能解析，就会返回一个ModelAndView对象
默认的异常谁都没法拦截和处理，只有Tomcat自己能处理。
要想转发到一个异常页面。去web.xml中配置
```
<error-page>
    <error-code>405</error-code>
    <location>/error2.jsp</location>
  </error-page>
```

### 统一的异常处理器 ###

所有异常定义一个统一的处理规则

SimpleMappingExceptionResolver解析异常
1、不仅返回的ModelAndView有要去的异常页面
2、Model里面还有异常的值  key="exception"  value="异常对象"

配置一个统一的异常处理器
```
<bean id="handlerExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <!-- 告诉SPringMVC这个解析器能解析哪些异常 -->
        <property name="exceptionMappings">
            <props>
                <!-- key：指定异常   value：指定异常发生后转向的视图 -->
                <prop key="java.lang.Exception">error2</prop>
            </props>
        </property>
</bean>
```
在页面上获取异常信息使用${exception}

## 国际化 ##

原理分析：
1、请求过来，doDispatch
	AcceptHeaderLocaleResolver

    区域信息的获取，如果是SpringMVC处理的，他是使用区域信息解析器去拿区域信息
    区域信息的获取，如果是原始的，就是request域带来的。
    fmt:message这个标签的作用，就是从域中拿出取出信息，按照指定的区域信息从i18n配置文件中获取值
    如果域中没有指定区域信息，就是使用操作系统默认的区域  request.getLocale();
    原生： javaWeb是直接就是使用系统默认的。
    i18n:fmt:setLocale重新设置  

     ①在渲染页面的时候，先获取locale信息
     ```
     Locale locale = this.localeResolver.resolveLocale(request);
     response.setLocale(locale);

     默认使用AcceptHeaderLocaleResolver去解析locale，就是从请求中直接获取locale信息
     return request.getLocale();
     ```

     ②来到页面取值
     就可以看到,不同区域的浏览器就是按照不同语言显示的消息

      ③使用默认的国际化模式,效果就是浏览器是哪个区域,国际化就会显示哪个区域信息

### LocaleChangeInterceptor ###
作用:从请求带来的某个参数中获取指定的区域信息,如果和 SessionLocaleResolver配合

效果：
	①LocaleChangeInterceptor从请求参数中拿到区域信息，参数的名就是locale
        ②获取当前区域信息解析器LocaleResolver，调用这个解析的重新设置locale的方法

	默认的AcceptHeaderLocaleResolvere是不支持设置locale信息的，而SessionLocaleResolver是支持的
	```
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws ServletException {

		String newLocale = request.getParameter(this.paramName);
		if (newLocale != null) {
			LocaleResolver localeResolver = RequestContextUtils.getLocaleResolver(request);
			if (localeResolver == null) {
				throw new IllegalStateException("No LocaleResolver found: not in a DispatcherServlet request?");
			}
			localeResolver.setLocale(request, response, StringUtils.parseLocaleString(newLocale));
		}
		// Proceed in any case.
		return true;
	}
	```

	③页面就是按照重新设置的locale信息。fmt标签来进行解析。

	springMVC配置：
	```
	<!-- 配置国际化处理的bean -->
	<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
		<property value="i18n" name="basename"></property>
	</bean>

	<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver"></bean>
	<!-- 配置一个拦截器将我们从request中获取到的locale值传递到页面 -->
	<mvc:interceptors>
		<bean id="LocaleChangeInterceptor" class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"></bean>
	</mvc:interceptors>
	```