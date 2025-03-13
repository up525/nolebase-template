​
# **函数式编程**

此“**函数**”类似于数学中的函数(强调做什么)，==**只要输入的数据一致返回的结果也是一致的**==

![](https://i-blog.csdnimg.cn/direct/fa3b4034d5c24c05821de317b470766a.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

![](https://i-blog.csdnimg.cn/direct/5a56909978934ad08210b640a8dd4b08.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

## **函数式编程解决了什么问题？**

使用Lambda函数替代某些匿名内部类对象，从而让程序**代码更简洁，可读性更好**。 

## **Lambda表达式**    **(形参)->{方法体}**

> JDK 8开始新增的一种语法形式，**它表示函数。**
> 
> 可以**用于替代某些匿名内部类对象，从而让程序更简洁，可读性更好。**

![](https://i-blog.csdnimg.cn/direct/be63beff33304ced870a6ca083114cfd.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​![](https://i-blog.csdnimg.cn/direct/2b5beb0c19a145769d14806c44638d45.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

##  什么是函数式接口？

> **有且仅有一个抽象方法的接口。**  
> 注意：将来我们见到的大部分函数式接口，上面都可能会有一个**@Functionallnterface**的注解，该注解用于约束当前接口必须是函数式接口。
> 
>             **必须是接口不能是类。且抽象方法有且只能有一个。**

## **例：使用****Lambda****简化****comparator****接口的匿名内部类**

根据年龄对学生类排序

![](https://i-blog.csdnimg.cn/direct/ecd9fddc416c4821a8f926781ad601f3.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

##  Lambda表达式的省略规则

作用：用于进一步简化Lambda表达式的写法。

### 具体规则

> **参数类型全部可以省略不写。**  
> **如果只有一个参数，参数类型省略的同时“0”也可以省略，但多个参数不能省略"()"**  
> 如果Lambda表达式中只有一行代码，大括号可以不写，同时要省略分号“;”如果这行代码是return语句，也必须去掉return。

# 小结

>  1、什么是**函数式编程**？有是好处？  
> **使用Lambda函数替代某些匿名内部类对象**，从而让程序代码更简洁，可读性更好。  
> 2、Lambda表达式是啥？有什么用？怎么写？  
> **JDK8新增的一种语法，代表函数；可以用于替代并简化函数式接口的匿名内部类。**  
> 3、什么样的接口是函数式接口？怎么确保一个接口必须是函数式接口？  
> **只有一个抽象方法**的接口就是函数式接口。  
> 在接口上加上**@Funcationallnterface**注解即可。

​