![](assets/Pasted%20image%2020250319103340.png)
每一个请求都是一个线程，为了防止出现多线程并发修改的问题，可以使用ThreadLocal，ThreadLocal会在每一个线程的内部创建一个map，这样可以保证每一个线程的请求来了之后都会有一个独立的存储空间，这样就可以保证各个线程之间相互没有干扰。
![](assets/Pasted%20image%2020250319104359.png)
## 1.发送手机验证码
![](assets/Pasted%20image%2020250319105133.png)
controller层
```java
/**  
 * 发送手机验证码  
 */  
@PostMapping("code")  
public Result sendCode(@RequestParam("phone") String phone, HttpSession session) {  
    // 发送短信验证码并保存验证码  
    return userService.sendCode(phone, session);  
}
```
service层
```java
public interface IUserService extends IService<User> {  
  
    Result sendCode(String phone, HttpSession session);  
}
```
serviceImpl层
```java
package com.hmdp.service.impl;  
  
import cn.hutool.core.util.RandomUtil;  
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;  
import com.hmdp.dto.Result;  
import com.hmdp.entity.User;  
import com.hmdp.mapper.UserMapper;  
import com.hmdp.service.IUserService;  
import com.hmdp.utils.RegexUtils;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.stereotype.Service;  
  
import javax.servlet.http.HttpSession;  
  
/**  
 * <p>  
 * 服务实现类  
 * </p>  
 */
@Slf4j  
@Service  
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {  
  
    // 发送短信验证码并保存验证码  
    public Result sendCode(String phone, HttpSession session) {  
        //1.校验手机号  
        if (RegexUtils.isPhoneInvalid(phone)) {  
            //2.如果不符合，返回错误信息  
            return Result.fail("手机号格式错误！");  
        }  
        //3.符合，生成验证码  
        String code = RandomUtil.randomNumbers(6);//hutool工具随机生成6位数字验证码  
  
        //4.保存验证码到session  
        session.setAttribute("code", code);  
  
        //5.发送验证码  
        //TODO 发送验证码 阿里云短信服务 也可以用容联云或者七牛云 或者邮箱服务  
        log.debug("发送验证码成功，验证码：{}", code);  
  
        //6.返回ok  
        return Result.ok();  
    }  
}
```
字符串校验工具类：RegexUtils
```java
package com.hmdp.utils;  
  
import cn.hutool.core.util.StrUtil;  
  
public class RegexUtils {  
    /**  
     * 是否是无效手机格式  
     * @param phone 要校验的手机号  
     * @return true:符合，false：不符合  
     */  
    public static boolean isPhoneInvalid(String phone){  
        return mismatch(phone, RegexPatterns.PHONE_REGEX);  
    }  
    /**  
     * 是否是无效邮箱格式  
     * @param email 要校验的邮箱  
     * @return true:符合，false：不符合  
     */  
    public static boolean isEmailInvalid(String email){  
        return mismatch(email, RegexPatterns.EMAIL_REGEX);  
    }  
  
    /**  
     * 是否是无效验证码格式  
     * @param code 要校验的验证码  
     * @return true:符合，false：不符合  
     */  
    public static boolean isCodeInvalid(String code){  
        return mismatch(code, RegexPatterns.VERIFY_CODE_REGEX);  
    }  
  
    // 校验是否不符合正则格式  
    private static boolean mismatch(String str, String regex){  
        if (StrUtil.isBlank(str)) {  
            return true;  
        }  
        return !str.matches(regex);  
    }  
}
```
## 2.登陆,注册模块
![](assets/Pasted%20image%2020250319114515.png)
controller层
```java
/**  
 * 登录功能  
 * @param loginForm 登录参数，包含手机号、验证码；或者手机号、密码  
 */  
@PostMapping("/login")  
public Result login(@RequestBody LoginFormDTO loginForm, HttpSession session){  
    // TODO 实现登录功能  
    return Result.fail("功能未完成");  
}
```
自定义常量类
```java
package com.hmdp.utils;  
  
public class SystemConstants {  
    public static final String IMAGE_UPLOAD_DIR = "D:\\lesson\\nginx-1.18.0\\html\\hmdp\\imgs\\";  
    public static final String USER_NICK_NAME_PREFIX = "user_";  
    public static final int DEFAULT_PAGE_SIZE = 5;  
    public static final int MAX_PAGE_SIZE = 10;  
}
```
service层
```java
Result login(LoginFormDTO loginForm, HttpSession session);
```
seriveImpl层
```java
/**  
 * 登录功能  
 *  
 * @param loginForm 登录参数，包含手机号、验证码；或者手机号、密码  
 * @return  
 */  
public Result login(LoginFormDTO loginForm, HttpSession session) {  
    //1.校验手机号  
    if (RegexUtils.isPhoneInvalid(loginForm.getPhone())) {  
        //2.如果不符合，返回错误信息  
        return Result.fail("手机号格式错误！");  
    }  
    //反向验证避免if嵌套过深  
    //2.校验验证码  
    Object cacheCode = session.getAttribute("code");//从session中取出验证码  
    String code = loginForm.getCode();//前端传递的验证码  
    if (cacheCode == null || !cacheCode.toString().equals(code)) {//如果session中没有验证码或者验证码不一致  
        //3.不一致，报错  
        return Result.fail("验证码错误");  
    }  
  
    //4.一致，根据手机号查询用户  
    User user = query().eq("phone", loginForm.getPhone()).one();  
    //5.判断用户是否存在  
    if (user == null) {  
        //6.不存在，保存用户到数据库  
        String phone = loginForm.getPhone();  
        user = createUserWithPhone(phone);  
    }  
    
  
    //7.保存用户到session  
    session.setAttribute("user", user);  
    return Result.ok(user);//不需要返回登录凭证，因为session的原理是cookie，，每一个session都有一个唯一的sessionId，在你访问Tomcat的时候，这个sessionId就已经自动保存到cookie中，下次再访问的时候，Tomcat会根据这个sessionId找到对应的session，从而实现自动登录。  
}  
  
/**  
 * 根据手机号创建用户  
 * @param phone  
 * @return  
 */  
private User createUserWithPhone(String phone) {  
    User user = new User();  
    user.setPhone(phone);  
    user.setNickName(USER_NICK_NAME_PREFIX + RandomUtil.randomString(10));  
    save(user);//使用mybatisplus保存用户到数据库  
    return user;  
}
```
用户登录成功之后不需要返回登录凭证，因为session的原理是cookie，每一个session都有一个唯一的sessionId，在你访问Tomcat的时候，这个sessionId就已经自动保存到cookie中，下次再访问的时候，Tomcat会根据这个sessionId找到对应的session，从而实现自动登录。
![](assets/Pasted%20image%2020250319114141.png)
登录测试：
![](assets/Pasted%20image%2020250319115127.png)

![](assets/Pasted%20image%2020250319114926.png)
## 3.登录验证功能
![](assets/Pasted%20image%2020250319154609.png)
![](assets/Pasted%20image%2020250319154624.png)
对所有请求都要单端存储一次，麻烦，有没有什么解决方案呢？同时还要满足线程的安全。
![](assets/Pasted%20image%2020250319160133.png)
![](assets/Pasted%20image%2020250319160422.png)
1.编写一个拦截器（实现一个拦截器要继承一个拦截器接口`HandlerInterceptor`）
```java
//ctrl+i 快速实现接口
/**  
 * 前置请求拦截器:拦截除login外的请求  
 * @param request  
 * @param response  
 * @param handler  
 * @return  
 * @throws Exception  
 */
 public boolean preHandle(HttpServletRequest request, HttpServletResponse response,  Object handler) throws Exception {  
    //1.获取session  
    HttpSession session = request.getSession();  
    //2.获取session中的用户  
    Object user = session.getAttribute("user");  
    //3.判断用户是否存在  
    if (user == null) {  
        //4.不存在，拦截  
        response.setStatus(401);//401代表未登录  
        return false;  
    }  
    //5.存在，保存用户信息到ThreadLocal  
    UserHolder.saveUser((UserDTO) user);  
    //6.存在，放行  
    return true;  
}
```
UserHolder类：
```java
package com.hmdp.utils;  
  
import com.hmdp.dto.UserDTO;  
  
public class UserHolder {  
    private static final ThreadLocal<UserDTO> tl = new ThreadLocal<>();  
  
    public static void saveUser(UserDTO user){  
        tl.set(user);  
    }  
  
    public static UserDTO getUser(){  
        return tl.get();  
    }  
  
    public static void removeUser(){  
        tl.remove();  
    }  
}
```
配置拦截器，使其生效(配置类继承WebMvcConfigurer)
```java
package com.hmdp.config;  
  
import com.hmdp.Interceptor.LoginInterceptor;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;  
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;  
  
@Configuration  
public class MvcConfig implements WebMvcConfigurer {  
  
    /**  
     * 添加拦截器  
     * @param registry  
     */  
    public void addInterceptors(InterceptorRegistry registry) {  
       registry.addInterceptor(new LoginInterceptor())  
                .excludePathPatterns(  
                        "/user/code",  
                        "/user/login",  
                        "/shop/**",  
                        "/shop-type/**",  
                        "/voucher/**",  
                        "/blog/hot",  
                        "/upload/**"  
                );  
    }  
}
```

controller层
```java
@GetMapping("/me")  
public Result me(){  
    UserDTO userDTO = UserHolder.getUser();
    return Result.ok(userDTO);  
}
```
    建议使用DTO而不是User的原因：
    1.这样可以减少存储容量，缓解服务器压力
    2.避免缓存敏感信息，更加安全
## 集群的session共享问题
session共享问题：多台Tomcat并不共享session存储空间，当请求切换到不同tomcat服务时导致数据丢失的问题。
![](assets/Pasted%20image%2020250319174248.png)
为什么不采用tomcat的session技术？
1.数据拷贝需要时间，相同数据放在多台服务器浪费存储空间。

2.session的替代方案应该满足：
•数据共享
•内存存储（热点数据要满足高并发）
•key、value结构
![](assets/Pasted%20image%2020250319175147.png)
## 基于Redis实现集群的session共享问题
#### 1、设计key的结构

首先我们要思考一下利用redis来存储数据，那么到底使用哪种结构呢？由于存入的数据比较简单，我们可以考虑使用String，或者是使用哈希，如下图，如果使用String，同学们注意他的value，用多占用一点空间，如果使用哈希，则他的value中只会存储他数据本身，如果不是特别在意内存，其实使用String就可以啦。



#### 2、设计key的具体细节

所以我们可以使用String结构，就是一个简单的key，value键值对的方式，但是关于key的处理，session他是每个用户都有自己的session，但是redis的key是共享的，咱们就不能使用code了

在设计这个key的时候，我们之前讲过需要满足两点

1、key要具有唯一性

2、key要方便携带

如果我们采用phone：手机号这个的数据来存储当然是可以的，但是如果把这样的敏感数据存储到redis中并且从页面中带过来毕竟不太合适，所以我们在后台生成一个随机串token，然后让前端带来这个token就能完成我们的整体逻辑了

#### 3、整体访问流程

当注册完成后，用户去登录会去校验用户提交的手机号和验证码，是否一致，如果一致，则根据手机号查询用户信息，不存在则新建，最后将用户数据保存到redis，并且生成token作为redis的key，当我们校验用户是否登录时，会去携带着token进行访问，从redis中取出token对应的value，判断是否存在这个数据，如果没有则拦截，如果存在则将其保存到threadLocal中，并且放行。





### 基于Redis实现短信登录
![](assets/Pasted%20image%2020250319175801.png)