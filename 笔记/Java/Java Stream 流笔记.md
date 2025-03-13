# Java Stream 流笔记

## 目录

1. [Stream 流是什么](#1-stream-流是什么)
2. [常用 API 及使用](#2-常用-api-及使用)
3. [与传统 API 对比](#3-与传统-api-对比)
4. [总结](#4-总结)

---

## 1. Stream 流是什么

**Stream** 是 Java 8 引入的用于处理集合数据的抽象 API，特点：

- **声明式编程**：关注"做什么"而非"怎么做"
- **链式操作**：支持流水线式操作组合
- **并行处理**：内置并行处理能力
- **不存储数据**：仅对数据源进行计算
- **不可复用**：每个流只能被消费一次

```java
// 示例：创建流
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();  // 顺序流
Stream<String> parallelStream = list.parallelStream();  // 并行流 
```

## 2. 常用 API 及使用

### 2.1 核心操作分类

| 操作类型 | 方法示例                      | 特点                     |
| -------- | ----------------------------- | ------------------------ |
| 中间操作 | filter(), map(), sorted()     | 惰性求值，返回新Stream   |
| 终端操作 | forEach(), collect(), count() | 触发实际计算，流不可复用 |

### 2.2 常用 API 示例

#### 过滤（filter）

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)  // 中间操作：过滤偶数
    .collect(Collectors.toList());  // 终端操作：收集结果
// 结果：[2, 4]    
```

#### 映射（map）

```java
List<String> words = Arrays.asList("Java", "Stream");
List<Integer> lengths = words.stream()
    .map(String::length)  // 中间操作：转换为字符串长度
    .collect(Collectors.toList());
// 结果：[4, 6]
```

#### 排序（sorted）

```java
List<String> names = Arrays.asList("Bob", "Alice", "David");
List<String> sortedNames = names.stream()
    .sorted()  // 自然排序
    .collect(Collectors.toList());
// 结果：[Alice, Bob, David]

// 自定义排序
List<String> customSorted = names.stream()
    .sorted((a, b) -> b.compareTo(a))  // 倒序
    .collect(Collectors.toList());
// 结果：[David, Bob, Alice]
```

------

## 3. 与传统 API 对比

### 3.1 排序功能对比

#### 集合工具类（Collections.sort）

```java
List<String> list = new ArrayList<>(Arrays.asList("c", "b", "a"));
Collections.sort(list);  // 直接修改原集合
// 优点：内存占用小
// 缺点：破坏原数据，难以组合复杂操作
```

#### Stream 实现排序

```java
List<String> original = Arrays.asList("c", "b", "a");
List<String> sorted = original.stream()
    .sorted()
    .collect(Collectors.toList());  // 生成新集合
// 优点：保持原数据不变，可组合其他操作
// 缺点：需要额外内存存储新集合  
```

### 3.2 综合对比

| 特性         | Stream API                 | 传统集合操作          |
| ------------ | -------------------------- | --------------------- |
| 代码可读性   | ⭐⭐⭐⭐（声明式）             | ⭐⭐（命令式）          |
| 并行处理     | 内置 parallel() 方法       | 需手动实现多线程      |
| 内存效率     | 较高内存占用（生成新集合） | 较低内存占用          |
| 操作组合能力 | 支持链式操作               | 需分步实现            |
| 数据修改     | 不修改源数据               | 可能修改源数据        |
| 适用场景     | 复杂数据处理流水线         | 简单操作/性能敏感场景 |

------

## 4. 总结

### 优势：

1. **声明式编程**：提高代码可读性和维护性
2. **函数式组合**：轻松实现复杂数据处理流水线
3. **并行优化**：简单启用 parallel() 即可利用多核
4. **延迟执行**：优化计算过程，避免不必要的操作

### 注意事项：

1. **性能敏感场景**：简单操作可能不如传统方式高效
2. **状态问题**：避免在 lambda 中修改外部状态
3. **资源管理**：注意流的自动关闭特性（如文件流）

### 选择建议：

- 需要组合多个操作时优先使用 Stream
- 处理大数据集时考虑使用并行流
- 简单单步操作可保留传统方式
- 需要保持原始数据不变时使用 Stream

```java
// 最佳实践示例：组合操作
List<String> result = dataList.stream()
    .filter(item -> item.startsWith("A"))  // 过滤
    .map(String::toUpperCase)             // 转换
    .sorted(Comparator.reverseOrder())    // 排序
    .limit(10)                            // 限制数量
    .collect(Collectors.toList());        // 收集结果
```