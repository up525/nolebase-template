## 环境：

IntelliJ IDEA 2024.3.1.1 + Spring Boot + Maven

---

## 问题描述

使用`@AllArgsConstructor`注解一个用户类，然后调用全参构造方法创建对象，出现错误：

> java: 无法将类 com.itheima.pojo.User中的构造器 User应用到给定类型;   需要: 没有参数   找到:    java.lang.Integer,java.lang.String,java.lang.String,java.lang.String,java.lang.Integer,java.time.LocalDateTime   原因: 实际参数列表和形式参数列表长度不同

---

## 解决方案：

第一种方法：直接使用ptg插件自动生成空参构造，有参构造，get，set方法

第二种方法 ：不要在项目创建时引入，而是项目创建后直接在`pom.xml`以`<dependency>`的方式引入

第三种解决方法：给lombok注解加入**1.18.30**的版本号，并删除下方build中所有和lombok有关的配置。注意版本一定得是1.18.30，**并且不要忘记刷新maven仓库。**

```java
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.30</version>
        </dependency>
```



以上三个方法足以解决上述问题。 

​