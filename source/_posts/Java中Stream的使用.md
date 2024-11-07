---
title: Java中Stream的使用
date: 2019-12-05 23:34:42
tags:
- Java
- Lambda
---
JDK1.8中进一步加强了lambda表达式，并提出一个集合流的抽象工具stream，在Stream接口里是这么描述的 
> A sequence of elements supporting sequential and parallel aggregate operations.   
支持顺序和并行聚合操作的一系列元素。

流是一个抽象的概念，我们只能基于一个集合（Collection）或数组生成流，之后我们可以进行遍历、过滤、排序、搜集等中间操作，听起来像是Iterator？操作流并不会改变源数据（⚠但是在调用对象内部方法时还是会改变该对象，不建议在流中这样做），并且操作是惰性的，只有在最后执行终止操作时一系列的中间操作才真正执行并输出最终结果。操作stream有以下三个步骤：

---
## 创建stream

流需要数组或集合来创建
```java
//List
List<Student> students = new ArrayList<>();
Stream<Student> stream = students.stream();//顺序流
Stream<Student> stream1 = students.parallelStream();//并行流
//数组
String[] strings = {"a","b","c"};
Stream<String> stream2 = Stream.of(strings);
```
## 操作stream

这里只列举下常规的操作😃
- 排序（sorted）：可以对实现了Comparable的对象直接排序，也可以自定义比较器Comparator来排序
```java
List<Student> list;
list = students.stream().sorted(getComparator()).collect(Collectors.toList());
//对于已实现Comparable的类型(如String)，可以直接使用该字段排序,这个是按照name来倒排
list = students.stream().sorted(Comparator.comparing(Student::getName).reversed()).collect(Collectors.toList());
```
- 过滤（filter）：筛选出那些符合条件的数据
```java
//选择出male为false的数据
list = students.stream().filter(Student->Student.isMale()==false).collect(Collectors.toList());
//操作并不仅限于简单语句，使用{}包裹你的逻辑，最后return出去即可，不过lambda表达式为人诟病的一点是出了问题不好排查，所以这里面的逻辑还是尽量简洁吧
list = students.stream().filter(Student -> {
    //这里可以有很多逻辑
    //……
    return Student.isMale() == false;
    }).collect(Collectors.toList());
```
- 遍历（map）：遍历操作数据
```java
//注意：map之后新生成的Stream只包含转换生成的元素
//所以以下代码是错误的
list = students.stream().map(Student -> Student.getName().toLowerCase()).collect(Collectors.toList());
//它只操作了name，那么在之后的stream中就只能针对name进行操作了，若不转换结果就是 List<String>
List<String> stringList = students.stream().map(Student -> Student.getName().toLowerCase()).collect(Collectors.toList());
```
- distinct（去重）
- limit（限定前n个数据）
- skip（限定跳过前n个数据）
## 终止stream
上面的例子都用了collect来终止stream，因为如果我们不调用collect就无法把结果转化为list，所有的中间操作返回的都是抽象的Stream。实际上还有其它很多终止方法，这里也列举些比较常见的😁
- 汇聚（collect），这个后续可以展开来说
```java
//Collectors.toList()这个是指定结果集为List
//List转Map，toMap接收两个参数分别为Map的key、value，但是注意这里不允许有相同的key，当有相同的key时会抛一个 IllegalStateException: Duplicate key 的异常
Map<String,String> map = students.stream().collect(Collectors.toMap(Student::getStudentNo,Student::getName));
```
- 遍历（foreach）
```java
//一个简单的打印
students.stream().forEach(System.out::println);
```
- allMatch：每个元素都符合
- noneMatch：每个元素都不符合
- anyMatch：任一元素符合
- findFirst：返回流中第一个元素
- findAny：返回流中的任意元素
- count：返回流中元素的总个数
- max：返回流中元素最大值
- min：返回流中元素最小值
- **reduce**：



