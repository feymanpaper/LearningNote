Spring框架：Java开发的行业标准

Spring全家桶：
Web：Spring Web MVC, Spring Web Flux
持久层：Spring Data/Spring Data JPA, Spring Data Redis, Spring Data MongoDB
安全校验: Spring Security
构建工程脚手架: Spring Boot
微服务: Spring Cloud

IoC是Spring全家桶各个功能模块的基础，创建对象的容器

AOP也是以IoC为基础，是面向切面编程,抽象化的面向对象
应用: 打印日志，事务，权限处理

## IoC
控制反转，将对象的创建进行反转，常规情况下，对象都是开发者手动创建的，使用IoC开发者不再需要创建对象，而是由IoC容器根据需求自动创建项目所需要的对象

不用IoC：所有对象开发者自己创建
使用IoC：对象不用开发者创建，而是交给Spring框架来完成

1. pom.xml导入
```java
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>5.3.15</version>
</dependency> 
```

基于XML和基于注解两种方式：

基于XML： 开发者把需要的对象在XML中进行配置，Spring框架读取这个配置文件，然后根据配置文件的内容来创建对象。（在XML配置bean，核心是XML解析加反射 ）

基于注解：
1、配置类
用一个Java类来替代XML文件, 把在XML中配置的内容放到配置类中
2、**扫包+注解（最简单的方式）**
不再需要依赖于XML或者配置类，而是直接将bean的创建交给目标类，在目标类添加注解`@Component`来创建 

## AOP
面向切面编程，是一种抽象化的面向对象编程，对面向对象编程的一种补充，把重复代码抽离出来，核心代码和非业务代码解耦合
打印日志例子：
底层是动态代理
1.创建切面类
2.实现类添加`@Component`注解 
3.配置自动扫包，开启自动生成代理对象
4.使用  
![[AOP原理.png]]
切面对象要和实现类保持一致性

## 零散知识点：
lombok引入之后，只需要在创建的类加上一个@Data注解，就可以自动生成get, set方法