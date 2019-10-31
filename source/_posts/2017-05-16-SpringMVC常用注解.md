---
title: SpringMVC常用注解
type: SpringMVC
date: 2017-05-16 10:10:10
category: 
- 后端开发
tags:
- SpringMVC
- Spring
description: SpringMVC常用注解深入解释
---

## SpringMVC常用注解

### @Controller

> - 定义了一个控制器类或者方法
> - 控制器Controller 负责处理由DispatcherServlet 分发的请求
> - 分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解
> - 使用Model对象在View和Controller层传输数据
> - 单独使用不是真正能处理请求的处理器，需要配合@RequestMapping 使用

#### 配置控制器bean
```xml
<!--方式一-->
<bean class="com.packageName.MyController"/>
<!--方式二 路径写到controller的上一层(扫描包详解见下面浅析)-->
< context:component-scan base-package = "com.packageName" />
```

<!-- more -->

### @RequestMapping

> - RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。
> - 用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。
> - RequestMapping注解有六个属性

#### value

> - 指定请求的实际地址，指定的地址可以是URI Template 模式

#### method

> - 指定请求的method类型， GET、POST、PUT、DELETE等

#### consumes

> - 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;

#### produces

> - 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

#### params

> - 指定request中必须包含某些参数值是，才让该方法处理。

#### headers

> - 指定request中必须包含某些指定的header值，才能让该方法处理请求。

### @Autowired

> - 标记bean注入时使用
> - 使用在字段或者setter方法上
> - 如果使用在字段上，就不需要再写setter方法了
> - 默认是ByType注入
> - 默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false

```java
import org.springframework.beans.factory.annotation.Autowired;
public class TestServiceImpl {
    // 下面两种@Autowired只要使用一种即可
    @Autowired
    private UserDao userDao; // 用于字段上
    
    @Autowired
    public void setUserDao(UserDao userDao) { // 用于属性的方法上
        this.userDao = userDao;
    }
}
```

> - 如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用,如下

```java
public class TestServiceImpl {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao; 
}
```

### @Resource

> - @Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。
> - @Resource并不是Spring的注解，它的包是javax.annotation.Resource，但是Spring支持该注解的注入
> - 标记bean注入时使用
> - 使用在字段或者setter方法上
> - 如果使用在字段上，就不需要再写setter方法了
> - 默认按照ByName自动注入
> - @Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。
> - 所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。
> - 如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。
> - 最好是将@Resource放在setter方法上，因为这样更符合面向对象的思想，通过set、get去操作属性，而不是直接去操作属性。

```java
import javax.annotation.Resource;
public class TestServiceImpl {
    // 下面两种@Resource只要使用一种即可
    @Resource(name="userDao")
    private UserDao userDao; // 用于字段上
    
    @Resource(name="userDao")
    public void setUserDao(UserDao userDao) { // 用于属性的setter方法上
        this.userDao = userDao;
    }
}
```

#### @Resource装配Bean的顺序
> - ①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。  
> - ②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。  
> - ③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。  
> - ④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。  

### @ModelAttribute

> - 该Controller的所有方法在调用前，先执行此@ModelAttribute方法，可用于注解和方法参数中
> - 应用在BaseController当中，所有的Controller继承BaseController，即可实现在调用Controller时，先执行@ModelAttribute方法。
> - 标记在处理器方法参数上的时候，表示该参数的值将从模型或者 Session 中取对应名称的属性值，该名称可以通过 @ModelAttribute(“attributeName”) 来指定，若未指定，则使用参数类型的类名称（首字母小写）作为属性名称。
> - 该注解有两个用法，一个是用于方法上，一个是用于参数上；
> - 用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；
> - 用于参数上时： 用来通过名称对应，把相应名称的值绑定到注解的参数bean上；要绑定的值来源于：
> - - A） @SessionAttributes 启用的attribute 对象上；
> - - B） @ModelAttribute 用于方法上时指定的model对象；
> - - C） 上述两种情况都没有时，new一个需要绑定的bean对象，然后把request中按名称对应的方式把值绑定到bean中。

> 用到方法上@ModelAttribute的示例代码
> 这种方式实际的效果就是在调用@RequestMapping的方法之前，为request对象的model里put（“account”， Account）。
```java
@ModelAttribute  
public Account addAccount(@RequestParam String number) {  
    return accountManager.findAccount(number);  
} 
```

> 用在参数上的@ModelAttribute示例代码
> 首先查询 @SessionAttributes有无绑定的Pet对象，若没有则查询@ModelAttribute方法层面上是否绑定了Pet对象，若没有则将URI template中的值按对应的名称绑定到Pet对象的各属性上
```java
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)  
public String processSubmit(@ModelAttribute Pet pet) {  
     
} 
```

### @SessionAttributes
> - 支持使用 @ModelAttribute 和 @SessionAttributes 在不同的模型（model）和控制器之间共享数据。 @ModelAttribute 主要有两种使用方式，一种是标注在方法上，一种是标注在 Controller 方法参数上。
> - @SessionAttributes即将值放到session作用域中，写在class上面。
> - @SessionAttributes 属性标记哪些是需要存放到 session 中的
> - 等处理器方法执行完成后 Spring 才会把模型中对应的属性添加到 session 中
> - 该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。
> - 该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象；

```java
@Controller  
@RequestMapping("/editPet.do")  
@SessionAttributes("pet")  
public class EditPetForm {  

} 
```

### @PathVariable

> - 用于将请求URL中的模板变量映射到功能处理方法的参数上，即取出uri模板中的变量作为参数

```java
@Controller  
public class TestController {  
     @RequestMapping(value="/user/{userId}/roles/{roleId}",method = RequestMethod.GET)  
     public String getLogin(@PathVariable("userId") String userId,  
         @PathVariable("roleId") String roleId){  
         System.out.println("User Id : " + userId);  
         System.out.println("Role Id : " + roleId);  
         return "hello";  
     }  
     @RequestMapping(value="/product/{productId}",method = RequestMethod.GET)  
     public String getProduct(@PathVariable("productId") String productId){  
           System.out.println("Product Id : " + productId);  
           return "hello";  
     }  
     @RequestMapping(value="/javabeat/{regexp1:[a-z-]+}",  
           method = RequestMethod.GET)  
     public String getRegExp(@PathVariable("regexp1") String regexp1){  
           System.out.println("URI Part 1 : " + regexp1);  
           return "hello";  
     }  
}
```

### @RequestParam

> - 获取请求url中的参数值
> - 类似request.getParameter("name")
> - defaultValue 表示设置默认值，
> - required 通过boolean设置是否是必须要传入的参数，
> - value 值表示接受的传入的参数类型。
> - 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；

```java
@Controller  
public class TestController {  
     @RequestMapping(value="/update",method=RequestMethod.POST)
	public Object update(@RequestParam(value="id",required = true, defaultValue = 0)int id){
        return "OK";  
    }
}
```

### @RequestBody

> - 获取请求中的数据
> - 常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；
> - 因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容
```java
@RequestMapping(value = "/something", method = RequestMethod.PUT)  
public void handle(@RequestBody String body, Writer writer) throws IOException {  
    writer.write(body);  
}
```

### @ResponseBody

> - 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区
> - 返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用

### @Component

> - 相当于通用的注解，当不知道一些类归到哪个层时使用，但是不建议。

### @Repository

> - 用于注解dao层，在daoImpl类上面注解。
> - @Repository, @Service 和 @Controller有着类似的作用

## 注：

---

### RequestMapping 详细说明

#### 支持通配符“*”
```java
@Controller
@RequestMapping ( "/myTest" )
public class MyController {
    @RequestMapping ( "*/wildcard" )
    public String testWildcard() {
       System. out .println( "wildcard------------" );
       return "wildcard" ;
    }  
}
```

/myTest/whatever/wildcard.do          :heavy_check_mark:

####  params属性
- 表示参数param1 的值必须等于value1 ，参数param2 必须存在，值无所谓，参数param3 必须不存在
```java
@RequestMapping (value= "testParams" , params={ "param1=value1" , "param2" , "!param3" })
public String testParams() {
    return "testParams" ;
}
```
- /testParams.do?param1=value1&param2=value2                             :x:
- /testParams.do?param1=value1&param2=value2&param3=value3               :heavy_check_mark:

#### method属性
```java
@RequestMapping (value= "testMethod" , method={RequestMethod. GET , RequestMethod. DELETE })
public String testMethod() {
    return "method" ;
}
```
- method 参数限制了以GET 或DELETE 方法请求/testMethod 的时候才能访问到该Controller 的testMethod 方法

#### headers属性
```java
@RequestMapping (value= "testHeaders" , headers={ "host=localhost" , "Accept" })
public String testHeaders() {
    return "headers" ;
}
```
- 只有当请求头包含Accept 信息，且请求的host 为localhost 的时候才能正确的访问到testHeaders 方法

#### 支持的方法参数类型
- 使用方法：在方法的参数中加入
- （ 1 ）HttpServlet 对象，主要包括HttpServletRequest 、HttpServletResponse 和HttpSession 对象。
- （ 2 ）Spring 自己的WebRequest 对象。
- （ 3 ）InputStream 、OutputStream 、Reader 和Writer 。
- - InputStream 和Reader 是针对HttpServletRequest 而言的，可以从里面取数据；
- - OutputStream 和Writer 是针对HttpServletResponse 而言的，可以往里面写数据。
- （ 4 ）使用@PathVariable 、@RequestParam 、@CookieValue 和@RequestHeader 标记的参数。
- （ 5 ）使用@ModelAttribute 标记的参数。
- （ 6 ）java.util.Map 、Spring 封装的Model 和ModelMap 。
- - 用来封装模型数据，用来给视图做展示
- （ 7 ）实体类
- - 可以用来接收上传的参数。
- （ 8 ）Spring 封装的MultipartFile
- - 用来接收上传文件的。
- （ 9 ）Spring 封装的Errors 和BindingResult 对象。
- - 这两个对象参数必须紧接在需要验证的实体对象参数之后，它里面包含了实体对象的验证结果。

#### 支持的返回类型

- （ 1 ）一个包含模型和视图的ModelAndView 对象。
- （ 2 ）一个模型对象，这主要包括Spring 封装好的Model 和ModelMap ，以及java.util.Map ，当没有视图返回的时候视图名称将由RequestToViewNameTranslator 来决定。
- （ 3 ）一个View 对象。这个时候如果在渲染视图的过程中模型的话就可以给处理器方法定义一个模型参数，然后在方法体里面往模型中添加值。
- （ 4 ）一个String 字符串。这往往代表的是一个视图名称。这个时候如果需要在渲染视图的过程中需要模型的话就可以给处理器方法一个模型参数，然后在方法体里面往模型中添加值就可以了。
- （ 5 ）返回值是void 。这种情况一般是我们直接把返回结果写到HttpServletResponse 中了，如果没有写的话，那么Spring 将会利用RequestToViewNameTranslator 来返回一个对应的视图名称。如果视图中需要模型的话，处理方法与返回字符串的情况相同。
- （ 6 ）如果处理器方法被注解@ResponseBody 标记的话，那么处理器方法的任何返回类型都会通过HttpMessageConverters 转换之后写到HttpServletResponse 中，而不会像上面的那些情况一样当做视图或者模型来处理。
- （ 7 ）除以上几种情况之外的其他任何返回类型都会被当做模型中的一个属性来处理，而返回的视图还是由RequestToViewNameTranslator 来决定，添加到模型中的属性名称可以在该方法上用@ModelAttribute(“attributeName”) 来定义，否则将使用返回类型的类名称的首字母小写形式来表示。使用@ModelAttribute 标记的方法会在@RequestMapping 标记的方法执行之前执行。

### @PathVariable和@RequestParam的区别

> handler method 参数绑定常用的注解,我们根据他们处理的Request的不同内容部分分为四类：（主要讲解常用类型）
- A、处理requet uri 部分（这里指uri template中variable，不含queryString部分）的注解：   @PathVariable;
- B、处理request header部分的注解：   @RequestHeader, @CookieValue;
- C、处理request body部分的注解：@RequestParam,  @RequestBody;
- D、处理attribute类型是注解： @SessionAttributes, @ModelAttribute;

#### @PathVariable
- 通过 @Pathvariable注解绑定它传过来的值到方法的参数
```java
@Controller  
@RequestMapping("/owners/{ownerId}")  
public class RelativePathUriTemplateController {  
    @RequestMapping("/pets/{petId}")  
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model){      
    // implementation omitted   
    }  
}
```

#### @RequestHeader
- @RequestHeader 注解可以把Request请求header部分的值绑定到方法的参数上。
```xml
Host                    localhost:8080  
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9  
Accept-Language         fr,en-gb;q=0.7,en;q=0.3  
Accept-Encoding         gzip,deflate  
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7  
Keep-Alive              300  
```

```java
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,@RequestHeader("Keep-Alive") long keepAlive)  {  
}
```

#### @CookieValue
- @CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。
```java
@RequestMapping("/displayHeaderInfo.do")  
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {  
}
```

#### component-scan浅析

>   @Repository, @Service 和 @Controller 是 @Component 的子注解
- `<context:component-scan>`有一个use-default-filters属性，属性默认为true,表示会扫描指定包下的全部的标有@Component的类，并注册成bean.也就是@Component的子注解@Service,@Reposity等

> `<context:annotation-config/>` 提供了两个子标签
- `<context:include-filter>` 指定扫描的路径
- `<context:exclude-filter>` 排除扫描的路径

- use-default-filters属性为false示例

```xml
<context:component-scan base-package="com.tan" use-default-filters="false">
        <context:include-filter type="regex" expression="com.tan.*"/>//注意后面要写.*
</context:component-scan>
```

- use-default-filters属性为true示例

```xml
<context:component-scan base-package="com.tan" >
        <context:include-filter type="regex" expression=".controller.*"/>
        <context:include-filter type="regex" expression=".service.*"/>
        <context:include-filter type="regex" expression=".dao.*"/>
</context:component-scan>
```

- 相当于：

```xml
<context:component-scan base-package="com.tan" >
        <context:exclude-filter type="regex" expression=".model.*"/>
</context:component-scan>
```

- 无论哪种情况`<context:include-filter>`和`<context:exclude-filter>`都不能同时存在

### 参考文献

[博客园leskang](https://www.cnblogs.com/leskang/p/5445698.html)  

### SpringMVC中文文档

[国内W3CSchool](https://www.w3cschool.cn/spring_mvc_documentation_linesh_translation/)  
[国外GitBook](https://linesh.gitbooks.io/spring-mvc-documentation-linesh-translation/content/)