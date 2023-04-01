# Java 数组&枚举

[TOC]

## 数组

Java 数组中的元素一定会被初始化，并且无法访问数组边界之外的元素。这种边界检查的代价是需要消耗少许内存，以及运行时需要少量时间来验证索引的正确性。其背后的假设是，安全性以及生产力的改善完全可以抵消这些代价（同时 Java 也会优化这些操作）。

### 数组初始化

~~~java
int[] a1 = {1, 2, 3, 4, 5};
int[] a2 = new int[10];
int[] a3 = new int[] {1, 2, 3, 4, 5,};			//初始值列表的最后一个逗号都是可选的（此功能可以让维护长列表更容易）。

//int[] a4 = new int[10] {1, 2, 3};  不支持
~~~

### 数组成员

所有数组都有`length`字段，不可修改。

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

枚举类的静态方法`values()`，它按照声明顺序生成一个enum常量值的数组。枚举对象的ordinal()方法，来表示特定enum常量的声明顺序。



### 枚举与switch 

~~~java
Spiciness s = Spiciness.NOT
switch (s) {
	case NOT:				//无需再添加枚举名Spiciness
    case MILD:
    case HOT:
}
~~~



