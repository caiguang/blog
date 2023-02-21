---
title: Apache Commons Exec
date: 2023-2-21
category: Java
tags: apache-commons
excerpt: Apache Commons Exec主要用于执行外部进程的命令。Exec目前最新版本是1.3，最低要求Java5以上。用Java执行外部进程命令也是比较常见的一种需求，这种操作依赖特定操作系统，需要我们了解特定系统的行为，例如在Windows上使用cmd.exe。想要可靠地执行外部进程还需要在执行命令之前或之后处理环境变量。
---
**Apache Commons Exec**主要用于执行外部进程的命令。Exec目前最新版本是1.3，最低要求Java5以上。

用Java执行外部进程命令也是比较常见的一种需求，这种操作依赖特定操作系统，需要我们了解特定系统的行为，例如在Windows上使用cmd.exe。想要可靠地执行外部进程还需要在执行命令之前或之后处理环境变量。

**Apache Commons Exec**就是为了处理上面概述的各种问题。而且代码实现起来也比较简单。

## maven坐标

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-exec</artifactId>
    <version>1.3</version>
</dependency>
```

下面简单介绍一下用法。



## 同步调用



同步调用系统命令后会阻塞当前线程，直到获取到结果。



### 使用JDK写法

```java
// 不使用工具类的写法
Process process = Runtime.getRuntime().exec("cmd /c ping 192.168.1.10");
int exitCode = process.waitFor(); // 阻塞等待完成
if (exitCode == 0) { // 状态码0表示执行成功
    String result = IOUtils.toString(process.getInputStream()); // "IOUtils" commons io中的工具类，详情可以参见前续文章介绍
    System.out.println(result);
} else {
    String errMsg = IOUtils.toString(process.getErrorStream());
    System.out.println(errMsg);
}
```



等等，这么写其实有坑。如果执行一个安装脚本会在控制台输出大量内容，这时可能会导致进程卡死（其实是一直阻塞状态）。

这是由于缓冲区满了，无法写入数据，导致线程阻塞，对外现象就是进程无法停止，也不占资源，什么反应也没有。

这种情况可以单独启动一个线程去读取输入流的内容，避免缓冲区占满，示例如下：



```java
final Process process = Runtime.getRuntime().exec("cmd /c ping 192.168.1.10");
new Thread(() -> {
    try (BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()))) {
        String line;
        while ((line = br.readLine()) != null) {
            try {
                process.exitValue();
                break; // exitValue没有异常表示进程执行完成，退出循环
            } catch (IllegalThreadStateException e) {
                // 异常代表进程没有执行完
            }
            //此处只做输出，对结果有其他需求可以在主线程使用其他容器收集此输出
            System.out.println(line);
        }
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}).start();
process.waitFor();
```

如果是异常信息打印过多则处理process.getErrorStream()。



### 使用commons写法



commons-exec的command不需要考虑执行环境了，比如windows下不需要添加"cmd /c "的前缀。可以使用自定义的流来接受结果，比如使用文件流将结果保存到文件，使用网络流保存到远程服务器上等。下面的例子为了简单直接使用字节流去接收（如果结果非常大就不要用字节流了，容易内容溢出）。



```java
String command = "ping 192.168.1.10";
//接收正常结果流
ByteArrayOutputStream susStream = new ByteArrayOutputStream();
//接收异常结果流
ByteArrayOutputStream errStream = new ByteArrayOutputStream();
CommandLine commandLine = CommandLine.parse(command);
DefaultExecutor exec = new DefaultExecutor();
PumpStreamHandler streamHandler = new PumpStreamHandler(susStream, errStream);
exec.setStreamHandler(streamHandler);
int code = exec.execute(commandLine);
System.out.println("result code: " + code);
// 不同操作系统注意编码，否则结果乱码
String suc = susStream.toString("GBK");
String err = errStream.toString("GBK");
System.out.println(suc);
System.out.println(err);
```



## 异步调用



### 使用JDK写法

JDK自带的Runtime的API不支持异步执行，如果要异步拿到执行结果需要自己单独创建线程不断轮询进程状态然后通知主线程，下面看一个例子。例子力求简单，所以很多细节不是很严谨，只看大体思路即可（如果要实现exec方便的API需要更多的代码来实现）。



```java
public class RuntimeAsyncDemo {

    public static void main(String[] args) throws Exception {
        System.out.println("1. 开始执行");
        String cmd = "cmd /c ping 192.168.1.11"; // 假设是一个耗时的操作
        execAsync(cmd, processResult -> {
            System.out.println("3. 异步执行完成，success=" + processResult.success + "; msg=" + processResult.result);
            System.exit(0);
        });
        // 做其他操作 ... ...
        System.out.println("2. 做其他操作");
        // 避免主线程退出导致程序退出
        Thread.currentThread().join();
    }
    private static void execAsync(String command, Consumer<ProcessResult> callback) throws IOException {
        final Process process = Runtime.getRuntime().exec(command);
        new Thread(() -> {
            StringBuilder successMsg = new StringBuilder();
            try (BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream(), "GBK"))) {
                // 存放临时结果
                String line;
                while ((line = br.readLine()) != null) {
                    try {
                        successMsg.append(line).append("\r\n");
                        int exitCode = process.exitValue();
                        ProcessResult pr = new ProcessResult();
                        if (exitCode == 0) {
                            pr.success = true;
                            pr.result = successMsg.toString();
                        } else {
                            pr.success = false;
                            pr.result = IOUtils.toString(process.getErrorStream());
                        }
                        callback.accept(pr); // 回调主线程注册的函数
                        break; // exitValue没有异常表示进程执行完成，退出循环
                    } catch (IllegalThreadStateException e) {
                        // 异常代表进程没有执行完
                    }
                    try {
                        // 等待100毫秒在检查是否完成
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }).start();
    }

    private static class ProcessResult {
        boolean success;
        String result;
    }
}
```



### 使用commons写法

commons-exec原生支持异步调用，下面直接看例子。



```java
@Test
public void execAsync() throws IOException, InterruptedException {
    String command = "ping 192.168.1.10";
    //接收正常结果流
    ByteArrayOutputStream susStream = new ByteArrayOutputStream();
    //接收异常结果流
    ByteArrayOutputStream errStream = new ByteArrayOutputStream();
    CommandLine commandLine = CommandLine.parse(command);
    DefaultExecutor exec = new DefaultExecutor();

    PumpStreamHandler streamHandler = new PumpStreamHandler(susStream, errStream);
    exec.setStreamHandler(streamHandler);
    ExecuteResultHandler erh = new ExecuteResultHandler() {
        @Override
        public void onProcessComplete(int exitValue) {
            try {
                String suc = susStream.toString("GBK");
                System.out.println(suc);
                System.out.println("3. 异步执行完成");
            } catch (UnsupportedEncodingException uee) {
                uee.printStackTrace();
            }
        }
        @Override
        public void onProcessFailed(ExecuteException e) {
            try {
                String err = errStream.toString("GBK");
                System.out.println(err);
                System.out.println("3. 异步执行出错");
            } catch (UnsupportedEncodingException uee) {
                uee.printStackTrace();
            }
        }
    };
    System.out.println("1. 开始执行");
    exec.execute(commandLine, erh);
    System.out.println("2. 做其他操作");
    // 避免主线程退出导致程序退出
    Thread.currentThread().join();
}
```



## 监控

commons-exec支持监控外部进程的执行状态并做一些操作，如超时，停止等。

在使用Runtime.getRuntime().exec(cmd)执行某些系统命令，如nfs共享的mount时，会由于nfs服务异常等原因导致进程阻塞，使程序没法往下执行，而且也无法捕获到异常，相当于卡死在了。这时如果有超时放弃的功能就好了，当然超时功能可以自己轮询process.exitValue()去实现，稍微麻烦一些，这里就不做示例了。

commons-exec主要通过**ExecuteWatchdog**类来处理超时，下面看例子



```java
String command = "ping 192.168.1.10";
ByteArrayOutputStream susStream = new ByteArrayOutputStream();
ByteArrayOutputStream errStream = new ByteArrayOutputStream();
CommandLine commandLine = CommandLine.parse(command);
DefaultExecutor exec = new DefaultExecutor();
//设置一分钟超时
ExecuteWatchdog watchdog = new ExecuteWatchdog(60*1000);
exec.setWatchdog(watchdog);
PumpStreamHandler streamHandler = new PumpStreamHandler(susStream, errStream);
exec.setStreamHandler(streamHandler);
try {
    int code = exec.execute(commandLine);
    System.out.println("result code: " + code);
    // 不同操作系统注意编码，否则结果乱码
    String suc = susStream.toString("GBK");
    String err = errStream.toString("GBK");
    System.out.println(suc+err);
} catch (ExecuteException e) {
    if (watchdog.killedProcess()) {
        // 被watchdog故意杀死
        System.err.println("超时了");
    }
}
```

ExecuteWatchdog还支持销毁进程，只需调用destroyProcess()，由于ExecuteWatchdog是异步执行的，所以调用后不会马上停止。使用起来也比较简单就不做说明了。



## 总结

commons-exec屏蔽了不同操作系统的命令差异，解决了Runtime缓冲区问题导致的线程卡死，同时支持超时和等，用来代替JDK的Runtime API是非常不错的选择。