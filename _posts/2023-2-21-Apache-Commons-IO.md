---
title: Apache Commons IO
date: 2023-2-21 09:21:50
category: Java
tags: "apache commons"
excerpt: **Apache Commons IO**是对java.io的扩展，用于协助开发IO功能。
---
## Apache Commons IO简介

**Apache Commons IO**是对java.io的扩展，主要是对Java中的bio封装了一些好用的工具类，nio涉及的较少，关于bio和nio问题我们后续再聊。

Commons IO目前最新版本是2.11.0，最低要求Java8以上。

以下为整体结构：

```text
io - 此包定义了用于处理流、读取器、写入器和文件的实用程序类。
comparator - 此软件包为文件提供了各种比较器实现。
file - 此软件包在 java.nio.file 领域提供扩展。
filefilter - 此包定义了一个接口 （IOFileFilter），该接口结合了 FileFilter 和 FilenameFilter。
function - 此包为 lambda 表达式和方法引用定义仅 IO 相关函数接口。
input - 此包提供输入类的实现，例如 InputStream 和 Reader。
input.buffer - 此包提供缓冲输入类的实现，例如 CircularBufferInputStream 和 PeekableInputStream。
monitor - 此包提供用于监控文件系统事件（目录和文件创建、更新和删除事件）的组件。
output - 此包提供输出类的实现，例如输出流和编写器。
serialization - 此包提供用于控制类反序列化的框架。
```

## maven坐标

```xml
<!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>

```

下面只列举其中常用的加以说明，其余感兴趣的可以自行翻阅源码研究。

## IOUtils

IOUtils可以说是Commons IO中最常用的了，下面直接看例子。

### 关闭流

```java
InputStream inputStream = new FileInputStream("test.txt");
OutputStream outputStream = new FileOutputStream("test.txt");
// 原生写法
if (inputStream != null) {
    try {
        inputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
if (outputStream != null) {
    try {
        outputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
// commons写法(可以传任意数量的流)
IOUtils.closeQuietly(inputStream, outputStream);
```



### 读取流

```java
// ==== 输入流转换为byte数组 ====
// 原生写法
InputStream is = new FileInputStream("foo.txt");
byte[] buf = new byte[1024];
int len;
ByteArrayOutputStream out = new ByteArrayOutputStream();
while ((len = is.read(buf)) != -1) {
    out.write(buf, 0, len);
}
byte[] result = out.toByteArray();
// commons写法
byte[] result2 = IOUtils.toByteArray(is);

// ---------------------------------------

// ==== 输入流转换为字符串 ====
// 原生写法
InputStream is = new FileInputStream("foo.txt");
BufferedReader br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
StringBuilder sb = new StringBuilder();
String line;
while ((line = br.readLine()) != null) {
    sb.append(line);
}
String result = sb.toString();
// commons写法
String result2 = IOUtils.toString(is, "UTF-8");

// IOUtils.toString 还有很多重载方法，保证有你想要的
// 将reader转换为字符串
String toString(Reader reader, String charset) throws IOException;
// 将url转换为字符串，也就是可以直接将网络上的内容下载为字符串
String toString(URL url, String charset) throws IOException;
```



### 其他

```java
// 按照行读取结果
InputStream is = new FileInputStream("test.txt");
List<String> lines = IOUtils.readLines(is, "UTF-8");

// 将行集合写入输出流
OutputStream os = new FileOutputStream("newTest.txt");
IOUtils.writeLines(lines, System.lineSeparator(), os, "UTF-8");

// 拷贝输入流到输出流
InputStream inputStream = new FileInputStream("src.txt");
OutputStream outputStream = new FileOutputStream("dest.txt");
IOUtils.copy(inputStream, outputStream);
```



## 文件相关

文件相关主要有FileUtils：文件工具类，FilenameUtils：文件名工具类，PathUtils：路径工具类（主要是操作JDK7新增的java.nio.file.Path类）



#### 文件读写

```java
File readFile = new File("test.txt");
// 读取文件
String str = FileUtils.readFileToString(readFile, "UTF-8");
// 读取文件为字节数组
byte[] bytes = FileUtils.readFileToByteArray(readFile);
// 按行读取文件
List<String> lines =  FileUtils.readLines(readFile, "UTF-8");

File writeFile = new File("out.txt");
// 将字符串写入文件
FileUtils.writeStringToFile(writeFile, "测试文本", "UTF-8");
// 将字节数组写入文件
FileUtils.writeByteArrayToFile(writeFile, bytes);
// 将字符串列表一行一行写入文件
FileUtils.writeLines(writeFile, lines, "UTF-8");
```



### 移动和复制

```java
File srcFile = new File("src.txt");
File destFile = new File("dest.txt");
File srcDir = new File("/srcDir");
File destDir = new File("/destDir");
// 移动/拷贝文件
FileUtils.moveFile(srcFile, destFile);
FileUtils.copyFile(srcFile, destFile);
// 移动/拷贝文件到目录
FileUtils.moveFileToDirectory(srcFile, destDir, true);
FileUtils.copyFileToDirectory(srcFile, destDir);
// 移动/拷贝目录
FileUtils.moveDirectory(srcDir, destDir);
FileUtils.copyDirectory(srcDir, destDir);
// 拷贝网络资源到文件
FileUtils.copyURLToFile(new URL("http://xx"), destFile);
// 拷贝流到文件
FileUtils.copyInputStreamToFile(new FileInputStream("test.txt"), destFile);
// ... ...
```



### 其他文件操作

```java
File file = new File("test.txt");
File dir = new File("/test");
// 删除文件
FileUtils.delete(file);
// 删除目录
FileUtils.deleteDirectory(dir);
// 文件大小，如果是目录则递归计算总大小
long s = FileUtils.sizeOf(file);
// 则递归计算目录总大小，参数不是目录会抛出异常
long sd = FileUtils.sizeOfDirectory(dir);
// 递归获取目录下的所有文件
Collection<File> files = FileUtils.listFiles(dir, null, true);
// 获取jvm中的io临时目录
FileUtils.getTempDirectory();
// ... ...
```



### 文件名称相关

```java
// 获取名称，后缀等
String name = "/home/xxx/test.txt";
FilenameUtils.getName(name); // "test.txt"
FilenameUtils.getBaseName(name); // "test"
FilenameUtils.getExtension(name); // "txt"
FilenameUtils.getPath(name); // "/home/xxx/"

// 将相对路径转换为绝对路径
FilenameUtils.normalize("/foo/bar/.."); // "/foo"
```



### JDK7的Path操作

```java
// path既可以表示目录也可以表示文件

// 获取当前路径
Path path = PathUtils.current();
// 删除path
PathUtils.delete(path);
// 路径或文件是否为空
PathUtils.isEmpty(path);
// 设置只读
PathUtils.setReadOnly(path, true);
// 复制
PathUtils.copyFileToDirectory(Paths.get("test.txt"), path);
PathUtils.copyDirectory(Paths.get("/srcPath"), Paths.get("/destPath"));
// 统计目录内文件数量
Counters.PathCounters counters = PathUtils.countDirectory(path);
counters.getByteCounter(); // 字节大小
counters.getDirectoryCounter(); // 目录个数
counters.getFileCounter(); // 文件个数
// ... ...
```



## 流相关

org.apache.commons.io.input和org.apache.commons.io.output包下有许多好用的过滤流，下面列举几个做下说明



### 自动关闭的输入流 AutoCloseInputStream

```java
/**
 * AutoCloseInputStream是一个过滤流，用来包装其他流，读取完后流会自动关掉
 * 实现原理很简单，当读取完后将底层的流关闭，然后创建一个ClosedInputStream赋值给它包装的输入流。
 * 注：如果输入流没有全部读取是不会关掉底层流的
 */
public void autoCloseDemo() throws Exception {
    InputStream is = new FileInputStream("test.txt");
    AutoCloseInputStream acis = new AutoCloseInputStream(is);
    IOUtils.toByteArray(acis); // 将流全部读完
    // 可以省略关闭流的逻辑了
}
```



### 倒序文件读取 ReversedLinesFileReader

```java
// 从后向前按行读取
try (ReversedLinesFileReader reader = new ReversedLinesFileReader(new File("test.txt"), Charset.forName("UTF-8"))) {
    String lastLine = reader.readLine(); // 读取最后一行
    List<String> line5 = reader.readLines(5); // 从后再读5行
}
```



### 带计数功能的流 CountingInputStream，CountingOutputStream

```java
/**
 * 大家都知道只给一个输入流咱们是没办法准确的知道它的大小的，虽说流提供了available()方法
 * 但是这个方法只有在ByteArrayInputStream的情况下拿到的是准确的大小，其他如文件流网络流等都是不准确的
 * （当然用野路子也可以实现，比如写入临时文件通过File.length()方法获取，然后在将文件转换为文件流）
 * 下面这个流可以实现计数功能，当把文件读完大小也就计算出来了
 */
public void countingDemo() {
    InputStream is = new FileInputStream("test.txt");
    try (CountingInputStream cis = new CountingInputStream(is)) {
        String txt = IOUtils.toString(cis, "UTF-8"); // 文件内容
        long size = cis.getByteCount(); // 读取的字节数
    } catch (IOException e) {
        // 异常处理
    }
}
```



### 可观察的输入流 ObservableInputStream

可观察的输入流（典型的观察者模式），可实现边读取边处理

比如将某些字节替换为另一个字节，计算md5摘要等

当然你也可以完全写到文件后在做处理，这样相当于做了两次遍历，性能较差。

这是一个基类，使用时需要继承它来扩展自己的流，示例如下：

```java
private class MyObservableInputStream extends ObservableInputStream {
    class MyObserver extends Observer {
        @Override
        public void data(final int input) throws IOException {
            // 做自定义处理
        }
        @Override
        public void data(final byte[] input, final int offset, final int length) throws IOException {
            // 做自定义处理
        }
    }
    public MyObservableInputStream(InputStream inputStream) {
        super(inputStream);
    }
}
```



### 其他

BOMInputStream: 同时读取文本文件的bom头

BoundedInputStream：有界的流，控制只允许读取前x个字节

BrokenInputStream: 一个错误流，永远抛出IOException

CharSequenceInputStream: 支持StringBuilder,StringBuffer等读取

LockableFileWriter: 带锁的Writer，同一个文件同时只允许一个流写入，多余的写入操作会跑出IOException

StringBuilderWriter: StringBuilder的Writer

... ...



## 文件比较器

org.apache.commons.io.compare包有很多现成的文件比较器，可以对文件排序的时候直接拿来用。

**DefaultFileComparator**：默认文件比较器，直接使用File的compare方法。（文件集合排序（ Collections.sort() ）时传此比较器和不传效果一样）

**DirectoryFileComparator**：目录排在文件之前

**ExtensionFileComparator**：扩展名比较器，按照文件的扩展名的ascii顺序排序，无扩展名的始终排在前面

**LastModifiedFileComparator**：按照文件的最后修改时间排序

**NameFileComparator**：按照文件名称排序

**PathFileComparator**：按照路径排序，父目录优先排在前面

**SizeFileComparator**：按照文件大小排序，小文件排在前面（目录会计算其总大小）

**CompositeFileComparator**：组合排序，将以上排序规则组合在一起

使用示例如下：

```java
List<File> files = Arrays.asList(new File[]{
        new File("/foo/def"),
        new File("/foo/test.txt"),
        new File("/foo/abc"),
        new File("/foo/hh.txt")});
// 排序目录在前
Collections.sort(files, DirectoryFileComparator.DIRECTORY_COMPARATOR); // ["/foo/def", "/foo/abc", "/foo/test.txt", "/foo/hh.txt"]
// 排序目录在后
Collections.sort(files, DirectoryFileComparator.DIRECTORY_REVERSE); // ["/foo/test.txt", "/foo/hh.txt", "/foo/def", "/foo/abc"]
// 组合排序，首先按目录在前排序，其次再按照名称排序
Comparator dirAndNameComp = new CompositeFileComparator(
            DirectoryFileComparator.DIRECTORY_COMPARATOR,
            NameFileComparator.NAME_COMPARATOR);
Collections.sort(files, dirAndNameComp); // ["/foo/abc", "/foo/def", "/foo/hh.txt", "/foo/test.txt"]
```



## 文件监视器

org.apache.commons.io.monitor包主要提供对文件的创建、修改、删除的监听操作，下面直接看简单示例。

```java
public static void main(String[] args) throws Exception {
    // 监听目录下文件变化。可通过参数控制监听某些文件，默认监听目录所有文件
    FileAlterationObserver observer = new FileAlterationObserver("/foo");
    observer.addListener(new myListener());
    FileAlterationMonitor monitor = new FileAlterationMonitor();
    monitor.addObserver(observer);
    monitor.start(); // 启动监视器
    Thread.currentThread().join(); // 避免主线程退出造成监视器退出
}

private class myListener extends FileAlterationListenerAdaptor {
    @Override
    public void onFileCreate(File file) {
        System.out.println("fileCreated:" + file.getAbsolutePath());
    }
    @Override
    public void onFileChange(File file) {
        System.out.println("fileChanged:" + file.getAbsolutePath());
    }
    @Override
    public void onFileDelete(File file) {
        System.out.println("fileDeleted:" + file.getAbsolutePath());
    }
}
```



## 总结

除了以上介绍的工具类外，还有其他不是很常用的就不多做介绍了。感兴趣的可以自行翻阅源码研究。