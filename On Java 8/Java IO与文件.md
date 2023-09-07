# File & IO

[TOC]

## Path

`Path`对象代表的是一个文件的、目录的路径



构建Path对象

~~~java
Paths.get("C:\\path\\to");		 //绝对路径
Paths.get("src");				//相对路径，相对项目工程的根目录

Path.of("C:", "path", "to");	 //推荐使用，屏蔽各个平台文件分割符的差异
Path.of("PathInfo.java");		//构建了一个相对路径
Path.of(new URI(...));			//根据URI构建路径
~~~



> 在Windows中，文件分割符为反斜杠`\`，而在Linux中，文件分割符为`/`。
>
> Windows文件路径示例`C:\Users\AtsukoRuo\Desktop`。
>
> 注意相对路径是相对工程根目录的，而不是代码文件所在的目录





一些转换到其他对象的方法：

~~~java
Path p = Path.of("PathInfo.java");
Path ap = p.toAbsolutePath();			//转换为绝对路径

try {	
    ap = p.toRealPath();			    //转换为绝对路径
} catch(IOException e) {
    System.out.println(e);
}

URI u = p.toUri();						//转换为URI
File f = ap.toFile();					//转换为File类型
~~~

注意：toAbsolutePath不会访问文件系统来解析路径，而是仅仅做字符串的转换，下面给出一个例子来说明

~~~java
Path p = Path.of("src", "..", "doc");
System.out.println(p.toAbsolutePath());	//C:\Users\AtsukoRuo\Desktop\spring-demo\FileDemo\src\..\doc
System.out.println(p.toAbsolutePath().normalize());	//C:\Users\AtsukoRuo\Desktop\spring-demo\FileDemo\doc
System.out.println(p.toRealPath());		//C:\Users\AtsukoRuo\Desktop\spring-demo\FileDemo\doc
~~~

可以使用`normalize`方法来消除冗余路径，例如`..`、`.`，但是这个方法仍然不访问文件系统





选择Path的片段

~~~java
Path p = Path.of("PartsOfPaths.java").toAbsolutePath();

for(int i = 0; i < p.getNameCount(); i++)
    System.out.println(p.getName(i));

System.out.println("ends with '.java': " + p.endsWith(".java"));

System.out.println(p);
for(Path pp : p) {
    System.out.print(pp + ": ");
    System.out.print(p.startsWith(pp) + " : ");
    System.out.println(p.endsWith(pp));
}

System.out.println("Starts with " + p.getRoot() + " " + p.startsWith(p.getRoot()));
~~~

注意无论是getNameCount循环，还是for-each循环，都不包括根目录！

而且endsWith不能用来判断文件后缀，因为`endsWith()`比较的是整个路径组件，而不是名字中的一个子串。



`Files`工具类中包含了一整套用于检查`Path`的各种信息的方法：

~~~java
public class PathAnalysis {
  static void say(String id, Object result) {
    System.out.print(id + ": ");
    System.out.println(result);
  }
  public static void
  main(String[] args) throws IOException {
    System.out.println(System.getProperty("os.name"));
    Path p =
      Path.of("PathAnalysis.java").toAbsolutePath();
    say("Exists", Files.exists(p));
    say("Directory", Files.isDirectory(p));
    say("Executable", Files.isExecutable(p));
    say("Readable", Files.isReadable(p));
    say("RegularFile", Files.isRegularFile(p));
    say("Writable", Files.isWritable(p));
    say("notExists", Files.notExists(p));
    say("Hidden", Files.isHidden(p));
    say("size", Files.size(p));
    say("FileStore", Files.getFileStore(p));
    say("LastModified: ", Files.getLastModifiedTime(p));
    say("Owner", Files.getOwner(p));
    say("ContentType", Files.probeContentType(p));
    say("SymbolicLink", Files.isSymbolicLink(p));
    if(Files.isSymbolicLink(p))
      say("SymbolicLink", Files.readSymbolicLink(p));
    if(FileSystems.getDefault()
       .supportedFileAttributeViews().contains("posix"))
      say("PosixFilePermissions",
        Files.getPosixFilePermissions(p));
  }
}
/* 输出：
Windows 10
Exists: true
Directory: false
Executable: true
Readable: true
RegularFile: true
Writable: true
notExists: false
Hidden: false
size: 1617
FileStore: (C:)
LastModified: : 2021-11-08T00:34:52.693768Z
Owner: GROOT\Bruce (User)
ContentType: text/plain
SymbolicLink: false
*/
~~~



~~~java
Path p1 = paths.get("a/b");
Path p2 = Paths.get("b/c");

path relative = p1.relativize(p2);	//..\..\b\c
//p1到p2的相对路径。注意p1与p2要么都是相对路径，要么都是绝对路径，否则系统会报错

Path p3 = Paths.get("C:\\src");
Path p4 = Paths.get("b");
System.out.println(p3.resolve(p4));			//C:\src\c
//合并路径，p4最好是相对路径
~~~

### 监听Path



## 目录 & 文件

### 创建与删除

**这些静态方法都是Files类的**

删除目录或者文件

~~~java
public static void delete(Path path) throws IOException
~~~



创建文件

~~~java
public static Path createFile(Path path, FileAttribute<?>... attrs)
        throws IOException
~~~



创建目录

~~~java
public static Path createDirectories(Path dir, FileAttribute<?>... attrs)
        throws IOException
~~~



创建临时文件

~~~java
public static Path createTempFile(Path dir,
                                      String prefix,
                                      String suffix,
                                      FileAttribute<?>... attrs)
        throws IOException
~~~



递归地删除目录

~~~c
public class RmDir {
    public static void rmdir(Path dir) throws IOException {
        Files.walkFileTree(dir, new SimpleFileVisitor<>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                Files.delete(file);
                return FileVisitResult.CONTINUE;
            }

            @Override
            public FileVisitResult postVisitDirectory(Path dir) throws IOException {
                Files.delete(dir);
                return FileVisitResult.CONTINUE;
            }
        });
    }
}
~~~

- `preVisitDirectory()`：先在当前目录上运行，然后进入这个目录下的文件和目录。
- `visitFile()`：在这个目录下的每个文件上运行。
- `visitFileFailed()`：当文件无法访问时调用。
- `postVisitDirectory()`：先进入当前目录下的文件和目录（包括所有的子目录），最后在当前目录上运行。



获取Path流

~~~java
public static Stream<Path> walk(Path start, FileVisitOption... options) throws IOException			//获得包含整个目录树内容的流
    
public static DirectoryStream<Path> newDirectoryStream(Path dir) throws IOException			//获得当前目录内容的流（不考虑子目录中的内容）
~~~



### 文件系统

//TODO

### 查找文件

方案一：在`Path`上调用`toString()`，然后使用`String`的各种操作来查看结果

方案二：在`FileSystem`对象上调用`getPathMatcher()`来获得一个`PathMatcher`，并传入我们感兴趣的模式。模式有两个选项：`glob`和`regex`。然后用在目录树流中

~~~java
 PathMatcher matcher = FileSystems.getDefault()
      .getPathMatcher("glob:**/*.{tmp,txt}");

Files.walk(test)
  .filter(matcher::matches)
  .forEach(System.out::println);
~~~

### 读写文件

`Files.readAllLines()`可以一次性读入整个文件（适用于小文件），

生成一个`List<String>`，每个元素代表这文件中的一行。

如果文件是大文件，可以使用`Files.lines()`将一个文件变为一个由行组成的`Stream<String>`。



~~~c
public static Path write(Path path, byte[] bytes, OpenOption... options);
public static Path write(Path path, Iterable<? extends CharSequence> lines,
                             Charset cs, OpenOption... options)
public static Path write(Path path,
                         Iterable<? extends CharSequence> lines,
                         OpenOption... options)
    
Path path = ...
byte[] bytes = ...
Files.write(path, bytes, StandardOpenOption.APPEND);
~~~

- 参数`cs`指定编码集

- 参数options

  ~~~java
  public enum StandardOpenOption implements OpenOption {
      /**
       * Open for read access.
       */
      READ,
  
      /**
       * Open for write access.
       */
      WRITE,
  
      /**
       * If the file is opened for {@link #WRITE} access then bytes will be written
       * to the end of the file rather than the beginning.
       *
       * <p> If the file is opened for write access by other programs, then it
       * is file system specific if writing to the end of the file is atomic.
       */
      APPEND,
  
      /**
       * If the file already exists and it is opened for {@link #WRITE}
       * access, then its length is truncated to 0. This option is ignored
       * if the file is opened only for {@link #READ} access.
       */
      TRUNCATE_EXISTING,
  
      /**
       * Create a new file if it does not exist.
       * This option is ignored if the {@link #CREATE_NEW} option is also set.
       * The check for the existence of the file and the creation of the file
       * if it does not exist is atomic with respect to other file system
       * operations.
       */
      CREATE,
  
      /**
       * Create a new file, failing if the file already exists.
       * The check for the existence of the file and the creation of the file
       * if it does not exist is atomic with respect to other file system
       * operations.
       */
      CREATE_NEW,
  
      /**
       * Delete on close. When this option is present then the implementation
       * makes a <em>best effort</em> attempt to delete the file when closed
       * by the appropriate {@code close} method. If the {@code close} method is
       * not invoked then a <em>best effort</em> attempt is made to delete the
       * file when the Java virtual machine terminates (either normally, as
       * defined by the Java Language Specification, or where possible, abnormally).
       * This option is primarily intended for use with <em>work files</em> that
       * are used solely by a single instance of the Java virtual machine. This
       * option is not recommended for use when opening files that are open
       * concurrently by other entities. Many of the details as to when and how
       * the file is deleted are implementation specific and therefore not
       * specified. In particular, an implementation may be unable to guarantee
       * that it deletes the expected file when replaced by an attacker while the
       * file is open. Consequently, security sensitive applications should take
       * care when using this option.
       *
       * <p> For security reasons, this option may imply the {@link
       * LinkOption#NOFOLLOW_LINKS} option. In other words, if the option is present
       * when opening an existing file that is a symbolic link then it may fail
       * (by throwing {@link java.io.IOException}).
       */
      DELETE_ON_CLOSE,
  
      /**
       * Sparse file. When used with the {@link #CREATE_NEW} option then this
       * option provides a <em>hint</em> that the new file will be sparse. The
       * option is ignored when the file system does not support the creation of
       * sparse files.
       */
      SPARSE,
  
      /**
       * Requires that every update to the file's content or metadata be written
       * synchronously to the underlying storage device.
       *
       * @see <a href="package-summary.html#integrity">Synchronized I/O file integrity</a>
       */
      SYNC,
  
      /**
       * Requires that every update to the file's content be written
       * synchronously to the underlying storage device.
       *
       * @see <a href="package-summary.html#integrity">Synchronized I/O file integrity</a>
       */
      DSYNC;
  }
  ~~~



## I/O流

### 概述

I/O流要考虑以下两点：

- I/O的来源和去处：内存、文件、控制台、网络连接，等等
- 方式：顺序读取、随机访问、缓冲、字符、按行读取、按字读取



在Java中，一般会将多个I/O类分层放在一起来提供所需的功能（这便是**装饰器**设计模式），即生成一个流需要创建多个对象。在编写程序时，装饰器可以带来更大的灵活性（让你可以轻松地对属性进行混合和匹配），但也会增加代码的复杂性。



### InputStream & OutputStream

在Java 1.0中，库的设计者们选择让所有和输入相关的类都继承自`InputStream`，而所有和输出相关的类都继承自`OutputStream`

另外，`FilterInputStream`也是一种`InputStream`类型，它为“装饰器”类（用于为输入流增加属性或有用的接口）提供了基类

`InputStream`用于表示那些**从不同源生成输入**（读操作）的类。不同的**输入源**有不同的子类：

| 类                            | 功能                                                         | 构造器参数                                                   | 使用方法                                                     |
| :---------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ByteArray-` `InputStream`    | 使内存中的缓冲区可以充当`InputStream`                        | 用于提取出字节的缓冲区                                       | 作为一种数据源：通过将其连接到`FilterInputStream`对象来提供有用的接口 |
| `StringBuffer-` `InputStream` | 将字符串转换为`InputStream`                                  | 一个字符串。底层实现实际用的是`StringBuffer`                 | 作为一种数据源：通过将其连接到`FilterInputStream`对象来提供有用的接口 |
| `FileInputStream`             | 用于从一个文件中读取信息                                     | 一个用于表示文件名或`File`对象，还有`FileDescriptor`对象的字符串 | 作为一种数据源：通过将其连接到`FilterInputStream`对象来提供有用的接口 |
| `PipedInputStream`            | 用于生成写入到对应的`PipedOutputStream`中的数据。它实现了“管道传输”的概念 | `PipedOutputStream`                                          | 作为一种多线程形式的数据源：通过将其连接到`FilterInputStream`对象来提供有用的接口 |
| `Sequence-` `InputStream`     | 将两个以上的 `InputStream`转换为单个`InputStream`            | 两个`InputStream`对象，或一个作为`InputStream`对象容器的`Enumeration` | 作为一种数据源：通过将其连接到`FilterInputStream`对象来提供有用的接口 |
| `FilterInputStream`           | 作为装饰器接口的抽象类，装饰器用来为其他`InputStream`类提供有用的功能。参见表7-3 | 参见表7-3                                                    | 参见表7-3                                                    |





`OutputStream`用于表示那些**向不同源生成输出**（写操作）的类。不同的**输出源**有不同的子类：

| 类                          | 功能                                                         | 构造器参数                                                   | 使用方法                                                     |
| :-------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ByteArray-` `OutputStream` | 在内存创建一块缓冲区，所有发送到流中的数据都被放在该缓冲区   | 缓冲区初始大小，为可选参数                                   | 用于指定数据的目的地：通过将其连接到`FilterOutputStream`对象来提供有用的接口 |
| `FileOutputStream`          | 用于向文件发送信息                                           | 用于表示文件名或`File`对象，还有`FileDescriptor`对象的字符串 | 用于指定数据的目的地：通过将其连接到`FilterOutputStream`对象来提供有用的接口 |
| `PipedOutputStream`         | 向其中写入的任何信息都将自动作为对应的`PipedInputStream`的输入。实现了“管道传输”的概念 | `PipedInputStream`                                           | 用于为多线程指定数据的目的地：通过将其连接到`FilterOutputStream`对象来提供有用的接口 |
| `FilterOutputStream`        | 作为装饰器接口的抽象类，装饰器用来为其他`OutputStream`类提供有用的功能。参见表7-4 | 参见表7-4                                                    | 参见表7-4                                                    |

`FilterOutputStream`为“装饰器”类（用于为输出流增加属性或有用的接口）提供了基类





### FilterInputStream & FilterOutputStream 装饰器类

| 类                          | 功能                                                         | 构造器参数                                    | 使用方法                                                     |
| :-------------------------- | :----------------------------------------------------------- | :-------------------------------------------- | :----------------------------------------------------------- |
| `DataInputStream`           | 与`DataOutputStream`配合使用                                 | `InputStream`                                 | 包含用于读取基本类型的全部接口                               |
| `Buffered-` `InputStream`   | 用于防止在每次需要更多数据时都进行物理上的读取（磁盘，对于内存上的缓冲是无用）。相当于声明“使用缓冲区” | `InputStream`以及可选参数：（指定）缓冲区大小 | 这本质上并未提供接口，只是为进程增加缓冲操作而已，需要与接口对象搭配使用 |
| `LineNumber-` `InputStream` | 自动追踪并维护行号,每读取一行,行号会自动加1。可以调用`getLineNumber()`和`setLineNumber(int)`手动设置行号 | `InputStream`                                 | 只是增加了行号而已，因此可能需要与接口对象搭配使用           |
| `Pushback-` `InputStream`   | 包含一个单字节回退缓冲区，用于将最后读取的字符推回输入流     | `InputStream`                                 | 通常用于编译器的扫描器，一般不会用到                         |



| 类                         | 功能                                                         | 构造器参数                                                   | 使用方法                                                     |
| :------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `DataOutputStream`         | 与`DataInputStream`搭配使用，这样就能以可移植的方式向流中写入基本类型（`int`、`char`、`long`，等等）了 | `OutputStream`                                               | 包含用于写入基本类型数据的全部接口                           |
| `PrintStream`              | 用于生成格式化的输出。`DataOutputStream`是直接以数据的二进制来输出的，而`PrintStream`是以字符解释的形式来输出的 | `OutputStream`以及可选参数：`boolean`，表示是否在每次换行时都清空缓冲区 | 应该作为`OutputStream`对象的“最终”包装。可能会经常用到       |
| `Buffered-` `OutputStream` | 用来防止在每次发送数据的时候都发生物理写操作。相当于声明“使用缓冲”。可以调用`flush()`来清空缓冲区 | `OutputStream`以及可选参数：（指定）缓冲区大小               | 这本质上并未提供接口，只是为进程增加缓冲操作而已，需要与接口对象搭配使用 |



下面是一个简单的示例：

~~~java
class BufferedInputFile {
    public static String read(String filename) {
        try (BufferedReader in = new BufferedReader(new FileReader(filename))) {
            return in.lines().collect(Collectors.joining("\n"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    public static void main(String[] args){
        System.out.println(read("src\\BufferedInputFile.java"));
    }
}

public class FormattedMemoryInput {
  public static void main(String[] args) {
    try(
      DataInputStream in = new DataInputStream(
          new ByteArrayInputStream(BufferedInputFile
                                   .read("FormattedMemoryInput.java")
                                   .getBytes()))
    ) {
      while(true)
        System.out.write((char)in.readByte());
    } catch(EOFException e) {
      System.out.println("\nEnd of stream");
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  }
}

public class TestEOF {
  public static void main(String[] args) {
    try(
      DataInputStream in = new DataInputStream(
        new BufferedInputStream(
          new FileInputStream("TestEOF.java")))
    ) {
      while(in.available() != 0)
        System.out.write(in.readByte());
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  }
}
~~~





注意，`available()`的工作方式会随着所读取媒介类型的不同而有所不同——该方法的字面意思是“**在没有阻塞的情况下**所能读取的字节数量”。对于文件来说，这意味着整个文件，但是对于不同类型的流来说，则可能并不是这样。



还有DataOutputStream的`writeUTF()`和`readUTF()`使用的是一种适用于Java的特殊UTF-8变体（JDK文档中有关于这些方法的描述），因此如果你用非Java程序来读取用`writeUTF()`写入的字符串，就必须编写特殊的代码来妥当地读取该字符串。

### Reader & Writer

Java 1.1对基础流式I/O库进行了重大的修改。`InputStream`和`OutputStream`类仍然以**面向字节**（8位字节流）的I/O的形式提供了有价值的功能，而`Reader`和`Writer`类则提供了兼容Unicode（国际化）并且基于字符的I/O能力（**面向字符**）。



| 来源和去处：Java 1.0中的类           | Java 1.1中对应的类                    |
| :----------------------------------- | :------------------------------------ |
| `InputStream`                        | `Reader` 适配器：`InputStreamReader`  |
| `OutputStream`                       | `Writer` 适配器：`OutputStreamWriter` |
| `FileInputStream`                    | `FileReader`                          |
| `FileOutputStream`                   | `FileWriter`                          |
| `StringBufferInputStream` （已弃用） | `StringReader`                        |
| （没有对应的类）                     | `StringWriter`                        |
| `ByteArrayInputStream`               | `CharArrayReader`                     |
| `ByteArrayOutputStream`              | `CharArrayWriter`                     |
| `PipedInputStream`                   | `PipedReader`                         |
| `PipedOutputStream`                  | `PipedWriter`                         |



改变流的行为

| 过滤器：Java 1.0中的类             | Java 1.1中对应的类                                   |
| :--------------------------------- | :--------------------------------------------------- |
| `FilterInputStream`                | `FilterReader`                                       |
| `FilterOutputStream`               | `FilterWriter` （没有任何子类的抽象类）              |
| `BufferedInputStream`              | `BufferedReader` （同样包含`readLine()`）            |
| `BufferedOutputStream`             | `BufferedWriter`                                     |
| `DataInputStream`                  | `DataInputStream` 不推荐使用                         |
| `DataOutputStream`                 | 无                                                   |
| `PrintStream`                      | `PrintWriter`                                        |
| `LineNumberInputStream` （已弃用） | `LineNumberReader`                                   |
| `StreamTokenizer`                  | `StreamTokenizer` （使用以`Reader`作为参数的构造器） |
| `PushbackInputStream`              | `PushbackReader`                                     |

`BufferedOutputStream`是`FilterOutputStream`的子类，但`BufferedWriter`**不是**`FilterWriter`的子类，尽管`FilterWriter`是抽象类，但它没有任何子类，因此把它放在那里似乎只是当作占位符，或者仅仅是为了告诉你有这么一个类而已



### RandomAccessFile

`RandomAccessFile`适合用来处理由大小已知的记录组成的文件，由此可以通过`seek()`在各条记录上来回移动，然后读取或者修改记录



~~~java
String file = "";
try(
      RandomAccessFile rf =
        new RandomAccessFile(file, "r")
) {
      System.out.println("Value " + i + ": " + rf.readDouble());
} catch(IOException e) {
      throw new RuntimeException(e);
}
~~~





### 标准IO

**标准I/O**（standard I/O）指的是UNIX中“程序所使用的单一信息流”这个概念。程序的所有输入都可以来自**标准输入**，所有输出都可以发送到**标准输出**，所有错误都可以发送到**标准错误**。标准I/O的价值在于可以很容易地使多个程序串联起来，一个程序的标准输出可以成为另一个程序的标准输入。

遵循标准I/O模型，Java实现了`System.in`、`System.out`，以及`System.err`。其中`System.out`已经被预先包装为`PrintStream`对象。`System.err`也同样是一个`PrintStream`，但是`System.in`是未经包装的原生`InputStream`。这意味着虽然可以随时使用`System.out`和`System.err`，但是`System.in`则必须在读取前先进行包装。



从标准输入中读取

~~~java
BufferedReader br = new BufferReader(new InputStreamRead(System.in));
~~~

向标准输出中写入

~~~java
PrintWriter out = new PrintWriter(System.out, true);
~~~

第二个参数设为`true`来开启自动清空的功能。否则可能就看不到输出。



Java的`System`类可以通过简单的静态方法调用对标准输入、标准输出以及标准错误的I/O标准流进行重定向：

- `setIn(InputStream)`
- `setOut(PrintStream)`
- `setErr(PrintStream)`

~~~java
PrintStream console = System.out;
try (
    BufferedInputStream in = new BufferedInputStream(
            new FileInputStream("Redirecting.java"));
    PrintStream out = new PrintStream(
            new BufferedOutputStream(
                    new FileOutputStream("src\\Redirecting.txt")
            ))) {
    System.setIn(in);
    System.setOut(out);
    System.setErr(out);

    new BufferedReader(new InputStreamReader(System.in))
            .lines()
            .forEach(System.out::println);
} catch (IOException e) {
    throw new RuntimeException(e);
} finally {
     System.setOut(console);
}
~~~







## NIO

> 只有在遇到性能问题，才使用nio

![img](C:\Users\AtsukoRuo\Desktop\note\On Java 8\assets\01.jpg)



## 内存映射文件

## 文件加锁



