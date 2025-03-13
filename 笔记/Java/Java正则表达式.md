​
# 正则表达式

## 1.1 正则表达式的概念及演示

- 在Java中，我们经常需要验证一些字符串，例如：年龄必须是2位的数字、用户名必须是8位长度而且只能包含大小写字母、数字等。正则表达式就是用来验证各种字符串的规则。它内部描述了一些规则，我们可以验证用户输入的字符串是否匹配这个规则。
    
- 先看一个不使用正则表达式验证的例子：下面的程序让用户输入一个QQ号码，我们要验证：
    
    - QQ号码必须是5--15位长度
        
    - 而且必须全部是数字
        
    - 而且首位不能为0
        

```java
package com.itheima.a08regexdemo;
​
public class RegexDemo1 {
    public static void main(String[] args) {
        /* 假如现在要求校验一个qq号码是否正确。
            规则:6位及20位之内，日不能在开头，必须全部是数字。
            先使用目前所学知识完成校验需求然后体验一下正则表达式检验。
        */
​
        String qq ="1234567890";
        System.out.println(checkQQ(qq));
​
        System.out.println(qq.matches("[1-9]\\d{5,19}"));
​
    }
​
    public static boolean checkQQ(String qq) {
        //规则:6位及20位之内，日不能在开头，必须全部是数字 。
        //核心思想:
        //先把异常数据进行过滤
        //下面的就是满足要求的数据了。
        int len = qq.length();
        if (len < 6 || len > 20) {
            return false;
        }
        //0不能在开头
        if (qq.startsWith("0")) {
            return false;
        }
        //必须全部是数字
        for (int i = 0; i < qq.length(); i++) {
            char c = qq.charAt(i);
            if (c < '0' | c > '9') {
                return false;
            }
        }
        return true;
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

- 使用正则表达式验证：
    

```java
public class Demo {
    public static void main(String[] args) {
        String qq ="1234567890";
        System.out.println(qq.matches("[1-9]\\d{5,19}"));
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

**我们接下来就重点学习怎样写正则表达式**

## 1.2 正则表达式-字符类

> - 语法示例：
>     
> 
> 1. [abc]：代表a或者b，或者c字符中的一个。
>     
> 2. [^abc]：代表除a,b,c以外的任何字符。
>     
> 3. [a-z]：代表a-z的所有小写字符中的一个。
>     
> 4. [A-Z]：代表A-Z的所有大写字符中的一个。
>     
> 5. [0-9]：代表0-9之间的某一个数字字符。
>     
> 6. [a-zA-Z0-9]：代表a-z或者A-Z或者0-9之间的任意一个字符。
>     
> 7. [a-dm-p]：a 到 d 或 m 到 p之间的任意一个字符。
>     

- 代码示例：
    

```java
package com.itheima.a08regexdemo;
​
public class RegexDemo2 {
    public static void main(String[] args) {
        //public boolean matches(String regex):判断是否与正则表达式匹配，匹配返回true
        // 只能是a b c
        System.out.println("-----------1-------------");
        System.out.println("a".matches("[abc]")); // true
        System.out.println("z".matches("[abc]")); // false
​
        // 不能出现a b c
        System.out.println("-----------2-------------");
        System.out.println("a".matches("[^abc]")); // false
        System.out.println("z".matches("[^abc]")); // true
        System.out.println("zz".matches("[^abc]")); //false
        System.out.println("zz".matches("[^abc][^abc]")); //true
​
        // a到zA到Z(包括头尾的范围)
        System.out.println("-----------3-------------");
        System.out.println("a".matches("[a-zA-z]")); // true
        System.out.println("z".matches("[a-zA-z]")); // true
        System.out.println("aa".matches("[a-zA-z]"));//false
        System.out.println("zz".matches("[a-zA-Z]")); //false
        System.out.println("zz".matches("[a-zA-Z][a-zA-Z]")); //true
        System.out.println("0".matches("[a-zA-Z]"));//false
        System.out.println("0".matches("[a-zA-Z0-9]"));//true
​
​
        // [a-d[m-p]] a到d，或m到p
        System.out.println("-----------4-------------");
        System.out.println("a".matches("[a-d[m-p]]"));//true
        System.out.println("d".matches("[a-d[m-p]]")); //true
        System.out.println("m".matches("[a-d[m-p]]")); //true
        System.out.println("p".matches("[a-d[m-p]]")); //true
        System.out.println("e".matches("[a-d[m-p]]")); //false
        System.out.println("0".matches("[a-d[m-p]]")); //false
​
        // [a-z&&[def]] a-z和def的交集。为:d，e，f
        System.out.println("----------5------------");
        System.out.println("a".matches("[a-z&[def]]")); //false
        System.out.println("d".matches("[a-z&&[def]]")); //true
        System.out.println("0".matches("[a-z&&[def]]")); //false
​
        // [a-z&&[^bc]] a-z和非bc的交集。(等同于[ad-z])
        System.out.println("-----------6------------_");
        System.out.println("a".matches("[a-z&&[^bc]]"));//true
        System.out.println("b".matches("[a-z&&[^bc]]")); //false
        System.out.println("0".matches("[a-z&&[^bc]]")); //false
​
        // [a-z&&[^m-p]] a到z和除了m到p的交集。(等同于[a-1q-z])
        System.out.println("-----------7-------------");
        System.out.println("a".matches("[a-z&&[^m-p]]")); //true
        System.out.println("m".matches("[a-z&&[^m-p]]")); //false
        System.out.println("0".matches("[a-z&&[^m-p]]")); //false
​
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

​

## 1.3 正则表达式-逻辑运算符

- > 语法示例：
    > 
    > 1. &&：并且
    >     
    > 2. | ：或者
    >     
    > 3. \ ：转义字符
    >     
    
- 代码示例：
    

```java
public class Demo {
    public static void main(String[] args) {
        String str = "had";
        
        //1.要求字符串是小写辅音字符开头，后跟ad
        String regex = "[a-z&&[^aeiou]]ad";
        System.out.println("1." + str.matches(regex));
        
        //2.要求字符串是aeiou中的某个字符开头，后跟ad
        regex = "[a|e|i|o|u]ad";//这种写法相当于：regex = "[aeiou]ad";
        System.out.println("2." + str.matches(regex));
    }
}
​
package com.itheima.a08regexdemo;
​
public class RegexDemo3 {
    public static void main(String[] args) {
        // \ 转义字符 改变后面那个字符原本的含义
        //练习:以字符串的形式打印一个双引号
        //"在Java中表示字符串的开头或者结尾
​
        //此时\表示转义字符，改变了后面那个双引号原本的含义
        //把他变成了一个普普通通的双引号而已。
        System.out.println("\"");
​
        // \表示转义字符
        //两个\的理解方式：前面的\是一个转义字符，改变了后面\原本的含义，把他变成一个普普通通的\而已。
        System.out.println("c:Users\\moon\\IdeaProjects\\basic-code\\myapi\\src\\com\\itheima\\a08regexdemo\\RegexDemo1.java");
​
​
​
​
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

​

## 1.4 正则表达式-预定义字符

- > 语法示例：
    > 
    > 1. "." ： 匹配任何字符。
    >     
    > 2. "\d"：任何数字[0-9]的简写；
    >     
    > 3. "\D"：任何非数字[^0-9]的简写；
    >     
    > 4. "\s"： 空白字符：[ \t\n\x0B\f\r] 的简写
    >     
    > 5. "\S"： 非空白字符：[^\s] 的简写
    >     
    > 6. "\w"：单词字符：[a-zA-Z_0-9]的简写
    >     
    > 7. "\W"：非单词字符：[^\w]
    >     
    
- 代码示例：
    

```java
public class Demo {
    public static void main(String[] args) {
        //.表示任意一个字符
        System.out.println("你".matches("..")); //false
        System.out.println("你".matches(".")); //true
        System.out.println("你a".matches(".."));//true
​
        // \\d 表示任意的一个数字
        // \\d只能是任意的一位数字
        // 简单来记:两个\表示一个\
        System.out.println("a".matches("\\d")); // false
        System.out.println("3".matches("\\d")); // true
        System.out.println("333".matches("\\d")); // false
​
        //\\w只能是一位单词字符[a-zA-Z_0-9]
        System.out.println("z".matches("\\w")); // true
        System.out.println("2".matches("\\w")); // true
        System.out.println("21".matches("\\w")); // false
        System.out.println("你".matches("\\w"));//false
​
        // 非单词字符
        System.out.println("你".matches("\\W")); // true
        System.out.println("---------------------------------------------");
        // 以上正则匹配只能校验单个字符。
​
​
        // 必须是数字 字母 下划线 至少 6位
        System.out.println("2442fsfsf".matches("\\w{6,}"));//true
        System.out.println("244f".matches("\\w{6,}"));//false
​
        // 必须是数字和字符 必须是4位
        System.out.println("23dF".matches("[a-zA-Z0-9]{4}"));//true
        System.out.println("23 F".matches("[a-zA-Z0-9]{4}"));//false
        System.out.println("23dF".matches("[\\w&&[^_]]{4}"));//true
        System.out.println("23_F".matches("[\\w&&[^_]]{4}"));//false
        
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

## 1.5 正则表达式-数量词

- > 语法示例：
    > 
    > 1. X? : 0次或1次
    >     
    > 2. X* : 0次到多次
    >     
    > 3. X+ : 1次或多次
    >     
    > 4. X{n} : 恰好n次
    >     
    > 5. X{n,} : 至少n次
    >     
    > 6. X{n,m}: n到m次(n和m都是包含的)
    >     
    
- 代码示例：
    

```java
public class Demo {
    public static void main(String[] args) {
         // 必须是数字 字母 下划线 至少 6位
        System.out.println("2442fsfsf".matches("\\w{6,}"));//true
        System.out.println("244f".matches("\\w{6,}"));//false
​
        // 必须是数字和字符 必须是4位
        System.out.println("23dF".matches("[a-zA-Z0-9]{4}"));//true
        System.out.println("23 F".matches("[a-zA-Z0-9]{4}"));//false
        System.out.println("23dF".matches("[\\w&&[^_]]{4}"));//true
        System.out.println("23_F".matches("[\\w&&[^_]]{4}"));//false
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

​

## 1.6 正则表达式练习1

需求：

请编写正则表达式验证用户输入的手机号码是否满足要求。

请编写正则表达式验证用户输入的邮箱号是否满足要求。

请编写正则表达式验证用户输入的电话号码是否满足要求。

验证手机号码 13112345678 13712345667 13945679027 139456790271

验证座机电话号码 020-2324242 02122442 027-42424 0712-3242434

验证邮箱号码 [3232323@qq.com](mailto:3232323@qq.com) [zhangsan@itcast.cn](mailto:zhangsan@itcast.cn)n [dlei0009@163.com](mailto:dlei0009@163.com) [dlei0009@pci.com.cn](mailto:dlei0009@pci.com.cn)

代码示例：

```java
package com.itheima.a08regexdemo;
​
public class RegexDemo4 {
    public static void main(String[] args) {
        /*
            需求
            请编写正则表达式验证用户输入的手机号码是否满足要求。请编写正则表达式验证用户输入的邮箱号是否满足要求。请编写正则表达式验证用户输入的电话号码是否满足要求。
            验证手机号码 13112345678 13712345667 13945679027 139456790271
            验证座机电话号码 020-2324242 02122442 027-42424 0712-3242434
            验证邮箱号码 3232323@qq.com zhangsan@itcast.cnn dlei0009@163.com dlei0009@pci.com.cn
        */
​
        //心得:
        //拿着一个正确的数据，从左到右依次去写。
        //13112345678
        //分成三部分:
        //第一部分:1 表示手机号码只能以1开头
        //第二部分:[3-9] 表示手机号码第二位只能是3-9之间的
        //第三部分:\\d{9} 表示任意数字可以出现9次，也只能出现9次
        String regex1 = "1[3-9]\\d{9}";
        System.out.println("13112345678".matches(regex1));//true
        System.out.println("13712345667".matches(regex1));//true
        System.out.println("13945679027".matches(regex1));//true
        System.out.println("139456790271".matches(regex1));//false
        System.out.println("-----------------------------------");
​
        //座机电话号码
        //020-2324242 02122442 027-42424 0712-3242434
        //思路:
        //在书写座机号正则的时候需要把正确的数据分为三部分
        //一:区号@\\d{2,3}
        //      0:表示区号一定是以0开头的
        //      \\d{2,3}:表示区号从第二位开始可以是任意的数字，可以出现2到3次。
        //二:- ?表示次数，日次或一次
        //三:号码 号码的第一位也不能以日开头，从第二位开始可以是任意的数字，号码的总长度:5-10位
        String regex2 = "0\\d{2,3}-?[1-9]\\d{4,9}";
        System.out.println("020-2324242".matches(regex2));
        System.out.println("02122442".matches(regex2));
        System.out.println("027-42424".matches(regex2));
        System.out.println("0712-3242434".matches(regex2));
​
        //邮箱号码
        //3232323@qq.com zhangsan@itcast.cnn dlei0009@163.com dlei0009@pci.com.cn
        //思路:
        //在书写邮箱号码正则的时候需要把正确的数据分为三部分
        //第一部分:@的左边 \\w+
        //      任意的字母数字下划线，至少出现一次就可以了
        //第二部分:@ 只能出现一次
        //第三部分:
        //      3.1         .的左边[\\w&&[^_]]{2,6}
        //                  任意的字母加数字，总共出现2-6次(此时不能出现下划线)
        //      3.2         . \\.
        //      3.3         大写字母，小写字母都可以，只能出现2-3次[a-zA-Z]{2,3}
        //      我们可以把3.2和3.3看成一组，这一组可以出现1次或者两次
        String regex3 = "\\w+@[\\w&&[^_]]{2,6}(\\.[a-zA-Z]{2,3}){1,2}";
        System.out.println("3232323@qq.com".matches(regex3));
        System.out.println("zhangsan@itcast.cnn".matches(regex3));
        System.out.println("dlei0009@163.com".matches(regex3));
        System.out.println("dlei0009@pci.com.cn".matches(regex3));
​
​
        //24小时的正则表达式
        String regex4 = "([01]\\d|2[0-3]):[0-5]\\d:[0-5]\\d";
        System.out.println("23:11:11".matches(regex4));
​
        String regex5 = "([01]\\d 2[0-3])(:[0-5]\\d){2}";
        System.out.println("23:11:11".matches(regex5));
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

​

## 1.7 正则表达式练习2

需求 请编写正则表达式验证用户名是否满足要求。要求:大小写字母，数字，下划线一共4-16位 请编写正则表达式验证身份证号码是否满足要求。 简单要求: 18位，前17位任意数字，最后一位可以是数字可以是大写或小写的x 复杂要求: 按照身份证号码的格式严格要求。

身份证号码: 41080119930228457x 510801197609022309 15040119810705387X 130133197204039024 430102197606046442

代码示例：

```java
public class RegexDemo5 {
    public static void main(String[] args) {
        /*
            正则表达式练习:
            需求
            请编写正则表达式验证用户名是否满足要求。要求:大小写字母，数字，下划线一共4-16位
            请编写正则表达式验证身份证号码是否满足要求。
            简单要求:
                18位，前17位任意数字，最后一位可以是数字可以是大写或小写的x
            复杂要求:
                按照身份证号码的格式严格要求。
​
            身份证号码:
            41080119930228457x
            510801197609022309
            15040119810705387X
            130133197204039024 I
            430102197606046442
        */
​
        //用户名要求:大小写字母，数字，下划线一共4-16位
        String regex1 = "\\w{4,16}";
        System.out.println("zhangsan".matches(regex1));
        System.out.println("lisi".matches(regex1));
        System.out.println("wangwu".matches(regex1));
        System.out.println("$123".matches(regex1));
​
​
        //身份证号码的简单校验:
        //18位，前17位任意数字，最后一位可以是数字可以是大写或小写的x
        String regex2 = "[1-9]\\d{16}(\\d|x|x)";
        String regex3 = "[1-9]\\d{16}[\\dXx]";
        String regex5 = "[1-9]\\d{16}(\\d(?i)x)";
​
        System.out.println("41080119930228457x".matches(regex3));
        System.out.println("510801197609022309".matches(regex3));
        System.out.println("15040119810705387X".matches(regex3));
        System.out.println("130133197204039024".matches(regex3));
        System.out.println("430102197606046442".matches(regex3));
​
​
        //忽略大小写的书写方式
        //在匹配的时候忽略abc的大小写
        String regex4 = "a((?i)b)c";
        System.out.println("------------------------------");
        System.out.println("abc".matches(regex4));//true
        System.out.println("ABC".matches(regex4));//false
        System.out.println("aBc".matches(regex4));//true
​
​
        //身份证号码的严格校验
        //编写正则的小心得:
        //第一步:按照正确的数据进行拆分
        //第二步:找每一部分的规律，并编写正则表达式
        //第三步:把每一部分的正则拼接在一起，就是最终的结果
        //书写的时候:从左到右去书写。
​
        //410801 1993 02 28 457x
        //前面6位:省份，市区，派出所等信息，第一位不能是0，后面5位是任意数字       [1-9]\\d{5}
        //年的前半段: 18 19 20                                                (18|19|20)
        //年的后半段: 任意数字出现两次                                           \\d{2}
        //月份: 01~ 09 10 11 12                                               (@[1-9]|1[0-2])
        //日期: 01~09 10~19 20~29 30 31                                       (0[1-9]|[12]\\d|3[01])
        //后面四位: 任意数字出现3次 最后一位可以是数字也可以是大写x或者小写x        \\d{3}[\\dXx]
        String regex6 = "[1-9]\\d{5}(18|19|20)\\d{2}(@[1-9]|1[0-2])(@[1-9]|[12]\\d|3[01])\\d{3}[\\dxXx]";
​
        System.out.println("41080119930228457x".matches(regex6));
        System.out.println("510801197609022309".matches(regex6));
        System.out.println("15040119810705387X".matches(regex6));
        System.out.println("130133197204039024".matches(regex6));
        System.out.println("430102197606046442".matches(regex6));
​
​
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

# 总结

> - 语法示例：
>     
> 
> 1. [abc]：代表a或者b，或者c字符中的一个。
>     
> 2. [**^**abc]：代表除a,b,c**以外**的任何字符。
>     
> 3. [a**-**z]：代表**a-z**的所有小写字符中的一个。
>     
> 4. [A-Z]：代表A-Z的所有大写字符中的一个。
>     
> 5. [0-9]：代表0-9之间的某一个数字字符。
>     
> 6. [a-zA-Z0-9]：代表a-z或者A-Z或者0-9之间的任意一个字符。
>     
> 7. [a-dm-p]：a 到 d 或 m 到 p之间的任意一个字符。
>     
> 
> 语法示例：
> 
> 8. &&：并且
>     
> 2. | ：或者
>     
> 3. \ ：转义字符
>     
> 
> 语法示例：
> 
> 4. **"." ： 匹配任何字符。**
>     
> 2. **"\d"：任何数字[0-9]的简写；**
>     
> 3. "\D"：任何非数字[^0-9]的简写；
>     
> 4. "\s"： 空白字符：[ \t\n\x0B\f\r] 的简写
>     
> 5. "\S"： 非空白字符：[^\s] 的简写
>     
> 6. **"\w"：单词字符：[a-zA-Z_0-9]的简写**
>     
> 7. "\W"：非单词字符：[^\w]
>     
> 
> 语法示例：
> 
> 8. X? : 0次或1次
>     
> 2. X* : 0次到多次
>     
> 3. X+ : 1次或多次
>     
> 4. X{n} : 恰好n次
>     
> 5. **X{n,} : 至少n次**
>     
> 6. X{n,m}: n到m次(n和m都是包含的)
>     
> 
> tip：
> 
>          **1.正则语法忘记的时候可以直接在java文档里搜索string，找到match方法就可以找到正则的语法，或者直接搜索pattern**
> 
>         **2.一个[]表示一个字符，{}可以用来表示字符数量的范围**
> 
>         **3.Java中一个\\代表\,\u代表一个转义字符**

​

​