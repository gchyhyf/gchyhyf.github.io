---
title: 提高开发和发布效率-创建不依赖于外部环境可独立运行java程序的项目（一）
date: 2020-04-04 21:58:00
categories:
- [开发,java]
tags:
- 嵌入式tomcat
- maven简单项目
- java打包
---
# 1.前言
一个普通的java程序要能运行，除了需要基本的JDK环境之外，通常还要依赖多个其它第三方库文件；而作为web应用程序，还需要一个基本条件，即servlet容器的支持，例如：Tomcat、Jetty、Jboss等均提供servlet容器支持。   

Java程序项目依赖的复杂性，给开发者造成一定学习和使用上的困扰，许多初学者因此备受打击，极大的消磨了学习热情。 

那有没有减少复杂度的办法呢？事实上，spring boot就提供了从开发到运行再到发布的一系列简化方案，当然这种方法要求基于spring boot技术体系---但是，引入spring boot本身也增加了复杂性，结果不是我们期望的：为了解决问题，却引入了更多要解决的问题；这正是本文的目的---向你介绍简单、实用的方案，这也有助于你了解spring boot是如何做到这一点的。

# 2.所需要的环境清单

+ 安装JDK 1.8以上
+ 安装Aapche Maven 3.6.3(最新版，当然3.0以上应该都可以)
+ 一个简单的文本编辑器

什么，Maven没用过？也不要紧，只要安装上Maven即可。   

至于如何安装，不在本文讲述之列，请自行通过搜索引擎搜索安装教程。

# 3.用maven创建一个简单的java项目
## 3.1创建一个名为testapp的项目
**创建learnjava目录，然后创建项目：**
```shell
$ cd ~
$ mkdir learnjava
$ cd ~/learnjava
$ ~/learnjava mvn -B archetype:generate \
  -DarchetypeGroupId=org.apache.maven.archetypes \
  -DgroupId=com.gchsoft.runapp \
  -DartifactId=testapp
```
这条命令会在当前用户的learnjava下创建一个testapp的目录，里面存放着maven初始化后的项目文件和内容。   

**查看项目情况**
```shell
$ ~/learnjava cd testapp
$ ~/learnjava/testapp tree
---
#可以看到目录结构如下所示
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── gchsoft
│   │   │           └── runapp
│   │   │               └── App.java
│   └── test
│       └── java
│           └── com
│               └── gchsoft
│                   └── runapp
│                       └── AppTest.java
```

## 3.2试着运行一下

```shell
$ ~/learnjava/testapp mvn compile
$ ~/learnjava/testapp mvn package
$ ~/learnjava/testapp java -cp target/testapp-1.0-SNAPSHOT.jar com.gchsoft.runapp.App
---
#会输出初始默认信息
Hello World!
```
这说明，这个简单的项目运行正常，接下来我们继续完善它。
# 4.配置为一个独立运行的包
## 4.1项目不依赖其它外部jar包
修改pom.xml文件，增加：
```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.gchsoft.runapp.app</mainClass>
                        </manifest>
                    </archive>
                    <classesDirectory>
                    </classesDirectory>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```
## 4.2项目依赖外部jar包

修改pom.xml文件，在`</manifest>`之前增加：

```xml
<addClasspath>true</addClasspath>
<classpathPrefix>lib/</classpathPrefix>
```

**运行打包后的jar文件**

经过4.1或4.2设置之后，可直接运行打包后的jar文件：

```shell
$ ~/learnjava/testapp mvn compile
$ ~/learnjava/testapp mvn package
$ ~/learnjava/testapp java -jar target/testapp-1.0-SNAPSHOT.jar
---
#会输出初始默认信息
Hello World!
```

很好，我们简化了命令行的输入。但要注意，此时打包后的testapp-1.0-SNAPSHOT.jar文件只包含项目testapp本身的文件。
## 4.3将外部jar文件一起打包到testapp-1.0-SNAPSHOT.jar

**更改pom.xml文件的构建配置**
```xml
<build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.gchsoft.runapp.app</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
>注意：这里使用了maven-assembly-plugin插件，使用该插件后，打包方式也发生变化，变为`mvn assembly:assembly`，运行后会产生 `target/testapp-1.0-SNAPSHOT-jar-with-dependencies.jar`文件，这个文件包含了项目引用的第三方包，发布到服务器时只需要发布这一个文件即可，减少对环境的依赖，方便了运维和管理。

# 5.不安装servlet容器，运行测试web应用程序
## 5.1嵌入tomcat到项目中
**修改pom.xml，添加tomcat支持**
```xml
  <properties>
    <tomcat.version>8.5.30</tomcat.version>
  </properties>
  <dependencies>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-core</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>tomcat-jasper</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>tomcat-jasper-el</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>tomcat-jsp-api</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```
## 5.2修改App.java以启动tomcat
```java
package com.gchsoft.runapp;

import java.io.File;

import org.apache.catalina.WebResourceRoot;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.startup.Tomcat;
import org.apache.catalina.webresources.DirResourceSet;
import org.apache.catalina.webresources.StandardRoot;

/**
 * Start tomcat 
 *
 */
public class App
{
    public static void main( String[] args ) throws Exception
    {
        //初始化web应用程序的根目录为项目中的src/main/webapp
        String webappDirLocation = "src/main/webapp/";
        Tomcat tomcat = new Tomcat();
        //初始化tomcat端口号
        String webPort = System.getenv("PORT");
        if(webPort == null || webPort.isEmpty()) {
            webPort = "8080";
        }
        //设置端口
        tomcat.setPort(Integer.valueOf(webPort));

        //设置web应用程序的根目录
        StandardContext ctx = (StandardContext) tomcat.addWebapp("/", new File(webappDirLocation).getAbsolutePath());
        System.out.println("configuring app with basedir: " + new File("./" + webappDirLocation).getAbsolutePath());

        //tomcat默认的class目录是"WEB-INF/classes"，我们将 target/classes也加入进来
        File additionWebInfClasses = new File("target/classes");
        WebResourceRoot resources = new StandardRoot(ctx);
        resources.addPreResources(new DirResourceSet(resources, "/WEB-INF/classes",
                additionWebInfClasses.getAbsolutePath(), "/"));
        ctx.setResources(resources);
        
        //启动
        tomcat.start();
        tomcat.getServer().await();
    }
}
```
## 5.3创建一个简单的jsp文件,index.jsp
```shell
#创建webapp目录
$ ~/learnjava/testapp mkdir src/main/webapp
#创建jsp,这里用vi--可用任何编辑器
$ ~/learnjava/testapp vi src/main/webapp/index.jsp
```
index.jsp内容如下：
```html
<html>
<title>tomcat embedded</title>
    <body>
        <h2>Hello,tomcat!</h2>
    </body>
</html>
```
## 5.4创建一个简单的servlet
```shell
$ ~/learnjava/testapp vi src/main/java/com/gchsoft/servlet/FirstServlet.java
```
FirstServlet.java内容如下：
```java
package com.gchsoft.servlet;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(
        name = "FirstServlet",
        urlPatterns = {"/first"}
    )
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
## 5.4运行tomcat并访问index.jsp,FirstServlet
**编译、打包、运行**
```shell
$ ~/learnjava/testapp mvn compile
$ ~/learnjava/testapp mvn assembly:assembly
#运行
$ ~/learnjava/testapp java -jar target/testapp-1.0-SNAPSHOT-jar-with-dependencies.jar
#出现以下信息
---
四月 04, 2020 5:39:17 下午 org.apache.catalina.core.StandardContext setPath
警告: A context path must either be an empty string or start with a '/' and do not end with a '/'. The path [/] does not meet these criteria and has been changed to []
configuring app with basedir: /Users/gch/learnjava/testapp/./src/main/webapp
四月 04, 2020 5:39:17 下午 org.apache.coyote.AbstractProtocol init
信息: Initializing ProtocolHandler ["http-nio-8080"]
四月 04, 2020 5:39:18 下午 org.apache.tomcat.util.net.NioSelectorPool getSharedSelector
信息: Using a shared selector for servlet write/read
四月 04, 2020 5:39:18 下午 org.apache.catalina.core.StandardService startInternal
信息: Starting service [Tomcat]
四月 04, 2020 5:39:18 下午 org.apache.catalina.core.StandardEngine startInternal
信息: Starting Servlet Engine: Apache Tomcat/8.5.30
四月 04, 2020 5:39:18 下午 org.apache.catalina.startup.ContextConfig getDefaultWebXmlFragment
信息: No global web.xml found
四月 04, 2020 5:39:18 下午 org.apache.jasper.servlet.TldScanner scanJars
信息: At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
四月 04, 2020 5:39:18 下午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["http-nio-8080"]
```
这表明tomcat运行成功。
**访问index.jsp,FirstServlet**
打开浏览器，
输入 `http://localhost:8080`,出现如下图所示：   
![index.jsp](/images/tomcat/embed/index.jsp.png)

打开浏览器，
输入 `http://localhost:8080/first`,出现如下图所示：   
![firstservlet](/images/tomcat/embed/firstservlet.png)

这证明我们的jsp和servlet在内置的tomcat上正确运行了。

## 5.5接下来如何做

接下来，如果想继续学习`java及web开发`，可继续在项目中添加文件，进行测试即可。

# 6.将项目导入ide

## 6.1导入IDE的好处
+ 可以更方便的编辑、调试程序代码，使用其语法提示、断点跟踪等；
+ 无需每次打包才能运行，随时启动；
+ 常用的ide开发工具有IDEA,Eclipse等。

## 6.2导入过程

下面就以以IDEA 14为例介绍导入过程，实际上这也是一般maven项目导入idea的过程，具有通用性。

**Step 1:选择File->new->Project from existing souce,如下图所示：**

![1](/images/tomcat/embed/idea_import1.png)

**Step 2:选择所建项目的pom.xml进行导入,如下图所示：**

![2](/images/tomcat/embed/idea_import2.png)

**Step 3:对项目进行设置，使用默认值即可，如下图所示：**

![3](/images/tomcat/embed/idea_import3.png)

**Step 4:选择检测到的maven项目，这里只有一个项目所以使用默认值即可，如下图所示：**

![4](/images/tomcat/embed/idea_import4.png)

**Step 5:设置项目使用的JDK，使用默认值即可，如下图所示：**

![5](/images/tomcat/embed/idea_import5.png)

**Step 6:设置项目名称，使用默认值即可，如下图所示：**

![6](/images/tomcat/embed/idea_import6.png)

最后点`finish`按钮，耐心等待IDEA完成导入即可。

## 6.3在IDEA中运行调试

找到并打开 
`src/main/java/com/gchsoft/runaap/App.java`，然后点右键选择`Run App.main()`,如下图所示：

![runapp](/images/tomcat/embed/idea_import7.png)

看到如下信息，表示成功：
![runapp console](/images/tomcat/embed/idea_import8.png)

>现在我们可以在IDEA中愉快地玩耍了,开发、调试程序.....


