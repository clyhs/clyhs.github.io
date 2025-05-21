---
title: java8中filter()，map()，sorted()，distinct()，limit()，reduce()的用法
date: 2025-05-21 11:52:53
category: java
tags: java
---
# java8中filter()，map()，sorted()，distinct()，limit()，reduce()的用法

在Java 8中，引入了一个名为Stream的全新API，用于处理集合数据。Stream API提供了一种流式处理数据的方法，可以通过链式调用一系列的操作来处理集合中的元素。以下是主要的Stream API及其代码示例：

**1. filter()：根据指定条件过滤流中的元素，并返回一个新的流。**

```typescript
java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // 过滤出偶数
        List<Integer> evenNumbers = numbers.stream()
                                           .filter(number -> number % 2 == 0)
                                           .collect(Collectors.toList());

        System.out.println(evenNumbers); // 输出: [2, 4]
    }
}
```

**2. map()：将流中的元素映射为另一种类型，并返回一个新的流。**

```typescript
java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // 将名字转换为大写
        List<String> upperCaseNames = names.stream()
                                           .map(name -> name.toUpperCase())
                                           .collect(Collectors.toList());

        System.out.println(upperCaseNames); // 输出: [ALICE, BOB, CHARLIE]
    }
}
```

**3. sorted()：对流中的元素进行排序，默认是升序。也可以传入自定义的Comparator来进行排序。**

```java
java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9, 2);

        // 排序
        List<Integer> sortedNumbers = numbers.stream()
                                             .sorted()
                                             .collect(Collectors.toList());

        System.out.println(sortedNumbers); // 输出: [1, 1, 2, 3, 4, 5, 9]
    }
}
```

**4. distinct()：去除流中的重复元素。**

```java
java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5);

        // 去除重复元素
        List<Integer> distinctNumbers = numbers.stream()
                                               .distinct()
                                               .collect(Collectors.toList());

        System.out.println(distinctNumbers); // 输出: [1, 2, 3, 4, 5]
    }
}
```

**5. limit()：截取流中的前N个元素。**

```java
java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // 截取前3个元素
        List<Integer> limitedNumbers = numbers.stream()
                                              .limit(3)
                                              .collect(Collectors.toList());

        System.out.println(limitedNumbers); // 输出: [1, 2, 3]
    }
}
```

**6. reduce()：对流中的元素进行归约操作，可以通过提供的操作符（如加法、乘法等）进行计算。**

```java
java.util.Arrays;
import java.util.List;
import java.util.Optional;

public class Main {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // 求和
        Optional<Integer> sum = numbers.stream()
                                       .reduce((a, b) -> a + b);

        System.out.println(sum.orElse(0)); // 输出: 15
    }
}
```

------

> 除了以上提到的几个方法外，还有许多其他强大的Stream API，如**flatMap()**、**forEach()**、**count()**等。这些API提供了更高效、更简洁的方式来处理集合数据，使代码更易读、维护和扩展。希望以上信息能对你有所帮助！



**7 flatMap将嵌套集合展开为单一流**

用法

处理嵌套的 `List` 或 `Set`，将其扁平化为单一流。

示例代码

```java
import java.util.*;
import java.util.stream.Collectors;

public class NestedListExample {
    public static void main(String[] args) {
        List<List<String>> nestedList = Arrays.asList(
            Arrays.asList("a", "b"),
            Arrays.asList("c", "d", "e"),
            Arrays.asList("f", "g")
        );

        // 使用 flatMap 将嵌套集合展开为单一流
        List<String> flatList = nestedList.stream()
                .flatMap(List::stream)
                .collect(Collectors.toList());

        System.out.println("Flattened List: " + flatList);
    }
}
JAVA 复制 全屏
```

**输出**

```less
Flattened List: [a, b, c, d, e, f, g]
```