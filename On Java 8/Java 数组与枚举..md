# Java 数组&枚举

[TOC]

## 数组

Java 数组中的元素一定会被初始化（默认初始化为null），并且无法访问数组边界之外的元素。这种边界检查的代价是需要消耗少许内存，以及运行时需要少量时间来验证索引的正确性。其背后的假设是，安全性以及生产力的改善完全可以抵消这些代价（同时 Java 也会优化这些操作）。



人们常常出于效率原因使用数组。但是你都使用Java了，还在乎什么效率问题 : ) 所以我推荐优先使用Collection中的ArrayList类型，它动态维护数组的大小，消除了数组固定大小的限制。

### 数组初始化

在Java中，数组对象在声明时无需指定数组大小，这更加强调对象这个概念！这也就意味着同一个数组对象可以指向不同大小的数组。背后所依赖的机制是数组分配在堆当中，而不像C/C++那样分配在栈中，这提供了很大的灵活性。

~~~java
int[] a = new int[10];
a = new int[100];
~~~



~~~java
int[] a2 = new int[10];
int[] a1 = {1, 2, 3, 4, 5};
int[] a3 = new int[] {1, 2, 3, 4, 5,};		//初始值列表的最后一个逗号都是可选的（此功能可以让维护长列表更容易）。

//int[] a4 = new int[10] {1, 2, 3};  不支持
~~~



自动装箱机制可以应用于数组初始化中：

~~~java
Integer[][] a = {
    {1, 2, 3, 4},
    {1, 2}
}
~~~



### 数组长度

所有数组对象都有`length`字段，不可修改。

注意：length的含义是数组的最大容量，不是数组中实际保存的元素数量。

### 打印数组

使用`Arrays.toString`打印一维数组

使用`Arrays.deepToString`来打印多维数组



### 多维数组

Java支持**不规则数组（ragged array）**的特性，即在多维数组中，每个数组元素的大小是可以不同的。

~~~java
int[][] a = {
    {1, 2, 3},
    {3, 5}
}
//the length of a[0] is 3
//the length of a[1] is 2
~~~



初始化语句

~~~java
int[][] a = {
    {1, 2, 3},
    {3, 5}
};

int[][][] b = new int[3][3][3];		//此时元素都是null

for (int i = 0; i < 3; i++) {
    b[i] = new int[3][3]; 	//(1)
    b[i] = new int[3][];	//(2)
}
~~~

注意（1）（2）之间的区别。（1）表示`b[i][2][2]`为null。而（2）表示`b[i][2]`为null，因此此时若访问`b[i][2][2]`会抛出空异常，可以通过`b[i][2] = new int[3];`来修正这个问题。



### 数组与泛型

见 [Java泛型](./Java 泛型.md)笔记中的泛型数组一节

### 数组元素的填充

~~~java
int[] a = int[10];
Arrays.fill(a, 9);					// 9 9 9 9 9 9 9 9 9 9 
Arrays.fill(a, 3, 5, 1);			// 9 9 9 1 1 1 9 9 9 9
~~~

~~~java
<T> void setAll(T[] a, IntFunction<? extends T> gen);
~~~



### 其他实用工具

~~~java
Arrays.stream(new boolean[10]);
new MyClass[10].sort();		//MyClass必须实现Comparable接口
~~~



## 枚举

Java 5中添加了`enum`关键字

创建一个枚举

~~~java
public enum Spiciness {
	NOT, MILD, MEDIUM, HOT, FlAMING
}
~~~



创建一个枚举对象以及使用它

~~~java
public class EnumOrder {
	public void static main(String[] args) {
        Spiciness n = Spiciness.NOT;			//直接访问枚举值
		for (Spiciness s : Spiciness.values()) {
            System.out.println(s + " " + s.ordinal());
        }
    }
}
~~~

枚举类的静态方法`values()`，它按照声明顺序生成一个enum常量值的数组。枚举对象的`ordinal()`方法，来表示特定enum常量的声明顺序。



### 枚举与switch 

~~~java
Spiciness s = Spiciness.NOT
switch (s) {
	case NOT:				//无需再添加枚举名Spiciness
    case MILD:
    case HOT:
}
~~~



