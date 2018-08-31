[TOC]

# Spring Boot

## 1.工程结构解析

Spring Boot的基础结构有三大块:  
* src/main/java:主程序入口HelloApplication,可以通过直接运行该类来启动Spring Boot应用。
* src/main/resource:配置目录，该目录用来存放应用的一些配置信息。
* src/test:单元测试目录。通过JUnit 4实现，可以直接用运行Spring Boot应用的测试。
## 2.Maven配置分析  
> 1. Spring Boot默认将工程打包为jar的形式，因为默认的web模块依赖会包含嵌入式的Tomcat，这样使得我们的应用jar自身就具备了提供Web服务的功能。
> 2. 父项目parent配置指定为spring-boot-starter-parent，该父项目中定义了Spring Boot的版本的基础依赖以及一些默认配置内容等，比如配置文件applicaiton.yml的位置等。
> 3. spring-boot-starter-web:全栈Web开发模块，包含嵌入式Tomcat、Spring MVC。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com</groupId>
    <artifactId>wechat</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
    </parent>
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outpuutEncoding>UTF-8</project.reporting.outpuutEncoding>
        <java.version>1.8</java.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```
## 3.配置文件  
#### 1. Spring Boot的默认配置文件位置为src/main/resources/application.yml
> 目前yml还有一些不足，它无法通过@PropertySource注解来加载配置。但是yml将属性加载到内存中保存的时候是有序的，所以当配置文件当中的信息需要具备顺序含义时，yml的配置方式比起properties配置文件更有优势。
#### 2. 自定义参数  
比如在配置文件中添加如下属性：
```yaml
book:
    name: SPringCloudAction
    author: Sucreyy
```
然后我们可以在应用中通过 ```@Value```注解来加载这些自定义的参数，比如：
```java
@Component
public class Book{
    
    @Value("${book.name}")
    private String name;
    
    @Value("${book.author}")
    private String author;
    
    //省略setter和getter
}
```
#### 3.参数引用
在application.yml中的各个参数之间可以直接通过使用PlaceHolder的方式来进行引用，就像下面的配置：
```yaml
book:
    name: SpringCloud
    author: Sucreyy
    desc: ${book.author} is writing 《${book.name}》
```
#### 4. 使用随机数
在一些特殊情况下,我们希望有些参数每次被加载的时候不是一个固定的值,比如密钥、服务端口等。在Spring Boot的属性配置文件中，可以通过使用${random}配置来产生随机的int、long、或者String字符串，这样我们就可以容易通过配置随机生成属性，而不是在程序中通过编码来实现这些逻辑。
```yaml
Sucre:
    #随机字符串
    value: ${random.value}
    #随机int
    number: ${random.int}
    #随机long
    bignumber: ${random.long}
    #10以内的随机数
    test1: ${random.int(10)}
    #10-20的随机数
    test2: ${random.int(10,20)}
```
#### 5.命令行参数
在命令行方式启动Spring Boot应用时，连续的两个减号--就是对application.proerties中的属性进行赋值的标识。所以，```java -jar xxx.jar --server.port=8888```命令，等价于在application.properties中添加属性server.port=8888
#### 6.多环境配置
> 暂时跳过
#### 7. 加载顺序
为了能更合理的重写各属性的值，Spring Boot使用了下面这种较为特别的属性加载顺序：  
1. **在命令行传入的参数。**
2. SPRING_APPLICATION_JSON中的属性。
3. java：comp/env中的JNDI属性。
4. Java的系统属性，可以通过System.getProperties()获得的内容。
5. 操作系统的环境变量。
6. 通过random.*配置的随机属性。
7. **位于当前应用jar包之外，针对不同{profile}环境的配置文件内容。**
8. 位于当前应用jar包之内，针对不同{profile}环境的配置文件内容。
9. **位于当前应用jar包之外的applicaiton.properties和YAML配置内容。**
10. 位于当前应用jar包之内的applicaiton.properties和YAML配置内容。
11. 在@Configuration注解修改的类中，通过@PropertySource注解定义的属性。
12. 应用默认属性，使用SpringApplication.setDefaultProperties定义的内容。

==优先级按上面的顺序由高到低，数字越小优先级越高。==  
可以看到7，9都是从应用jar包之外读取配置文件，所以，实现外部化配置的原理就是从此切入，为其指定外部配置文件的加载位置来取代jar包之内的配置内容。通过这样的实现，我们的工程在配置中就变得非常干净，只需要在本地放置开发需要的配置即可。