---
title: Apache Commons Compress 提供了java语言实现的对文件的压缩解压工具
date: 2023-2-21
category: Java
tags: apache-commons
excerpt: Apache Commons Compress提供了许多解压缩、归档文件相关的工具类。
---
**Apache Commons Compress**提供了许多编解码相关的工具类。Compress目前最新版本是1.22，最低要求Java8以上。

以下为整体结构：
- archivers #归档
- changes #变化
- compressors #压缩
- parallel #并行
- utils #工具
- harmony #pack算法抽离
## maven坐标

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-compress -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.22</version>
</dependency>

```

下面只列举其中常用的加以说明，其余感兴趣的可以自行翻阅源码研究。

##  压缩



压缩：按某种算法减小文件所占用空间的大小

解压：按对应的逆向算法恢复文件



- **Compress**自带了很多压缩相关的类，主要以下几个

- **GzipCompressorOutputStream：**压缩"*.gz"文件
- **GzipCompressorInputStream**：解压"*.gz"文件

- **BZip2CompressorOutputStream**：压缩"*.bz2"文件
- **BZip2CompressorInputStream**：解压"*.bz2"文件

- **XZCompressorOutputStream**：压缩"*.xz"文件
- **XZCompressorInputStream**：解压"*.xz"文件

- **FramedLZ4CompressorOutputStream**：压缩"*.lz4"文件
- **FramedLZ4CompressorInputStream**：解压"*.lz4"文件

- **BlockLZ4CompressorOutputStream**：压缩"*.block_lz4"文件
- **BlockLZ4CompressorInputStream**：解压"*.block_lz4"文件

- **Pack200CompressorOutputStream**：压缩"*.pack"文件
- **Pack200CompressorInputStream**：解压"*.pack"文件

- **DeflateCompressorOutputStream**：压缩"*.deflate"文件
- **DeflateCompressorInputStream**：解压"*.deflate"文件

- **LZMACompressorOutputStream**：压缩"*.lzma"文件
- **LZMACompressorInputStream**：解压"*.lzma"文件

- **FramedSnappyCompressorOutputStream**：压缩"*.sz"文件
- **FramedSnappyCompressorInputStream**：解压"*.sz"文件

- **ZCompressorInputStream**：解压"*.Z"文件



下面简单看看例子



###  gzip

gzip是Unix，Linux上常用的压缩工具，也是当今的WEB站点上非常流行的压缩技术。其有压缩级别等概念，可以通过GzipParameters去设置。JDK8也自带了GZIPInputStream类，用法类似。

```java
// gzip压缩
String file = "/test.js";
GzipParameters parameters = new GzipParameters();
parameters.setCompressionLevel(Deflater.BEST_COMPRESSION);
parameters.setOperatingSystem(3);
parameters.setFilename(FilenameUtils.getName(file));
parameters.setComment("Test file");
parameters.setModificationTime(System.currentTimeMillis());
FileOutputStream fos = new FileOutputStream(file + ".gz");
try (GzipCompressorOutputStream gzos = new GzipCompressorOutputStream(fos, parameters);
    InputStream is = new FileInputStream(file)) {
    IOUtils.copy(is, gzos);
}
// gzip解压
String gzFile = "/test.js.gz";
FileInputStream is = new FileInputStream(gzFile);
try (GzipCompressorInputStream gis = new GzipCompressorInputStream(is)) {
    GzipParameters p = gis.getMetaData();
    File targetFile = new File("/test.js");
    FileUtils.copyToFile(gis, targetFile);
    targetFile.setLastModified(p.getModificationTime());
}
```



### bz2

bz2是Linux下常见的压缩文件格式，是由具有高压缩率的压缩工具bzip2生成，以后缀为.bz2结尾的压缩文件。

```java
// 压缩bz2
String srcFile = "/test.tar";
String targetFile = "/test.tar.bz2";
FileOutputStream os = new FileOutputStream(targetFile);
try (BZip2CompressorOutputStream bzos = new BZip2CompressorOutputStream(os);
    InputStream is = new FileInputStream(srcFile)) {
    IOUtils.copy(is, bzos);
}
// 解压bz2
String bzFile = "/test.tar.bz2";
FileInputStream is = new FileInputStream(bzFile);
try (BZip2CompressorInputStream bzis = new BZip2CompressorInputStream(is)) {
    File targetFile = new File("test.tar");
    FileUtils.copyToFile(bzis, targetFile);
}
```

其他压缩算法的使用方式和bz2基本一致，这里就不做代码示例了。



## 归档



归档：将许多零散的文件整理为一个文件，文件总大小基本不变

解包：从归档文件中释放文件



- **Compress**自带了很多归档相关的类，主要以下几个

- **TarArchiveOutputStream**：归档"*.tar"文件
- **TarArchiveInputStream**：解包"*.tar"文件

- **ZipArchiveOutputStream**：归档压缩"*.zip"文件
- **ZipArchiveInputStream**：解包解压"*.zip"文件

- **JarArchiveOutputStream**：归档压缩"*.jar"文件
- **JarArchiveInputStream**：解包解压"*.jar"文件

- **DumpArchiveOutputStream**：归档"*.dump"文件
- **DumpArchiveInputStream**：解包"*.dump"文件

- **CpioArchiveOutputStream**：归档压缩"*.cpio"文件
- **CpioArchiveInputStream**：解包解压"*.cpio"文件

- **ArArchiveOutputStream**：归档压缩"*.ar"文件
- **ArArchiveInputStream**：解包解压"*.ar"文件

- **ArjArchiveInputStream**：解包解压"*.arj"文件

- **SevenZOutputFile**：归档压缩"*.7z"文件
- **SevenZFile**：解包解压"*.7z"文件



其中zip，jar，cpio，ar，7z既支持归档也支持压缩，能在归档的过程中做压缩处理。

由于他们会处理一个个零散的文件，所以会有ArchiveEntry的概念，即一个ArchiveEntry代表归档包内的一个目录或文件，下面简单看看例子



### tar

tar是Unix和Linux系统上的常用的压缩归档工具，可以将多个文件合并为一个文件，打包后的文件后缀亦为"tar"。

```java
// tar压缩
public void tar() throws IOException {
    File srcDir = new File("/test");
    String targetFile = "/test.tar";
    try (TarArchiveOutputStream tos = new TarArchiveOutputStream(
            new FileOutputStream(targetFile))) {
        tarRecursive(tos, srcDir, "");
    }
}
// 递归压缩目录下的文件和目录
private void tarRecursive(TarArchiveOutputStream tos, File srcFile, String basePath) throws IOException {
    if (srcFile.isDirectory()) {
        File[] files = srcFile.listFiles();
        String nextBasePath = basePath + srcFile.getName() + "/";
        if (ArrayUtils.isEmpty(files)) {
            // 空目录
            TarArchiveEntry entry = new TarArchiveEntry(srcFile, nextBasePath);
            tos.putArchiveEntry(entry);
            tos.closeArchiveEntry();
        } else {
            for (File file : files) {
                tarRecursive(tos, file, nextBasePath);
            }
        }
    } else {
        TarArchiveEntry entry = new TarArchiveEntry(srcFile, basePath + srcFile.getName());
        tos.putArchiveEntry(entry);
        FileUtils.copyFile(srcFile, tos);
        tos.closeArchiveEntry();
    }
}
// tar解压
public void untar() throws IOException {
    InputStream is = new FileInputStream("/test.tar");
    String outPath = "/test";
    try (TarArchiveInputStream tis = new TarArchiveInputStream(is)) {
        TarArchiveEntry nextEntry;
        while ((nextEntry = tis.getNextTarEntry()) != null) {
            String name = nextEntry.getName();
            File file = new File(outPath, name);
            //如果是目录，创建目录
            if (nextEntry.isDirectory()) {
                file.mkdir();
            } else {
                //文件则写入具体的路径中
                FileUtils.copyToFile(tis, file);
                file.setLastModified(nextEntry.getLastModifiedDate().getTime());
            }
        }
    }
}
```



### 7z

7z 是一种全新的压缩格式，它拥有极高的压缩比。

7z 格式的主要特征：

- 开放的结构
- 高压缩比
- 强大的 AES-256 加密
- 能够兼容任意压缩、转换、加密算法
- 最高支持 16000000000 GB 的文件压缩
- 以 Unicode 为标准的文件名
- 支持固实压缩
- 支持文件头压缩

```java
// 7z压缩
public void _7z() throws IOException {
    try (SevenZOutputFile outputFile = new SevenZOutputFile(new File("/test.7z"))) {
        File srcFile = new File("/test");
        _7zRecursive(outputFile, srcFile, "");
    }
}
// 递归压缩目录下的文件和目录
private void _7zRecursive(SevenZOutputFile _7zFile, File srcFile, String basePath) throws IOException {
    if (srcFile.isDirectory()) {
        File[] files = srcFile.listFiles();
        String nextBasePath = basePath + srcFile.getName() + "/";
        // 空目录
        if (ArrayUtils.isEmpty(files)) {
            SevenZArchiveEntry entry = _7zFile.createArchiveEntry(srcFile, nextBasePath);
            _7zFile.putArchiveEntry(entry);
            _7zFile.closeArchiveEntry();
        } else {
            for (File file : files) {
                _7zRecursive(_7zFile, file, nextBasePath);
            }
        }
    } else {
        SevenZArchiveEntry entry = _7zFile.createArchiveEntry(srcFile, basePath + srcFile.getName());
        _7zFile.putArchiveEntry(entry);
        byte[] bs = FileUtils.readFileToByteArray(srcFile);
        _7zFile.write(bs);
        _7zFile.closeArchiveEntry();
    }
}
 // 7z解压
public void un7z() throws IOException {
    String outPath = "/test";
    try (SevenZFile archive = new SevenZFile(new File("test.7z"))) {
        SevenZArchiveEntry entry;
        while ((entry = archive.getNextEntry()) != null) {
            File file = new File(outPath, entry.getName());
            if (entry.isDirectory()) {
                file.mkdirs();
            }
            if (entry.hasStream()) {
                final byte [] buf = new byte [1024];
                final ByteArrayOutputStream baos = new ByteArrayOutputStream();
                for (int len = 0; (len = archive.read(buf)) > 0;) {
                    baos.write(buf, 0, len);
                }
                FileUtils.writeByteArrayToFile(file, baos.toByteArray());
            }
        }
    }
}
```



**3. ar，arj，cpio，dump，zip，jar**

这些压缩工具类的使用方式和tar基本类似，就不做示例了



## 修改归档文件 

有时候我们会有修改归档内文件的需求，比如添加、删除一个文件，修改其中的文件内容等，当然我们也可以全部解压出来改完后在压缩回去。这样除了代码量多一些外，归档文件大也会导致操作时间过长。那么有没有办法用代码去动态的修改归档文件里的内容呢？

org.apache.commons.compress.changes包下正好就提供了一些类用于动态的修改归档文件里的内容。下面看一个简单的例子



```java
String tarFile = "/test.tar";
InputStream is = new FileInputStream(tarFile);
// 替换后会覆盖原test.tar，如果是windows可能会由于文件被访问而覆盖报错
OutputStream os = new FileOutputStream(tarFile);
try (TarArchiveInputStream tais = new TarArchiveInputStream(is);
     TarArchiveOutputStream taos = new TarArchiveOutputStream(os)) {
    ChangeSet changes = new ChangeSet();
    // 删除"test.tar中"的"dir/1.txt"文件
    changes.delete("dir/1.txt");
    // 删除"test.tar"中的"t"目录
    changes.delete("t");
    // 添加文件，如果已存在则替换
    File addFile = new File("/a.txt");
    ArchiveEntry addEntry = taos.createArchiveEntry(addFile, addFile.getName());
    // add可传第三个参数：true: 已存在则替换(默认值)， false: 不替换
    changes.add(addEntry, new FileInputStream(addFile));
    // 执行修改
    ChangeSetPerformer performer = new ChangeSetPerformer(changes);
    ChangeSetResults result = performer.perform(tais, taos);
}
```



## 其他



### 简单工厂

commons-compress还提供了一些简单的工厂类用户动态的获取压缩流和归档流。

```java
// 使用factory动态获取归档流
ArchiveStreamFactory factory = new ArchiveStreamFactory();
String archiveName = ArchiveStreamFactory.TAR;
InputStream is = new FileInputStream("/in.tar");
OutputStream os = new FileOutputStream("/out.tar");
// 动态获取实现类，此时ais实际上是TarArchiveOutPutStream
ArchiveInputStream ais = factory.createArchiveInputStream(archiveName, is);
ArchiveOutputStream aos = factory.createArchiveOutputStream(archiveName, os);
// 其他业务操作

// ------------------------

// 使用factory动态获取压缩流
CompressorStreamFactory factory = new CompressorStreamFactory();
String compressName = CompressorStreamFactory.GZIP;
InputStream is = new FileInputStream("/in.gz");
OutputStream os = new FileOutputStream("/out.gz");
// 动态获取实现类，此时ais实际上是TarArchiveOutPutStream
CompressorInputStream cis = factory.createCompressorInputStream(compressName, is);
CompressorOutputStream cos = factory.createCompressorOutputStream(compressName, os);
// 其他业务操作
```



### 同时解压解包

上面说了很多都是单一的操作，那么如果解压"test.tar.gz"这种归档和压缩于一体的文件呢？

其实很简单，我们不需要先解压在解包，可以一步同时完成解压和解包，只需要将对应的流包装一下即可（不得不感叹Java IO的装饰者模式设计真的很巧妙）。下面看代码示例

```java
// 解压 解包test.tar.gz文件
String outPath = "/test";
InputStream is = new FileInputStream("/test.tar.gz");
// 先解压，所以需要先用gzip流包装文件流
CompressorInputStream gis = new GzipCompressorInputStream(is);
// 在解包，用tar流包装gzip流
try (ArchiveInputStream tgis = new TarArchiveInputStream(gis)) {
    ArchiveEntry nextEntry;
    while ((nextEntry = tgis.getNextEntry()) != null) {
        String name = nextEntry.getName();
        File file = new File(outPath, name);
        // 如果是目录，创建目录
        if (nextEntry.isDirectory()) {
            file.mkdir();
        } else {
            // 文件则写入具体的路径中
            FileUtils.copyToFile(tgis, file);
            file.setLastModified(nextEntry.getLastModifiedDate().getTime());
        }
    }
}
```



## 总结



除了以上介绍的工具类外，还有其他不是很常用的就不多做介绍了。感兴趣的可以自行翻阅源码研究。