---
title: 提高开发和发布效率-创建不依赖于外部环境可独立运行java程序的项目（二）加入spring mvc支持
description: 我们在上一篇文章《提高开发和发布效率-创建不依赖于外部环境可独立运行java程序的项目（一）》中提到了如何简化开发、调试、打包java程序，以及如何嵌入一个tomcat到自己应用程序的方法。但大多数人在实际web开发中，不会全部从最原始的`servlet`进行开发，一般要使用一些框架简化开发。本文以流行的`spring`为例，介绍如何整合`spring  mvc`。
date: 2020-04-06 20:43:00
categories:
- [开发,java]
tags:
- 嵌入式tomcat
- maven简单项目
- java打包
- spring
- spring mvc
- spring webmvc
---

# 1.前言
我们在上一篇文章《提高开发和发布效率-创建不依赖于外部环境可独立运行java程序的项目（一）》中提到了如何简化开发、调试、打包java程序，以及如何嵌入一个tomcat到自己应用程序的方法。但大多数人在实际web开发中，不会全部从最原始的`servlet`进行开发，一般要使用一些框架简化开发。本文以流行的`spring`为例，介绍如何整合`spring  mvc`。
# 2.修改`pom.xml`文件，导入`spring  mvc`相关依赖类
**修改属性**
```xml
  <properties>
    <tomcat.version>8.5.30</tomcat.version>
    <spring.version>4.0.7</spring.version>
    <commons-lang3.version>3.1</commons-lang3.version>
  </properties>
```
**增加依赖**
```xml
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>${spring.version}.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.apache.commons</groupId>
          <artifactId>commons-lang3</artifactId>
          <version>${commons-lang3.version}</version>
      </dependency>
```
# 3.配置`spring  mvc`
## 3.1启用对spring mvc的支持
**在com.gchsoft下创建一个`web package`,然后创建`WebConfig`类**

```java
package com.gchsoft.web;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc //启用spring mvc
@ComponentScan("com.gchsoft.web") //启用组件扫描
public class WebConfig extends WebMvcConfigurerAdapter {
  @Bean
  public ViewResolver viewResolver() {
    //指定视图解析器为jsp
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
  } 
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }
  //配置静态资源处理
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
     super.addResourceHandlers(registry);
  }

}
```
## 3.2创建一个根配置`RootConfig`，以方便增加其他config
**在com.gchsoft下创建一个`config package`,然后创建`RootConfig`类**
```java
package com.gchsoft.config;

import java.util.regex.Pattern;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.core.type.filter.RegexPatternTypeFilter;

/**
配置组件扫描包为com.gchsoft，排除com.gchsoft.web
**/
@Configuration
@ComponentScan(basePackages={"com.gchsoft"},
    excludeFilters={
        @Filter(type=FilterType.CUSTOM, value=RootConfig.WebPackage.class)
    })
public class RootConfig {
  public static class WebPackage extends RegexPatternTypeFilter {
    public WebPackage() {
      super(Pattern.compile("com\\.gchsoft\\.web"));
    }
  }
}
```
## 3.3配置DispatcherServlet
>DispatcherServlet是Spring MVC的核心,它负责将请求路由到其他的组件之中.
>
**在com.gchsoft.config下添加`MySpringMvcWebInitializer `类**
```java
package com.gchsoft.config;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
import com.gchsoft.web.WebConfig;

public class MySpringMvcWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { WebConfig.class };
    }
    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

# 4.测试`spring mvc`
## 4.1创建一个控制器
**在com.gchsoft.web下创建`HomeController.java`**
```java
package com.gchsoft.web;

import static org.springframework.web.bind.annotation.RequestMethod.*;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/")
public class HomeController {

  @RequestMapping(value="/home",method = GET)
  public String home(Model model) {
    return "home";
  }
}
```
## 4.2创建一个jsp文件
**在`webapp`下创建`WEB-INF/views`目录，在下面添加`home.jsp`**
```html
<html>
  <head>
    <title>MySpring webmvc</title>
  </head>
  <body>
    <h1>Welcome to MySpring Application!</h1>
  </body>
</html>
```
## 4.3测试
**测试访问控制器**
运行我们的`App.main()`方法以启动`tomcat`，然后打开浏览器输入：[http://localhost:8080/home](http://localhost:8080/home),出现：
```shell
Welcome to MySpring Application!
```
这明控制器的home方法访问成功。

**测试前一篇文章中创建的index.jsp**

在浏览器输入：[http://localhost:8080/](http://localhost:8080/)，出现:
```shell
Hello,tomcat!
```
**测试前一篇文章中创建的`FirstServlet`**

在浏览器输入：[http://localhost:8080/first](http://localhost:8080/first)，出现:
```html
HTTP Status 404 – Not Found
Type Status Report

Message /first

Description The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.

Apache Tomcat/8.5.30
```
这说明没有找到`FirstServlet`,为啥呢？
>这是因为我们启用`spring mvc`之后，spring拦截了所有的`/*`的servlet请求转发，`FirstServlet`没有针对`spring mvc`做任何设置，当然也就无法访问了。

# 5.让我们自己的`servlet`和`spring mvc`和谐相处
## 5.1配置加载servlet
**在com.gchsoft.config下添加`MyWebApplicationInitializer `类**
```java
package com.gchsoft.config;

import com.gchsoft.servlet.FirstServlet;
import org.springframework.web.WebApplicationInitializer;
import javax.servlet.Registration;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration.Dynamic;
/**
 * Created by gch 
 */
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    public void onStartup(ServletContext servletContext) throws ServletException{
    	//加载FirstServlet
        Dynamic firstServlet=servletContext.addServlet("/firstServlet", FirstServlet.class);
        firstServlet.addMapping("/first");
    }
}
```
## 5.2修改servlet
打开`src/main/java/com/gchsoft/servlet/FirstServlet.java`,去掉`@WebServlet`注解（为什么？因为不需要了）：
```java
//@WebServlet(
//        name = "FirstServlet",
//        urlPatterns = {"/first"}
//    )
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        ServletOutputStream out = resp.getOutputStream();
        out.write("This is first servlet for embedded tomcat. ".getBytes());
        out.flush();
        out.close();
    }
}
```

## 5.3再次测试
重新运行我们的`App.main()`方法，在浏览器输入：[http://localhost:8080/first](http://localhost:8080/first)，出现:
```shell
This is first servlet for embedded tomcat. 
```
我们的`FirstServlet`又回来啦。

# 6.总结
本文介绍了一般`java`程序导入`spring mvc`的方法,以及如何与原有的`servlet`程序和谐相处；这样可以根据自己的需要来扩充、灵活的使用`spring mvc`，不必为二选一烦恼；同时，你仍然可以使用前一篇文章的方法，将`tomcat`、`spring mvc`打包成jar包，进行发布。
