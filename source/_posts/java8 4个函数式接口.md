---
title: jdk8 4个函数式接口
date: 2025-05-21 09:27:00
category: java
tags: [java]
---
# jdk8 4个函数式接口

## Function 

这个单词的意思就有「函数」的意思，就数学中的 y = f(x)，接收一个 x 参数，通过函数 f 运算后，返回一个结果 y。

Function 接口包含四个方法：

* apply(T t)：这是 Function 接口的主要方法，它接收一个参数并返回一个结果。同时它也是唯一的抽象的方法，剩下的都是有默认实现的（Java 8 中接口的抽象方法支持默认实现）。
* andThen(Function after)：作用是将两个 Function 组合。首先执行当前函数，再执行 andThen 函数，并将当前函数的结果作为参数传递给 andThen 函数。
* compose(Function before)：同理，将两个 Function 组合，将先执行 compose 函数，再执行当前函数，并将 compose 函数的结果作为参数传递给当前函数。
* identity(): 返回一个执行恒等转换的函数，即返回输入参数本身。

### apply 方法

```
// Function 接口的泛型，第一个参数是入参类型，第二个参数是出参类型
// Function 接口只有一个抽象方法，就是 apply()，下面利用 Lambda 表达式实现这个抽象方法并创建 Function 对象
Function<Integer, String> function = num -> "GTA" + num;
// 将5这个参数传递给function，得到返回结果
String result = function.apply(5);
System.out.println(result); // 打印：GTA5
```

### andThen 和 compose 方法

```
// 定义两个 Function 对象进行相关转换操作
Function<String, String> upperCase = s -> s.toUpperCase();
Function<String, String> addPostfix = s -> s + "5";
// 链式调用，将 gta 这个字符串参数先传递 upperCase 这个函数进行操作，然后将得到的结果传递给 addPostfix 函数进行操作，得到返回结果
String str = upperCase.andThen(addPostfix).apply("gta");
System.out.println(str); // 打印：GTA5
```

### identify 方法

```
Function<String, String> identity = Function.identity();
String result = identity.apply("hello"); // result is "hello"
```



## Consumer 

这个单词的意思就有「消费者」的意思，就把入参消费了，并不会返回结果给你。

Consumer 接口包含两个方法：

- `accept(T t)`：该方法接受一个参数并执行一些操作。
- `andThen(Consumer after)`：同理，将两个 Consumer 组合，先后进行消费。

### accept 方法

```
// Consumer 接口，泛型参数是入参类型，接受一个参数，并不返回结果，相当于消费了这个参数
Consumer<String> consumer = s -> System.out.println(s);
consumer.accept("我输入什么就打印什么"); // 打印：我输入什么就打印什么

```

### andThen 方法

```
Consumer<String> first = s -> System.out.println(s + 5);
Consumer<String> second = s -> System.out.println(s + 6);
// 先执行 first 这个 Consumer，接着执行 second 这个 Consumer
Consumer<String> combination = first.andThen(second);
combination.accept("GTA"); // 打印：GTA5 GTA6

```

## Supplier

Supplier 接口只定义了一个 `get()` 方法，该方法不接受任何参数并返回一个结果。

Supplier 这个单词的意思就有「供应者」的意思，给我的感觉就是生产者，不用参数，直接生产一个东西给你。

Supplier 接口通常用于生成一个值。例如：

```
// Supplier 接口，泛型参数是出参类型，不接受参数，但是会提供结果，相当于生产了某个东西
Supplier<String> supplier = () -> "提供一个我随便打的字符串给调用方";
String text = supplier.get();
System.out.println(text); // 打印：提供一个我随便打的字符串给调用方

```

## Predicate

Predicate 这个单词的意思就有「预言，预测，谓语，谓词」的意思，就是用来预测判断的。

`Predicate `接口包含四个方法：

- `test(T t)`：该方法接受一个参数并返回一个**布尔值**。
- `and(Predicate other)`：与另一个 Predicate 进行组合，实现**逻辑与**操作。
- `negate()`：与另一个 Predicate 进行组合，实现**逻辑非**操作。
- `or(Predicate other)`：与另一个 Predicate 进行组合，实现**逻辑或**操作。

### test方法

Predicate 接口通常用于测试一个条件是否成立。例如：

```
// Predicate 接口，泛型参数是入参类型，返回布尔值
Predicate<String> predicate = s -> s.contains("god23bin");
boolean flag = predicate.test("god23bin能给你带来收获吗？");
System.out.println("god23bin能给你带来收获吗？" + flag); // 打印：god23bin能给你带来收获吗？true

```

### and 方法

```
Predicate<String> startsWithA = (str) -> str.startsWith("A"); // 如果传入的字符串是A开头，则返回 true
Predicate<String> endsWithZ = (str) -> str.endsWith("Z"); // 如果传入的字符串是Z结尾，则返回 true

Predicate<String> startsWithAAndEndsWithZ = startsWithA.and(endsWithZ);
System.out.println(startsWithAAndEndsWithZ.test("ABCDEFZ")); // true
System.out.println(startsWithAAndEndsWithZ.test("BCDEFGH")); // false

```

### negate 方法

使用 negate 进行组合，**非**操作：

```
Predicate<String> notStartsWithA = startsWithA.negate();
System.out.println(notStartsWithA.test("ABCDEF")); // false
System.out.println(notStartsWithA.test("BCDEFGH")); // true

```

### or 方法

使用 or 进行组合，**或**操作：

```Predicate<String> startsWithAOrEndsWithZ = startsWithA.or(endsWithZ);
System.out.println(startsWithAOrEndsWithZ.test("ABCDEF")); // true
System.out.println(startsWithAOrEndsWithZ.test("BCDEFGH")); // false