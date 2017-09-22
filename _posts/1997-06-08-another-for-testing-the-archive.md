---
layout: postcn
---
title: "面试题精华之SpringMVC"
---
date: 2017-09-22 14:20:00 +0800
lang: cn
nav: post
category: test
tags: [SpringMVC]
---

* content
{:toc}

<p>另一篇测试文章归档功能的文章</p>
<!-- more -->
<p>另一篇测试文章归档功能的文章</p>


## 1.web项目中怎么接入SpringMVC？
在web.xml中写入：
```xml
<!-- SpringMvc的配置 -->
  <servlet>
    <servlet-name>springMvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath*:spring-mvc.xml</param-value>
    </init-param>
    <!--容器启动就会加载这个servlet-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springMvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```
### DispatcherServlet 有什么作用
http://www.cnblogs.com/shilin000/p/4759015.html

一. DispatcherServlet是前端控制器设计模式的实现，提供Spring Web MVC的集中访问点，而且负责职责的分派，而且与Spring IoC容器无缝集成，从而可以获得Spring的所有好处。

DispatcherServlet主要用作职责调度工作，本身主要用于控制流程，主要职责如下：

1、文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；

2、通过HandlerMapping，将请求映射到处理器（返回一个HandlerExecutionChain，它包括一个处理器、多个HandlerInterceptor拦截器）；

3、通过HandlerAdapter支持多种类型的处理器(HandlerExecutionChain中的处理器)；

4、通过ViewResolver解析逻辑视图名到具体视图实现；

5、本地化解析；

6、渲染具体的视图等；

7、如果执行过程中遇到异常将交给HandlerExceptionResolver来解析。

二. DispatcherServlet使用的特殊的bean:

DispatcherServlet默认使用WebApplicationContext作为上下文，因此我们来看一下该上下文中有哪些特殊的Bean：

1、Controller：处理器/页面控制器，做的是MVC中的C的事情，但控制逻辑转移到前端控制器了，用于对请求进行处理；

2、HandlerMapping：请求到处理器的映射，如果映射成功返回一个HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象；如BeanNameUrlHandlerMapping将URL与Bean名字映射，映射成功的Bean就是此处的处理器；

3、HandlerAdapter：HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；如SimpleControllerHandlerAdapter将对实现了Controller接口的Bean进行适配，并且掉处理器的handleRequest方法进行功能处理；

4、ViewResolver：ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；如InternalResourceViewResolver将逻辑视图名映射为jsp视图；

5、LocalResover：本地化解析，因为Spring支持国际化，因此LocalResover解析客户端的Locale信息从而方便进行国际化；

6、ThemeResovler：主题解析，通过它来实现一个页面多套风格，即常见的类似于软件皮肤效果；

7、MultipartResolver：文件上传解析，用于支持文件上传；

8、HandlerExceptionResolver：处理器异常解析，可以将异常映射到相应的统一错误界面，从而显示用户友好的界面（而不是给用户看到具体的错误信息）；

9、RequestToViewNameTranslator：当处理器没有返回逻辑视图名等相关信息时，自动将请求URL映射为逻辑视图名；

10、FlashMapManager：用于管理FlashMap的策略接口，FlashMap用于存储一个请求的输出，当进入另一个请求时作为该请求的输入，通常用于重定向场景，后边会细述。

三. 解析DispatcherServlet：
1.HttpServletBean继承HttpServlet,因此在Web容器启动时将调用它的init方法，该初始化方法的主要作用
：：：将Servlet初始化参数（init-param）设置到该组件上（如contextAttribute、contextClass、namespace、contextConfigLocation），通过BeanWrapper简化设值过程，方便后续使用；

：：：提供给子类初始化扩展点，initServletBean()，该方法由FrameworkServlet覆盖。
2、FrameworkServlet继承HttpServletBean，通过initServletBean()进行Web上下文初始化，该方法主要覆盖一下两件事情：
    一.初始化web上下文；
    二.提供给子类初始化扩展点；

3、DispatcherServlet继承FrameworkServlet，并实现了onRefresh()方法提供一些前端控制器相关的配置
## 2.*-mvc.xml中主要配置哪些内容？
http://blog.csdn.net/mrslw/article/details/52824145
需要实现基本功能的配置 
1 配置 <mvc:annotation-driven/>
2 配置 <context:component-scan base-package="com.springmvc.controller"/> //配置controller的注入
3 配置视图解析器
<mvc:annotation-driven/>   相当于注册了DefaultAnnotationHandlerMapping(映射器)和AnnotationMethodHandlerAdapter(适配器)两个bean.即解决了@Controller注解的使用前提配置。 
context:component-scan  对指定的包进行扫描，实现注释驱动Bean定义，同时将bean自动注入容器中使用。即解决了@Controller标识的类的bean的注入和使用。

如何配置注解方式的控制器管理？
使用@Controller 定义一个控制器
使用@RequestMapping 映射请求
http://blog.csdn.net/caolaosanahnu/article/details/7391393

如何访问到静态的文件,如jpg,js,css
如何你的DispatcherServlet拦截"*.do"这样的有后缀的URL，就不存在访问不到静态资源的问题。 
如果你的DispatcherServlet拦截"/"，为了实现REST风格，拦截了所有的请求，那么同时对*.js,*.jpg等静态文件的访问也就被拦截了。 
在spring-mvc.xml中配置：
```
<!--<mvc:default-servlet-handler />-->
  <!-- 静态资源，无需过控制器 -->
  <mvc:resources mapping="/static/**" location="/static/"/>
```
http://www.cnblogs.com/JAYIT/p/4149232.html

## 3.@ModelAttribute的含义及用法
@ModelAttribute 绑定请求参数到命令对象
 
@ModelAttribute一个具有如下三个作用：
①绑定请求参数到命令对象：放在功能处理方法的入参上时，用于将多个请求参数绑定到一个命令对象，从而简化绑定流程，而且自动暴露为模型数据用于视图页面展示时使用；
②暴露表单引用对象为模型数据：放在处理器的一般方法（非功能处理方法）上时，是为表单准备要展示的表单引用对象，如注册时需要选择的所在城市等，而且在执行功能处理方法（@RequestMapping 注解的方法）之前，自动添加到模型对象中，用于视图页面展示时使用；
③暴露@RequestMapping 方法返回值为模型数据：放在功能处理方法的返回值上时，是暴露功能处理方法的返回值为模型数据，用于视图页面展示时使用。

一、绑定请求参数到指定对象     
 
Java代码  收藏代码
public String test1(@ModelAttribute("user") UserModel user)  
 只是此处多了一个注解@ModelAttribute("user")，它的作用是将该绑定的命令对象以“user”为名称添加到模型对象中供视图页面展示使用。我们此时可以在视图页面使用${user.username}来获取绑定的命令对象的属性。
 
 
如请求参数包含“?username=zhang&password=123&workInfo.city=bj”自动绑定到user 中的workInfo属性的city属性中。
 
```
@RequestMapping(value="/model2/{username}")  
public String test2(@ModelAttribute("model") DataBinderTestModel model) 
```
URI 模板变量也能自动绑定到命令对象中， 当你请求的URL 中包含“bool=yes&schooInfo.specialty=computer&hobbyList[0]=program&hobbyList[1]=music&map[key1]=value1&map[key2]=value2&state=blocked”会自动绑定到命令对象上。当URI模板变量和请求参数同名时，URI模板变量具有高优先权。 
 
 
二、暴露表单引用对象为模型数据 
```
/** 
 * 设置这个注解之后可以直接在前端页面使用hb这个对象（List）集合 
 * @return 
 */  
@ModelAttribute("hb")  
public List<String> hobbiesList(){  
    List<String> hobbise = new LinkedList<String>();  
    hobbise.add("basketball");  
    hobbise.add("football");  
    hobbise.add("tennis");  
    return hobbise;  
}  
```

```
<br>  
初始化的数据 ：    ${hb }  
<br>  
  
    <c:forEach items="${hb}" var="hobby" varStatus="vs">  
        <c:choose>  
            <c:when test="${hobby == 'basketball'}">  
            篮球<input type="checkbox" name="hobbies" value="basketball">  
            </c:when>  
            <c:when test="${hobby == 'football'}">  
                足球<input type="checkbox" name="hobbies" value="football">  
            </c:when>  
            <c:when test="${hobby == 'tennis'}">  
                网球<input type="checkbox" name="hobbies" value="tennis">  
            </c:when>  
        </c:choose>  
    </c:forEach>  
```

 备注:
1、通过上面这种方式可以显示出一个集合的内容
2、上面的jsp代码使用的是JSTL，需要导入JSTL相关的jar包

```
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
``` 

三、暴露@RequestMapping方法返回值为模型数据 

```
public @ModelAttribute("user2") UserModel test3(@ModelAttribute("user2") UserModel user)  
```
```
大家可以看到返回值类型是命令对象类型，而且通过@ModelAttribute("user2")注解，此时会暴露返回值到模型数据（ 名字为user2 ） 中供视图展示使用
 
@ModelAttribute 注解的返回值会覆盖@RequestMapping 注解方法中的@ModelAttribute 注解的同名命令对象
http://hbiao68.iteye.com/blog/1948380

## 4.如何将url映射至控制器的方法？
### RestFul架构风格注解

@RequestBody 
该注解用于读取Request请求的body部分数据，使用系统默认配置的 
HttpMessageConverter进行解析， 
然后把相应的数据绑定到要返回的对象上 ,再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上

@ResponseBody 
该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区

@PathVariable 
绑定 URL 占位符到入参, 请求URI中的模板变量部分到处理器功能处理方法的方法参数上的绑定

@ExceptionHandler 
注解到方法上，出现异常时会执行该方法

@ControllerAdvice 
使一个Contoller成为全局的异常处理类，类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常

URL路径映射

普通URL路径映射

@RequestMapping(value={“/test1”, “/user/create”})：多个URL路径可以映射到同一个处理器的功能处理方法，组合使用是或的关系，即“/test1”或“/user/create”请求URL路径都可以映射到@RequestMapping指定的功能处理方法。

### URI模板模式映射

@RequestMapping(value=”/users/{userId}”)：{×××}占位符， 请求的URL可以是 “/users/123456”, 通过@PathVariable可以提取URI模板模式中的{×××}中的×××变量。

@RequestMapping(value=”/users/{userId}/topics/{topicId}”)

@RequestMapping(value=”/products/{categoryCode:\d+}-{pageNumber:\d+}”)：可以匹配“/products/123-1, 通过@PathVariable提取模式中的{×××：正则表达式匹配的值}中的×××变量

正则表达式风格的URL路径映射是一种特殊的URI模板模式映射：

URI模板模式映射是{userId}，不能指定模板变量的数据类型，如是数字还是字符串；

正则表达式风格的URL路径映射，可以指定模板变量的数据类型，可以将规则写的相当复杂
请求方法映射限定

@RequestMapping(value=”/methodOr”, method = {RequestMethod.POST, RequestMethod.GET})：即请求方法可以是 GET 或 POST。

DispatcherServlet默认开启对 GET、POST、PUT、DELETE、HEAD的支持；
http://blog.csdn.net/scplove/article/details/52294958

## 5.如何获取url路径中的变量？
http://www.cnblogs.com/xiaoxi/p/5695783.html
通过@PathVariable获取路径中的参数(还有其他5种方法)

@DeleteMapping("/{id}/{roleid}")
    public Result moveRoleWithUserById(@PathVariable("id") Integer userId,@PathVariable("roleid") Integer roleId){
        userWithRoleService.moveRoleWithUserById(userId,roleId);
        return ResultGenerator.genSuccessResult();
    }


@PathValue和@RequestParam有什么区别？
使用@RequestParam时，URL是这样的：http://host:port/path?参数名=参数值
使用@PathVariable时，URL是这样的：http://host:port/path/参数值


@RequestMapping(value="/user",method = RequestMethod.GET)  
   public @ResponseBody  
   User printUser(@RequestParam(value = "id", required = false, defaultValue = "0")  
   int id) {  
    User user = new User();  
       user = userService.getUserById(id);  
       return user;  
   }  
     
   @RequestMapping(value="/user/{id:\\d+}",method = RequestMethod.GET)  
   public @ResponseBody  
   User printUser2(@PathVariable int id) {  
       User user = new User();  
       user = userService.getUserById(id);  
       return user;  
   }  

## 6.控制器的方法怎么获得表单数据？你有多少种方式？
http://www.cnblogs.com/codelir/p/5552248.html
## 7.handler方法如何向模板视图共享数据？
1.Handler的理解
 一个handler就是一个控制器里的某个方法，而通常情况下，该方法会对应到相应的url。
2.每个Handler的返回值
 1）返回的是ModelAndView对象
ModelAndView代表的是响应的视图，还有一个向该视图传递的数据。比如：

```java
@RequestMapping(value="/getalluser.action")
public ModelAndView getAllUser(){
ModelAndView model = new ModelAndView();
List<UserInfo> list = userInfoService.getList(null);
model.addObject("users", list);//表示向页面传递的数据
model.setViewName("/index.jsp");//表示呈现的页面
return model;
}
```
model含义
MVC中MODEL一般可以理解为数据结构，反映在Java里就是一种class，这种class的主要作用是把有关系的一组数据封装在一起。比如说，在用数据库时，经常把一个数据表对应到一个class，这个class的field对应于数据表的各个field。我们就可以说这个class是一个model

## 8.handler方法如何直接输出JSON
http://blog.csdn.net/shan9liang/article/details/42181345

## 9.jsp放在webapp下和放在WEB-INF下有什么区别？
JSP存放在 WEB-INF 跟webroot的区别
放在webroot下面:优点，程序结构清晰，便于编码和维护；缺点，要加过滤器。
放在web-inf下面：优点，不用过滤器；缺点，打乱了程序结构，编码和维护麻烦点。
webroot其实是一个名字而已，在部署后是看不到的，访问的时候在url里肯定也是没有的，当然webroot也可以换成别的如webcontent等都可以。


