---
title: Java算法向
date:  2021-08-31 12:09:33 +0800
category: Original
tags: Java
excerpt: 记录着使用Java做算法题的基本操作
---

* content
{:toc}

#### —. 在java中的基本类包

import java.io.*;//BufferedInputStream

import java.util.*; //输入Scanner

import java.math.*; //BigInteger && BigDecimal

#### 二. 输入与输出

读入: Scanner cin = new Scanner (System.in);

推荐：Scanner cin = new Scanner (new BufferedInputStream (System.in));

Scanner cin = new Scanner(System.in);

类名  对象名      构造函数    参数

Scanner类提供了非常丰富的成员函数来负责读取各种数据类型:

| 返回值                      | 成员函数                                                     |
| --------------------------- | ------------------------------------------------------------ |
| boolean hasNext()           | 相当于C++的!=EOF                                             |
| String next(String pattern) | 如果下一个标记与从指定字符串构造的模式匹配，则返回下一个标记,如果参数为空,就是读取一个字符串 |
| BigDecimal nextBigDecimal() | 将输入信息的下一个标记扫描为一个 BigDecimal                  |
| BigInteger nextBigInteger() | 将输入信息的下一个标记扫描为一个 BigInteger                  |
| boolean nextBoolean()       | 扫描解释为一个布尔值的输入标记并返回该值。                   |
| byte nextByte()             | 将输入信息的下一个标记扫描为一个 byte。                      |
| double nextDouble()         | 将输入信息的下一个标记扫描为一个 double                      |
| float nextFloat()           | 将输入信息的下一个标记扫描为一个 float                       |
| int nextInt()               | 将输入信息的下一个标记扫描为一个 int                         |
| String nextLine()           | 此扫描器执行当前行，并返回跳过的输入信息                     |
| long nextLong()             | 将输入信息的下一个标记扫描为一个 long                        |
| short nextShort()           | 将输入信息的下一个标记扫描为一个 short                       |

**对于输出浮点数保留几位小数的问题，可以使用DecimalFormat类**，

import java.text.*;

DecimalFormat f = new DecimalFormat("#.00#");

DecimalFormat g = new DecimalFormat("0.000");

double a = 123.45678, b = 0.12;

System.out.println(f.format(a)); //123.457

System.out.println(f.format(b)); //.12

System.out.println(g.format(b)); //0.120

System.out.print(); // cout << …;

System.out.println(); //与C++的cout << … <<endl;

System.out.printf(); //与C中的printf用法类似.

#### 三. 定义变量

定义单个变量：

int a,b,c;      //和C++ 中无区别

BigInteger a;   //定义大数变量a

BigInteger b = BigInteger.valueOf(9);   //定义大数变量 b赋值为 9；

BigDecimal n；     //定义大浮点数类 n；

```
//BigInteger支持radix36以内的进制转换
BigInteger integer = BigInteger.valueOf(9);
System.out.println(integer.toString(2));
//输出1001
```

boolean : 布尔值，仅有两个值，true和false.

byte :字节类型值，长度8位（一个字节），范围-128至127。

short：短整型值，长度16位（两个字节），范围-32768至32767。

int：整型值，长度32位（四个字节），范围-2147483648至2147483647

long：长整型，长度64位（八个字节），范围-9223372036854775808至9223372036854775807

float：单精度浮点数，长度32位（四个字节）。

double：双精度浮点数，长度64位（八个字节）。

char：字符型，长度16位，支持所有的UCS-2和ASCII编码。

除了以上的8种基本数据类型,对于ACMer还有BigInteger,BigDecimal,String三个类经常使用.

#### 四.写法

import java.math.*;

import java.io.*;

import java.util.*;

public class Main {

​    public static void main(String[] args) {

​    }

}

#### 五.注意事项

1. Java 是面向对象的语言，思考方法需要变换一下，里面的函数统称为方法，不要搞错。
2. Java 里的数组有些变动，多维数组的内部其实都是指针，所以**Java不支持fill多维数组**。数组定义后必须初始化，如 int[] a = new int[100];
3. 布尔类型为 boolean，只有true和false二值，在 if (...) / while (...) 等语句的条件中必须为boolean类型。 **在C/C++中的 if (n % 2) ... 在Java中无法编译通过。**
4. 下面在java.util包里Arrays类的几个方法可替代C/C++里的**memset**、**qsort/sort** 和 bsearch:

**Arrays.fill()**

**Arrays.sort()**

Arrays.binarySearch()

#### 六.变量的初始化值

只有成员变量才有默认值，而局部变量必须要赋初值，为什么非怎么设置？下面我们来看一下。

| 类型           | 值                               |
| -------------- | -------------------------------- |
| Int            | 0                                |
| Long           | 0                                |
| Boolean        | false                            |
| float          | 0.0                              |
| double         | 0.0                              |
| char           | /u000(NULL)                      |
| String         | NULL                             |
| Object         | NULL                             |
| 数组(未初始化) | NULL                             |
| 数组(初始化)   | 数组各个元素的值，其类型的默认值 |

#### 七.Collection集合接口方法说明

| 方法名                                    | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| boolean add(E e)                          | 向集合添加元素e，若指定集合元素改变了则返回true              |
| boolean addAll(Collection<? extends E> c) | 把集合C中的元素全部添加到集合中，若指定集合元素改变返回true  |
| void clear()                              | 清空所有集合元素                                             |
| boolean contains(Object o)                | 判断指定集合是否包含对象o                                    |
| boolean containsAll(Collection<?> c)      | 判断指定集合是否包含集合c的所有元素                          |
| boolean isEmpty()                         | 判断指定集合的元素size是否为0                                |
| boolean remove(Object o)                  | 删除集合中的元素对象o,若集合有多个o元素，则只会删除第一个元素 |
| boolean removeAll(Collection<?> c)        | 删除指定集合包含集合c的元素                                  |
| boolean retainAll(Collection<?> c)        | 从指定集合中保留包含集合c的元素,其他元素则删除               |
| int size()                                | 集合的元素个数                                               |
| T[] toArray(T[] a)                        | 将集合转换为T类型的数组                                      |

#### 八.String的常用方法

```
//char[]和String相互转换
String str = "ggg";
char[] bm;
bm = str.toCharArray();//String->char[]
str = String.valueOf(bm);//char[]->String
```

| 方法名                                         | 说明                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| char charAt(int index)                         | 返回 char指定索引处的值                                      |
| int compareTo(String anotherString)            | 按字典顺序比较两个字符串                                     |
| int compareToIgnoreCase(String str)            | 按字典顺序比较两个字符串，忽略大小写                         |
| String concat(String str)                      | 将指定的字符串连接到该字符串的末尾                           |
| boolean contains(CharSequence s)               | 当且仅当此字符串包含指定的char值序列时才返回true             |
| boolean endsWith(String suffix)                | 测试此字符串是否以指定的后缀结尾                             |
| boolean equals(Object anObject)                | 将此字符串与指定对象进行比较                                 |
| boolean equalsIgnoreCase(String anotherString) | 将此 String与其他 String比较，忽略大小写                     |
| boolean isEmpty()                              | 返回 true如果，且仅当 length()为 0                           |
| int length()                                   | 返回此字符串的长度                                           |
| boolean matches(String regex)                  | 告诉这个字符串是否匹配给定的 regular expression              |
| String replace(char oldChar, char newChar)     | 返回从替换所有出现的导致一个字符串 oldChar在此字符串 newChar |
| String substring(int beginIndex, int endIndex) | 返回一个字符串，该字符串是此字符串的子字符串                 |
| char[] toCharArray()                           | 将此字符串转换为新的字符数组                                 |
| static String valueOf(char[] data)             | 返回 char数组参数的字符串 char形式                           |

#### 九、数组

**System.arraycopy应用于自制数组ArrayList的remove删除元素**

System.arraycopy(elements, index + 1, elements, index, moveSize);

```
public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
代码解释:
　　Object src : 原数组
   int srcPos : 从元数据的起始位置开始
　　Object dest : 目标数组
　　int destPos : 目标数组的开始起始位置
　　int length  : 要copy的数组的长度
```

------

**Arrays.copyOf() 应用于自制数组ArrayList的add扩容**

Arrays的copyOf()方法传回的数组是**新的数组对象**，改变传回数组中的元素值，不会影响原来的数组。
copyOf()的第二个自变量指定要建立的**新数组长度**，如果新数组的长度**超过**原数组的长度，则**保留**数组默认值，例如：

```
import java.util.Arrays;

public class ArrayDemo {
 public static void main(String[] args) {
     int[] arr1 = {1, 2, 3, 4, 5}; 
     int[] arr2 = Arrays.copyOf(arr1, 5);
     int[] arr3 = Arrays.copyOf(arr1, 10);
     for(int i = 0; i < arr2.length; i++) 
         System.out.print(arr2[i] + " "); 
      System.out.println();
     for(int i = 0; i < arr3.length; i++) 
         System.out.print(arr3[i] + " ");
 }
}
```

运行结果：

```
1 2 3 4 5 
1 2 3 4 5 0 0 0 0 0
```

------

**List, Integer[], int[]的相互转换**

```
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
 
public class Main {
    public static void main(String[] args) {
        int[] data = {4, 5, 3, 6, 2, 5, 1};
 
        // int[] 转 List<Integer>
        List<Integer> list1 = Arrays.stream(data).boxed().collect(Collectors.toList());
        // Arrays.stream(arr) 可以替换成IntStream.of(arr)。
        // 1.使用Arrays.stream将int[]转换成IntStream。
        // 2.使用IntStream中的boxed()装箱。将IntStream转换成Stream<Integer>。
        // 3.使用Stream的collect()，将Stream<T>转换成List<T>，因此正是List<Integer>。
 
        // int[] 转 Integer[]
        Integer[] integers1 = Arrays.stream(data).boxed().toArray(Integer[]::new);
        // 前两步同上，此时是Stream<Integer>。
        // 然后使用Stream的toArray，传入IntFunction<A[]> generator。
        // 这样就可以返回Integer数组。
        // 不然默认是Object[]。
 
        // List<Integer> 转 Integer[]
        Integer[] integers2 = list1.toArray(new Integer[0]);
        //  调用toArray。传入参数T[] a。这种用法是目前推荐的。
        // List<String>转String[]也同理。
 
        // List<Integer> 转 int[]
        int[] arr1 = list1.stream().mapToInt(Integer::valueOf).toArray();
        // 想要转换成int[]类型，就得先转成IntStream。
        // 这里就通过mapToInt()把Stream<Integer>调用Integer::valueOf来转成IntStream
        // 而IntStream中默认toArray()转成int[]。
 
        // Integer[] 转 int[]
        int[] arr2 = Arrays.stream(integers1).mapToInt(Integer::valueOf).toArray();
        // 思路同上。先将Integer[]转成Stream<Integer>，再转成IntStream。
 
        // Integer[] 转 List<Integer>
        List<Integer> list2 = Arrays.asList(integers1);
        // 最简单的方式。String[]转List<String>也同理。
 
        // 同理
        String[] strings1 = {"a", "b", "c"};
        // String[] 转 List<String>
        List<String> list3 = Arrays.asList(strings1);
        // List<String> 转 String[]
        String[] strings2 = list3.toArray(new String[0]);
 
    }
}
```

#### 十、进制

二进制转十进制

```
int a = Integer.valueOf(in.next(),2);
```

十进制转二进制

`BigInteger` -> `import java.text.*;`

```
BigInteger b = BigInteger.valueOf(in.nextLong());
String bin = b.toString(2);
```

#### 十一、日期与时间

##### 导包

```
import java.time.*;
import java.time.format.*;
```

##### DateTimeFormatter

```
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String today = dateTimeFormatter.format(LocalDateTime.now());
```

##### LocalDateTime

LocalDateTime 修改 Day、Hour、Minute

LocalDate 修改 Day、Month、Year

```
LocalDateTime localDateTime = LocalDateTime.now();//2020-03-11T01:48:35.109
LocalDateTime today_morning = localDateTime.withHour(9);//2020-03-11T09:48:35.109
LocalDateTime localDate = localDate.parse("1998-12-20",dateTimeFormatter).atStartOfDay();
```

##### Duration

```
Duration duration = Duration.between(localDate,localDateTime);//后减前
long days = duration.toDays();
```

#### 十二、StreamAPI

##### 分组，计数和排序

0导包

```
import java.util.function.Function;
import java.util.stream.Collectors;
```

1.1分组`List`并显示其总数。

Java8Example1.java

```java
public class Java8Example1 {
    public static void main(String[] args) {
        //3 apple, 2 banana, others 1
        List<String> items =
                Arrays.asList("apple", "apple", "banana",
                        "apple", "orange", "banana", "papaya");
        Map<String, Long> result =
                items.stream().collect(
                        Collectors.groupingBy(
                                Function.identity(), Collectors.counting()
                        )
                );
        System.out.println(result);
    }
}
```

Output

```
{
 番木瓜= 1，橙= 1，香蕉= 2，苹果= 3
}
```

1.2添加排序。

Java8Example2.java

```java
public class Java8Example2 {
    public static void main(String[] args) {
        //3 apple, 2 banana, others 1
        List<String> items =
                Arrays.asList("apple", "apple", "banana",
                        "apple", "orange", "banana", "papaya");
        Map<String, Long> result =
                items.stream().collect(
                        Collectors.groupingBy(
                                Function.identity(), Collectors.counting()
                        )
                );
        Map<String, Long> finalMap = new LinkedHashMap<>();
        //Sort a map and add to finalMap
        result.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue()
                        .reversed()).forEachOrdered(e -> finalMap.put(e.getKey(), e.getValue()));
        System.out.println(finalMap);
    }
}
```

Output

```
{
 苹果= 3，香蕉= 2，木瓜= 1，橙= 1
}
```

“分组”用户定义的对象列表的示例。

2.1 Pojo。

Item.java

```java
public class Item {
    private String name;
    private int qty;
    private BigDecimal price;
    //constructors, getter/setters
}
```

2.2 按姓名+数字或数量组合。

Java8Examples3.java

```java
public class Java8Examples3 {
    public static void main(String[] args) {
        //3 apple, 2 banana, others 1
        List<Item> items = Arrays.asList(
                new Item("apple", 10, new BigDecimal("9.99")),
                new Item("banana", 20, new BigDecimal("19.99")),
                new Item("orang", 10, new BigDecimal("29.99")),
                new Item("watermelon", 10, new BigDecimal("29.99")),
                new Item("papaya", 20, new BigDecimal("9.99")),
                new Item("apple", 10, new BigDecimal("9.99")),
                new Item("banana", 10, new BigDecimal("19.99")),
                new Item("apple", 20, new BigDecimal("9.99"))
        );
        Map<String, Long> counting = items.stream().collect(
                Collectors.groupingBy(Item::getName, Collectors.counting()));
        System.out.println(counting);
        Map<String, Integer> sum = items.stream().collect(
                Collectors.groupingBy(Item::getName, Collectors.summingInt(Item::getQty)));
        System.out.println(sum);
    }
}
```

Output

```
// Group by + Count
{
 番木瓜= 1，香蕉= 2，苹果= 3，猩猩= 1，西瓜= 1
}

// Group by + Sum qty
{
 番木瓜= 20，香蕉= 30，苹果= 40，orang = 10，西瓜= 10
}
```

2.2按价格分组 - `Collectors.groupingBy`以`Collectors.mapping`示例为例。

Java8Examples4.java

```java
public class Java8Examples4 {
    public static void main(String[] args) {
        //3 apple, 2 banana, others 1
        List<Item> items = Arrays.asList(
                new Item("apple", 10, new BigDecimal("9.99")),
                new Item("banana", 20, new BigDecimal("19.99")),
                new Item("orang", 10, new BigDecimal("29.99")),
                new Item("watermelon", 10, new BigDecimal("29.99")),
                new Item("papaya", 20, new BigDecimal("9.99")),
                new Item("apple", 10, new BigDecimal("9.99")),
                new Item("banana", 10, new BigDecimal("19.99")),
                new Item("apple", 20, new BigDecimal("9.99"))
                );
  //group by price
        Map<BigDecimal, List<Item>> groupByPriceMap =
   items.stream().collect(Collectors.groupingBy(Item::getPrice));
        System.out.println(groupByPriceMap);
  // group by price, uses 'mapping' to convert List<Item> to Set<String>
        Map<BigDecimal, Set<String>> result =
                items.stream().collect(
                        Collectors.groupingBy(Item::getPrice,
                                Collectors.mapping(Item::getName, Collectors.toSet())
                        )
                );
        System.out.println(result);
    }
}
```

Output

```
{
 19.99 = [
   Item {name ='banana'，qty = 20，price = 19.99}， 
   Item {name ='banana'，qty = 10，price = 19.99}
  ] 
 29.99 = [
   Item {name ='orang'，qty = 10，price = 29.99}， 
   Item {name ='watermelon'，qty = 10，price = 29.99}
  ] 
 9.99 = [
   Item {name ='apple'，qty = 10，price = 9.99}， 
   Item {name ='papaya'，qty = 20，price = 9.99}， 
   Item {name ='apple'，qty = 10，price = 9.99}， 
   Item {name ='apple'，qty = 20，price = 9.99}
  ]
}

// group by +映射到Set
{
 19.99 = [香蕉] 
 29.99 = [orang，西瓜]， 
 9.99 = [番木瓜，苹果]
}
```

#### 十三、遍历

List

```
ArrayList<Integer> arrayList = new ArrayList<>();
for (Integer integer : arrayList) {

}
```

Map

```
for (Map.Entry<Integer, Integer> integerIntegerEntry : hashMap.entrySet()) {

}
```
