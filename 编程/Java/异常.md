# Java 异常

## try - with - resource

我们先看一个处理构造器异常的错误例子：

~~~java
BufferedReader in = null;
try {
    in = new BufferedReader(new FileReader(filename));
} catch (Exception e) {
    
} finally {
    in.close()；
}

~~~

如果构造器抛出了异常，那么表明对象处于不安全的状态。但是这里不管对象构造成功与否，finally总会调用对象的close方法。这显然是不合理的。

正确的做法是使用嵌套的try块：

~~~java
try {
    BufferReader in = new BufferedReader(new FileReader(filename));
    try {
        
    } catch (Excpetion e) {
        //处理其他异常
    } finally {
        in.close();
    }
} catch (Exception e) {
    //处理构造器抛出的异常
}
~~~



再给出一个更加常见的例子：

~~~java
class NeedsCleanup {			
    //构造不会失败
    public void dispose() {}
}

class ConstructionException extends Exception {}

class NeedsCleanup2 {
    //构造可能失败
    NeedsCleanup2（） throws Exception {}
    public void clean() {}
}

public class CleanupIdiom {
    public static void main(String[] args) {
        
        NeedsCleanup nc1 = new NeedsCleanup();		//[1]
        NeedsCleanup nc2 = new NeedsCleanup();
        try {
            //其他代码
        } catch (Exception e) {
            
        } finally {
            nc2.dispose();
            nc1,dispose();						//[3]
        }
        
        try {
            NeedsCleanup2 nc3 = new NeedsCleanup2();
            try {
                NeedsCleanup2 nc4 = new NeedsCleanup2();	//[2]
                try {
                    //其他代码
                } catch (Exception e) {
                    
                } finally {
                    nc4.clean();
                }
            } catch (ConstructionException e) {
                //处理nc4的构造异常
            }
            finally {
                nc3.clean(); 
            }
        } catch (ConstructionException e) {
            //处理nc3的构造异常
        }
    }
}
~~~



总结上述惯例用法为：

1. 对于每个需要清理的类，在构造完后立即跟着try-finally语句来保证清理

2. 对于在构造时不会抛出异常的类，它们放在一起。反之，需要嵌套try块。

3. 按照相反的构造顺序，执行清理工作




这种处理构造器异常的模板代码还有一种简化形式：

~~~java
InputStream in = null;
try {
    in = new FileInputStream(new File(""));
    //其他代码
} catch (Exception e) {
    //其他异常
} catch (ConstructionException e) {
    //构造器异常
} finally {
    if (in != null) {
        //对象已成功构造
        try {
            in.clean();
        } catch (Exception e) {
            
        }
    }
}
~~~



​	但编写起来还是有点繁琐，为此Java 7 引入了try - with - resources语法

~~~java
try (
    InputStream in = new FileInputStream(new File(""));
    PrintWriter outfile = new PrintWriter("");
) {
    
} catch (Exception e) {
    
}
~~~

- try括号中的内容称为**资源说明头（resource specification header）**
- 可以在资源说明头中定义多个对象
- 每个对象必须实现`java.lang.AutoCloseable`接口。该接口只有一个方法`void close()`。





~~~java
package exceptions;
class ConstructionException extends Exception {}
class CloseException extends Exception {}
class First implements AutoCloseable {
    public First() { System.out.println("First()"); }
    public void close() { System.out.println("First close()"); }
}

class Second implements AutoCloseable {
    public Second() { System.out.println("Second()"); }
    public void close() throws CloseException { System.out.println("Second close()"); throw new CloseException(); }
}

class Third implements AutoCloseable {
    public Third() throws ConstructionException { System.out.println("Third()"); throw new ConstructionException(); }
    public void close() { System.out.println("Third close()"); }
}

class Forth implements AutoCloseable {
    public Forth() { System.out.println("Forth()"); }
    public void close() { System.out.println("Forth close()"); }
}


public class TryWithResources {
    public static void main(String[] args) {
        try (
            First first = new First();
            Second second = new Second();
            Third third = new Third();
            Forth forth = new Forth();
            )
        {
            System.out.println("In Method");
        } catch (ConstructionException e) {
            System.out.println("ConstructionException");
        } catch (CloseException e) {
            System.out.println("CloseException");
        } finally {
            System.out.println("Finally");
        }
    }
}

/*

*/
~~~





