# Java 流式编程

[TOC]

## 概述

集合优化了对象的存储，而流（Streams）则是优化了一组对象的处理。流实现了*声明式编程*（Declarative programming），这是一种编程风格——它声明了要做什么，而不是指明（每一步）如何做，这使得我们能够像编写SQL语句那样方便地操纵Java里的数据。你会注意到，命令式（Imperative）编程的形式（指明每一步如何做）会更难理解，下面是详细的代码对比：

~~~java
import java.util.*;
public class Randoms {
    public static void main(String[] args) {
        new Random(47)
            .ints(5, 20)
            .distinct()
            .limit(7)
            .sorted()			//内部迭代（internal iteration），看不到任何上述的迭代过程
            .forEach(System.out::println);
    }
    
}

import java.util.*;
public class ImperativeRandoms {
    public static void main(String[] args) {
        Random rand = new Random(47);
        SortedSet<Integer> rints = new TreeSet<>();
        while(rints.size() < 7) {				//外部迭代（external iteration）
            int r = rand.nextInt(20);
            if(r < 5) continue;
            rints.add(r);
        }
        System.out.println(rints);
    }
}
~~~

流式编程的一个核心特征就是内部迭代。当对象拆分迭代与整合迭代等价时，命令式编程才可转换为声明式编程（个人理解）。另一个重要特征，流是惰性加载的。这代表着它只在绝对必要时（在执行终端操作时）才计算。由于计算延迟，流使我们能够处理非常大（甚至无限）的序列，而不需要考虑内存问题。之后我们进一步阐述这些特征。此外，流还适合进行并行处理。



为了兼容旧代码并实现平滑的过渡，有关流的方法在接口中修饰为了default（默认的），这样以前的类库无需重构代码以实现相应的新增接口。在包`java.util.stream.*`中



流操作的类型有三种：

- 创建流
- 修改流元素（中间操作， Intermediate Operations）
- 消费流元素（终端操作， Terminal Operations）





## 创建流

### Stream.of

~~~java
int[] i = {1, 2, 3, 4};
Stream s = Stream.of(i);					//本以为会得到Int的流，但是得到是Int[]的数据流
Stream s1 = Stream.of(1, 2, 3, 4);			//会自动装箱
IntStream s1 = IntStream.of(1, 2, 3, 4);	//针对基本类型的流，在本章节中不做考虑，这是设计上的败笔。

~~~

### Stream.generate(Supplier)

Stream.generate(Supplier s)静态方法。`generate`方法返回一个无限连续的无序流，其中每个元素由提供的供应商(`Supplier`)生成，这个Supplier可以是构造器的方法引用。

~~~java
 class A implements Supplier<T> {
     @Override
     T get() { return new A(); }
 }
 
 class B {
     public B() { }
 }
 
 class TestUnit {
     public static void main(String[] args) {
         Stream s = Stream.generate(B::new);
         Stream s1 = Stream.generate(() -> new B());
         Stream s2 = Stream.generate(new A());
 	}
 }
~~~

### Stream.iterate(T , UnaryOperator)

iterate(T seed, UnaryOperator f)，可以无限迭代来生成流元素，这点与iterate相似。第一个参数是种子，是第一次迭代的初始值。对于第二个参数来说，一方面生成当前流元素，另一方面当作生成下一个流元素的初始值。

~~~java
Stream.iterate(1, item -> item + 1)
~~~



### Stream.builder()

~~~java
 class A {
     Stream.Builder<T> builder = Stream.builder();	//获取一个对象
     void f(T item) { builder.add(item); }					//构建这一个对象
     Stream<T> g() { return builder.build(); }		//返回这个流
 }
~~~



### 集合与数组

有些集合类可以同通过调用stream()方法产生一个流。数组只能通过Arrays.stream(T[])方法将数组转换为流

~~~java
List<String> list = Arrays.asList("java,python,ruby", "c++,scala", "javascript,kotlin");
Stream s = list.stream();

int[] i = {1, 2, 3, 4};
IntStream s = Arrays.stream(i);
~~~



### 其他方法

- random.ints()
- Pattern.splitAsStream();
- Stream.empty()
- IntStream.range();  可以代替迭代语句
- string.chars();



## 中间操作

**中间操作**又可以分为以下两种操作：

- 无状态：元素的处理不受之前元素的影响；
- 有状态：该操作只有拿到所有元素之后才能继续下去。



- `peek()` 对stream流中的每个元素进行逐个遍历处理，返回处理后的stream流。

- `sorted()`可以传入比较器

- `distinct()` 可用于消除流中的重复元素

- `filter(Predicate)`：按照条件过滤符合要求的元素， 返回新的stream流

- `map(Function)`：将已有元素转换为另一个对象类型，一对一逻辑，返回新的stream流

- `limit` 仅保留集合前面指定个数的元素，返回新的stream流

- `skip` 跳过集合前面指定个数的元素，返回新的stream流

- `flatMap`会将一个流中的每个元素映射为一个流，然后将这些流合并成一个新的流。这个新的流中包含了所有映射流中的元素。

  ~~~java
  List<List<Integer>> numberLists = Arrays.asList(
          Arrays.asList(1, 2, 3),
          Arrays.asList(2, 3, 4),
          Arrays.asList(3, 4, 5)
  );
  
  List<Integer> distinctNumbers = numberLists.stream()
          .flatMap(list -> list.stream())
          .distinct()
          .collect(Collectors.toList());
  System.out.println(distinctNumbers);
  ~~~





## 终端操作

**终结操作**又可以分为以下两种操作：

- **非短路**：必须处理完所有元素才能得到最终结果
- **短路**：遇到某些符合条件的元素就可以得到最终结果





### 数组

- `toArray()`：将流转换成适当类型的数组。
- `toArray(generator)`：在特殊情况下，生成自定义类型的数组。

### 循环

- `forEach(Consumer)`无返回值，对元素进行逐个遍历，然后执行给定的处理逻辑
- `forEachOrdered(Consumer)`： 保证 `forEach` 按照原始流顺序操作。

### 集合

- `collect(Collector)`：将流转换为指定的类型，通过Collectors进行指定



`collect`是Stream流的一个**终止方法**，会使用传递的收集器（传参）对结果执行相关的操作，这个收集器必须是`Collector接口`的某个具体实现类。`Collectors`是一个**工具类**，提供了很多的静态工厂方法，**提供了很多Collector接口的具体实现类**，是为了方便程序员使用而预置的一些较为通用的收集器。

![img](https://github.com/AtsukoRuo/note-deprecated/raw/default/Java%20SE/figure/collect%E3%80%81clooector%E5%8C%BA%E5%88%AB.png)

Stream结果collect操作的本质，其实**就是将Stream中的元素通过收集器定义的函数处理逻辑进行加工，然后输出加工后的结果**：

![img](https://pics.codingcoder.cn/pics/202207161348487.png)

根据其执行的操作类型来划分，又可将Collector分为几种不同的**大类**：

![img](https://pics.codingcoder.cn/pics/202207161818493.png)

对于**归约汇总**类的操作，Stream流中的元素逐个遍历，进入到Collector处理函数中，然后会与上一个元素的处理结果进行合并处理，并得到一个新的结果，以此类推，直到遍历完成后，输出最终的结果。比如`Collectors.summingInt()`方法的处理逻辑如下：![img](https://pics.codingcoder.cn/pics/202207161623531.png)

下面给出一个归约汇总maxBy的例子：

~~~java
Optional<Employee> highestSalaryEmployee = getAllEmployees().stream()
    	.filter(employee -> "青岛大学".equals(employee.getSubCompany()))
    	.collect(Colectors.maxBy(Comparator.comparingInt(Employee::getSalary)));

//事实上，我们可以用max()方法来简化
//.filter(employee -> "上海公司".equals(employee.getSubCompany())).max(Comparator.comparingInt(Employee::getSalary));
~~~

注：这里的Comparator不是java.util.function中的函数式接口，而是java.util中的比较器。

下面偏离一下主题，阐述以下Comparable以及Comparator这两个接口之间的区别：

- Comparable 是通过重写 compareTo 方法实现排序的，而 Comparator 是通过重写 compare 方法实现排序的；

- Comparable 必须由自定义类内部实现排序方法，而 Comparator 是外部定义并实现排序的。下面给出一个例子来说明这一点：

  ~~~java
   list.sort(new Comparator<Person>() {
      @Override
      public int compare(Person p1, Person p2) {
          return p2.getAge() - p1.getAge();
      }
  });
  
   Collections.sort(list);		//此时Person必须实现Comparable接口
  ~~~

  Comparator有更大的灵活性。即使 Person 类是第三方提供的，我们依然可以通过创建新的自定义比较器 Comparator，来实现对第三方类 Person 的排序功能。



这里Comparator中有一个comparing方法，它的作用是通过指定一个 keyExtractor 函数来实现对对象中某个属性的比较。其中这个比较属性要实现Comparable接口。

~~~java
public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }

~~~



**Collectors工具类**中提供了`groupingBy`方法用来得到一个分组操作Collector，其内部处理逻辑可以参见下图的说明：

![img](https://pics.codingcoder.cn/pics/202207162055560.png)

`groupingBy()`操作需要指定两个关键输入，即**分组函数**和**值收集器**：

- **分组函数**：一个处理函数，用于基于指定的元素进行处理，返回一个用于分组的值（即**分组结果HashMap的Key值**），对于经过此函数处理后返回值相同的元素，将被分配到同一个组里。
- **值收集器**：对于分组后的数据元素的进一步处理转换逻辑。



对于`groupingBy`分组操作而言，**分组函数**与**值收集器**二者必不可少。为了方便使用，在Collectors工具类中，提供了两个`groupingBy`重载实现，其中有一个方法只需要传入一个分组函数即可，这是因为其默认使用了toList()作为值收集器：![img](https://pics.codingcoder.cn/pics/202207161557070.png)

注意：collect接受groupingBy的返回值`Collector<T, ?, Map<K, List<T>>>`作为参数后，其返回值为`Map<K, List<T>>`

而如果不仅需要分组，还需要对分组后的数据进行处理的时候，则需要同时给定分组函数以及值收集器：

~~~java
public void groupAndCaculate() {
    // 按照子公司分组，并统计每个子公司的员工数
    Map<String, Long> resultMap = getAllEmployees().stream()
            .collect(Collectors.groupingBy(Employee::getSubCompany,
                    Collectors.counting()));
    System.out.println(resultMap);
}

/**
	{南京公司=2, 上海公司=3}
*/
~~~



下面演示一个多层级的Collector：

~~~java
public void groupByCompanyAndDepartment() {
    // 按照子公司+部门双层维度，统计各个部门内的人员数
    Map<String, Map<String, Long>> resultMap = getAllEmployees().stream()
            .collect(Collectors.groupingBy(Employee::getSubCompany,		//这里的groupingBy是一个分组函数
                    Collectors.groupingBy(Employee::getDepartment,		//这里的groupingBy是一个值收集器
                            Collectors.counting())));
    System.out.println(resultMap);
}
~~~

更多关于Collection知识请参阅[讲透JAVA Stream的collect用法与原理，远比你想象的更强大 - 架构悟道 - 博客园 (cnblogs.com)](https://www.cnblogs.com/softwarearch/p/16490440.html)

### 组合

将所有元素合并计算得到一个新的元素，降维处理

- `reduce(BinaryOperator)`：使用 **BinaryOperator** 来组合所有流中的元素。因为流可能为空，其返回值为 **Optional**。
- `reduce(identity, BinaryOperator)`：功能同上，但是使用 **identity** 作为其组合的初始值。因此如果流为空，**identity** 就是结果。
- `reduce(identity, BiFunction, BinaryOperator)`：更复杂的使用形式（暂不介绍）



### 匹配

- `allMatch(Predicate)` ：如果流的每个元素提供给 **Predicate** 后都返回 true ，结果返回为 true。在第一个 false 时，则停止执行计算。
- `anyMatch(Predicate)`：如果流的任意一个元素提供给 **Predicate** 后返回 true ，结果返回为 true。在第一个 true 是停止执行计算。
- `noneMatch(Predicate)`：如果流的每个元素提供给 **Predicate** 都返回 false 时，结果返回为 true。在第一个 true 时停止执行计算。



### 查找

- `findFirst()`： 找到第一个符合条件的元素时则终止流处理
- `findAny()`找到任何一个符合条件的元素时则退出流处理，这个**对于串行流时与findFirst相同，对于并行流时比较高效**，任何分片中找到都会终止后续计算逻辑



### 其他

- `count()`：流中的元素个数。
- `max(Comparator)`：根据所传入的 **Comparator** 返回“最大”元素。
- `min(Comparator)`：根据所传入的 **Comparator** 返回“最小”元素。
- `average()` ：求取流元素平均值。
- `max()` 和 `min()`：数值流操作无需 **Comparator**。
- `sum()`：对所有流元素进行求和。
- `iterator()`将流转换为Iterator对象
