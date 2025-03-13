# **什么是异常？**

## **先来看两个例子:**

###         1.数组索引越界异常

> ![](https://i-blog.csdnimg.cn/direct/5dfeec9e83fd438fa7b47a392c46d09d.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​
> 
> ![](https://i-blog.csdnimg.cn/direct/a16ce97419b345db8bcc01ce6c1b0972.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

###          2.除0异常

> ![](https://i-blog.csdnimg.cn/direct/b3280eb7c6c24e25bfbb1618ca00f576.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​
> 
> ![](https://i-blog.csdnimg.cn/direct/096c0da8b3344c37a106dc28f5bef2df.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

# **Java****的异常体系** 

![](https://i-blog.csdnimg.cn/direct/4a9c51c9eb6441098f4ec80937c40a6b.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

##  **Error**：

> **Error****：**代表的系统级别错误(属于严重问题)，也就是说系统一旦出现问题，sun公司会把这些问题封装成Error对象给出来
> 
> ( 说白了，Error是给sun公司自己用的，不是给我们程序员用的，因此我们开发人员不用管它)

## **Exception** ：

> **Exception**：叫异常，它代表的才是我们程序可能出现的问题，所以，我们程序员通常会用Exception以及它的孩子来封装程序出现的问题。
> 
> **运行时异常：**RuntimeException及其子类，编译阶段不会出现错误提醒，运行时出现的异常（如：数组索引越界异常）
> 
> **编译时异常**：编译阶段就会出现错误提醒的。（如：日期解析异常）

# **异常的基本处理** 

## **抛出异常（throws**）

在方法上使用throws关键字，可以将方法内部出现的异常抛出去给调用者处理。

![](https://i-blog.csdnimg.cn/direct/5eefd512809c44fda273af31dc36bc55.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

##  **捕获异常**(try…catch)

直接捕获程序出现的异常。

![](https://i-blog.csdnimg.cn/direct/086718605f0f4bd783824624f8072619.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

# **异常的作用？** 

![](https://i-blog.csdnimg.cn/direct/9a76ed1f1a394bcfadee902695df0aa5.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

# 小结

> 1.异常是什么？  
> 异常是代码在编译或者执行的过程中可能出现的错误。  
> 2.异常的代表是谁？分为几类？  
> **Exception，分为两类：编译时异常、运行时异常。**  
> 编译时异常：没有继承RuntimeExcpetion的异常，**编译阶段就会出错。**  
> 运行时异常：继承自RuntimeException的异常或其子类，**编译阶段不报错，运行时出现  
> 的。**  
> 3．异常的作用是啥？  
> 用来查找bug：可以作为方法内部的特殊返回值，通知上层调用者底层的执行情况。

---

#   异常进阶

## **自定义异常** （重点）

> **Java无法为这个世界上全部的问题都提供异常类来代表， 如果企业自己的某种问题，想通过异常来表示，以便用异常来管理该问题，那就需要自己来定义异常类了。**

 ![](https://i-blog.csdnimg.cn/direct/770d6cec06604109892dc9a57a6255bf.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

###   **自定义运行时异常（用的多）**

> 1.定义一个异常类继承RuntimeException.  
> 2.重写构造器。  
> 3.通过throw new异常类（xxx）来创建异常对象并抛出  
> 特点：编译阶段不报错，运行时才可能出现！提醒不属于激进型。

例如：

```java
/**
 * 业务异常
 */
public class BaseException extends RuntimeException {

    public BaseException() {
    }

    public BaseException(String msg) {
        super(msg);
    }

}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

在企业级项目中，你还可以在此之上你还可以在定义其他的异常 

```java
/**
 * 密码错误异常
 */
public class PasswordErrorException extends BaseException {

    public PasswordErrorException() {
    }

    public PasswordErrorException(String msg) {
        super(msg);
    }

}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

###  自定义编译时异常（用的少）

> 定义一个异常类继承Exception.  
> 重写构造器。  
> 通过throw new 异常类（xxx）创建异常对象并抛出。  
> **特点：编译阶段就报错，提醒比较激进****（应尽量减少使用）**

## **异常的处理方案**

![](https://i-blog.csdnimg.cn/direct/0c42ec513fbc45aa889719457800e36b.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​![](https://i-blog.csdnimg.cn/direct/bcc260ecc9314939a694bcccdfd612ca.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

**异常的两种处理方式**

### **抛出异常（****throws****）**

在方法上使用throws关键字，可以将方法内部出现的异常抛出去给调用者处理。

![](https://i-blog.csdnimg.cn/direct/c382f16c86244a7d9a1599b6b18af993.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​![](https://i-blog.csdnimg.cn/direct/90a9bd0dc44e41e4914c02a119b49c27.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

###  **捕获异常****(try…catch)**

直接捕获程序出现的异常。

![](https://i-blog.csdnimg.cn/direct/341e0aa0421940b587bcaa725e75f683.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​![](https://i-blog.csdnimg.cn/direct/489d5700d2854662aab1d2083ffd24b3.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

## 小结 

> 1、异常处理的常见方案是什么样的？
> 
> **在开发中异常的常见处理方式是：底层的异常抛出去给最外层，最外层集中捕获处理。**

---

#  spring boot全局异常拦截器

## 1 什么是全局异常处理器

> 软件开发springboot项目过程中，不可避免的需要处理各种异常，spring mvc架构中各层会出现大量的try{…} catch{…} finally{…}代码块，不仅有大量的**冗余代码**，而且还影响代码的**可读性。这样就需要定义个全局统一异常处理器，以便业务层再也不必处理异常。**
> 
> Spring在3.2版本增加了一个注解@[ControllerAdvice](https://so.csdn.net/so/search?q=ControllerAdvice&spm=1001.2101.3001.7020 "ControllerAdvice")，可以与@ExceptionHandler、@InitBinder、@ModelAtribute等注解配套使用。不过跟异常处理相关的只有注解@ExceptionHandler,从字面上看，就是**异常处理器**的意思。

## 2 为什么需要全局异常

> - 不用强制写try-catch，由全局异常处理器统一捕获处理。
> - 自定义异常，只能用全局异常来捕获。不能直接返回给客户端，客户端是看不懂的，需要接入全局异常处理器
> - JSR303规范的Validator参数校验器，参数校验不通过会抛异常，是无法使用try-catch语句直接捕获，只能使用全局异常处理器。 

##  **@RestControllerAdvice是什么**

> **@RestControllerAdvice是Spring框架提供的一个注解，用于定义全局异常处理器和全局数据绑定设置。****它结合了@ControllerAdvice和@ResponseBody两个注解的功能。**
> 
> @ControllerAdvice  
> @ControllerAdvice是一个用于定义全局控制器增强（即全局异常处理和全局数据绑定）的注解。**通过使用@ControllerAdvice，我们可以将异常处理和数据绑定逻辑集中到一个类中，避免在每个控制器中重复编写相同的异常处理代码。**
> 
> @ResponseBody  
> @ResponseBody是用于指示控制器方法返回的对象将被直接写入响应体中的注解。**它告诉Spring将方法的返回值序列化为JSON或其他适当的响应格式**，并将其作为HTTP响应的主体返回给客户端。

## **@RestControllerAdvice作用**

> 当我们在类上使用@RestControllerAdvice注解时，它**相当于同时使用了@ControllerAdvice和@ResponseBody。**这意味着被@RestControllerAdvice注解标记的类将被视为全局异常处理器，并且异常处理方法的返回值将以JSON格式直接写入响应体中。
> 
> 通过在@RestControllerAdvice类中定义异常处理方法，我们可以捕获和处理控制器中抛出的异常，提供自定义的异常处理逻辑，以及返回适当的响应给客户端。这样可以统一处理应用程序中的异常情况，提高代码的可维护性和可读性。

## @ExceptionHandler 

> 用法：  
> @ExceptionHandler 注解可以**用于方法**级别，**用于标记一个方法为异常处理方法。**  
> 异常处理方法需要定义在控制器类中，并且可以有任意的访问修饰符。  
> **总的来说，@ExceptionHandler 注解提供了一种在控制器中处理异常的机制，能够根据不同类型的异常来执行不同的异常处理逻辑，使代码更加清晰和易于维护。**

## 具体用法 

```java
/**
 * 全局异常处理器，处理项目中抛出的业务异常
 */
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    /**
     * 捕获业务异常
     * @param ex
     * @return
     */
    @ExceptionHandler
    public Result exceptionHandler(BaseException ex){
        log.error("异常信息：{}", ex.getMessage());
        return Result.error(ex.getMessage());
    }

    /**
     * 处理SQL异常
     * @param ex
     * @return
     */
    @ExceptionHandler
    public Result exceptionHandler(SQLIntegrityConstraintViolationException ex){
        //Duplicate entry 'zhangsan' for key 'employee.idx_username'
        String message = ex.getMessage();
        if(message.contains("Duplicate entry")){
            String[] split = message.split(" ");
            String username = split[2];
            String msg = username + MessageConstant.ALREADY_EXISTS;
            return Result.error(msg);
        }else{
            return Result.error(MessageConstant.UNKNOWN_ERROR);
        }
    }

}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

#  小结

> **1. @RestControllerAdvice是什么？**
> 
> **@RestControllerAdvice是Spring框架提供的一个注解，用于定义全局异常处理器和全局数据绑定设置。****它结合了@ControllerAdvice和@ResponseBody两个注解的功能。**
> 
> 2.@ExceptionHandler ？
> 
> @ExceptionHandler 注解可以**用于方法**级别，**用于标记一个方法为异常处理方法。**  
> 异常处理方法需要定义在控制器类中，并且可以有任意的访问修饰符。  
> **总的来说，@ExceptionHandler 注解提供了一种在控制器中处理异常的机制，能够根据不同类型的异常来执行不同的异常处理逻辑，使代码更加清晰和易于维护。**

​