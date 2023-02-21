---
title: Apache Commons CLI
date: 2023-2-21
category: Java
tags: apache-commons
excerpt: Apache Commons CLI 是 Apache 下面的一个解析命令行输入的工具包，该工具包还提供了自动生成输出帮助文档的功能。
---
## Apache Commons CLI 简介

Apache Commons CLI 是 Apache 下面的一个解析命令行输入的工具包，该工具包还提供了自动生成输出帮助文档的功能。  

Apache Commons CLI 支持多种输入参数格式，主要支持的格式有以下几种：

- POSIX（Portable Operating System Interface of Unix）中的参数形式，例如 `tar -zxvf foo.tar.gz`
- GNU 中的长参数形式，例如 `du --human-readable --max-depth=1`
- Java 命令中的参数形式，例如 `java -Djava.net.useSystemProxies=true Foo`
- 短杠参数带参数值的参数形式，例如 `gcc -O2 foo.c`
- 长杠参数不带参数值的形式，例如 `ant – projecthelp`

CLI 命令代码实现

命令行程序处理流程相对比较简单，主要流程为设定命令行参数 -> 解析输入参数 -> 使用输入的数据进行逻辑处理



## CLI 定义阶段

​    每一条命令行都必须定义一组参数，它们被用来定义应用程序的接口。Apache Commons CLI 使用 Options 这个类来定义和设置参数，它是所有 Option 实例的容器。在 CLI 中，目前有两种方式来创建 Options，一种是通过构造函数，这是最普通也是最为大家所熟知的一种方式；另外一种方法是通过 Options 中定义的工厂方式来实现。

​    CLI 定义阶段的目标结果就是创建 Options 实例。

```java
// 创建 Options 对象
 Options options = new Options(); 

 // 添加 -h 参数
 options.addOption("h", false, "Lists short help"); 

 // 添加 -t 参数
 options.addOption("t", true, "Sets the HTTP communication protocol for CIM connection");
```



​    其中 addOption() 方法有三个参数，第一个参数设定这个 option 的单字符名字，第二个参数指明这个 option 是否需要输入数值，第三个参数是对这个 option 的简要描述。在这个代码片段中，第一个参数只是列出帮助文件，不需要用户输入任何值，而第二个参数则是需要用户输入 HTTP 的通信协议，所以这两个 option 的第二个参数分别为 false 和 true





## CLI 解析阶段

​    在解析阶段中，通过命令行传入应用程序的文本来进行处理。处理过程将根据在解析器的实现过程中定义的规则来进行。在 CommandLineParser 类中定义的 parse 方法将用 CLI 定义阶段中产生的 Options 实例和一组字符串作为输入，并返回解析后生成的 CommandLine。

​    CLI 解析阶段的目标结果就是创建 CommandLine 实例。

```java
CommandLineParser parser = new PosixParser(); 
 CommandLine cmd = parser.parse(options, args); 

 if(cmd.hasOption("h")) { 
    // 这里显示简短的帮助信息
 }
```







## CLI 询问阶段

​    在询问阶段中，应用程序通过查询 CommandLine，并通过其中的布尔参数和提供给应用程序的参数值来决定需要执行哪些程序分支。这个阶段在用户的代码中实现，CommandLine 中的访问方法为用户代码提供了 CLI 的询问能力。

​    CLI 询问阶段的目标结果就是将所有通过命令行以及处理参数过程中得到的文本信息传递给用户的代码。

```java
commandLine = parser.parse(options, args);
            if (commandLine.hasOption('h')) {
                //打印使用帮助
                hf.printHelp("testApp", options, true);
            }
```





## 代码示例 

```java
import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.CommandLineParser;
import org.apache.commons.cli.HelpFormatter;
import org.apache.commons.cli.Option;
import org.apache.commons.cli.Options;
import org.apache.commons.cli.ParseException;
import org.apache.commons.cli.PosixParser;

public class Test {
    public static void main(String[] args) {
        String[] arg = { "-h", "-c", "config.xml" };
        testOptions(arg);
    }


    public static void testOptions(String[] args) {
        Options options = new Options();
        Option opt = new Option("h", "help", false, "Print help");
        opt.setRequired(false);
        options.addOption(opt);

        opt = new Option("c", "configFile", true, "Name server config properties file");
        opt.setRequired(false);
        options.addOption(opt);

        opt = new Option("p", "printConfigItem", false, "Print all config item");
        opt.setRequired(false);
        options.addOption(opt);

        HelpFormatter hf = new HelpFormatter();
        hf.setWidth(110);
        CommandLine commandLine = null;
        CommandLineParser parser = new PosixParser();
        try {
            commandLine = parser.parse(options, args);
            if (commandLine.hasOption('h')) {
                // 打印使用帮助
                hf.printHelp("testApp", options, true);
            }

            // 打印opts的名称和值
            System.out.println("--------------------------------------");
            Option[] opts = commandLine.getOptions();
            if (opts != null) {
                for (Option opt1 : opts) {
                    String name = opt1.getLongOpt();
                    String value = commandLine.getOptionValue(name);
                    System.out.println(name + "=>" + value);
                }
            }
        }
        catch (ParseException e) {
            hf.printHelp("testApp", options, true);
        }
    }
}
```