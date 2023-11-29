# Kotlin

Kotlin REPL is an interactive shell.



![image-20231120212026496](assets/image-20231120212026496.png)



Kotlin keeps numbers as primitives, but it lets you call methods on numbers as if they were objects.

~~~kotlin
2.times(3)
3.5.plus(4)
2.4.div(2)
~~~

this is done by **boxing ** mechanism, namely it creates actual object wrappers around numbers.   Boxing happens automatically



声明变量：

~~~kotlin
var i : Int = 5;			// 显式指定类型
var b = i.toByte();			// 隐式推断类型
~~~



To make long numeric constants more readable, Kotlin allows you to place underscores in the numbers：

~~~kotlin
val oneMillion = 1_000_000
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
~~~



`val`相当于Java中的 `final`，而`var`就是声明一个可修改的变量。



Assign a `Byte` value to variables of different types.

~~~kotlin
val b2: Byte = 1 // OK, literals are checked statically
println(b2)
⇒ 1

val i1: Int = b2
⇒ error: type mismatch: inferred type is Byte but Int was expected

val i2: String = b2
⇒ error: type mismatch: inferred type is Byte but String was expected

val i3: Double = b2
⇒ error: type mismatch: inferred type is Byte but Double was expected
~~~

~~~kotlin
val i4: Int = b2.toInt() // OK!
println(i4)
⇒ 1

val i5: String = b2.toString()
println(i5)
⇒ 1

val i6: Double = b2.toDouble()
println(i6)
⇒ 1.0
~~~



String类型支持` ' ‘`、`" "`、`$variable`、`${expression}`·、`+` 语法  