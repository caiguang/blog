---
title: Apache Commons简介
date: 2023-2-20
category: Java
tags: apache java 编程
---

### Apache Commons的三个组成部分

**Apache Commons**是Apache软件基金会的项目，曾隶属于Jakarta项目。Commons的目的是提供可重用的、开源的Java代码。

Apache Commons项目的由三部分组成：

- The Commons Proper - 一个可重用的Java组件库。(已经发布过的)
- The Commons Sandbox - Java组件开发工作区. (正在开发的项目)
- The Commons Dormant - 当前处于非活动状态的组件库.(刚启动或者已经停止维护的项目)

建立和维护可重用的Java组件。使用组件可以提高开发效率和质量。

---

### Commons Proper

**Commons Proper**

Commons Proper的目的是创建和维护可重用的Java组件库。Commons Proper是一个协作与共享的地方，Commons的开发者努力确保其组件对其他的软件库的依赖最少，以便可以轻松地部署这些组件。此外，Commons组件会尽可能的保持其接口的稳定，因而Apache用户以及其他Apache项目可以实现这些组件，而无需担心未来接口的变化。

以下部分组件节选自官方，详情请参见官网：[https://commons.apache.org/

| 组件          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| BCEL          | 字节码工程库——分析、创建和操作 Java 类文文件                 |
| BeanUtils     | 围绕 Java 反射和内省 API 的易于使用的包装器。                |
| CLI           | 命令行参数解析器。                                           |
| Codec         | 通用编码/解码算法（例如语音、base64、URL）。                 |
| Collections   | 扩展或增强 Java 集合框架。                                   |
| Compress      | 定义用于处理 tar、zip 和 bzip2 文件的 API。                  |
| Configuration | 读取各种格式的配置/首选项文件。                              |
| Crypto        | 使用 AES-NI 包装 Openssl 或 JCE 算法实现优化的加密库。       |
| CSV           | 用于读写逗号分隔值文件的组件。                               |
| Daemon        | unix-daemon-like java 代码的替代调用机制。                   |
| DBCP          | 数据库连接池服务。                                           |
| DbUtils       | JDBC 帮助程序库。                                            |
| Email         | 用于从 Java 发送电子邮件的库。                               |
| Exec          | 用于处理 Java 中外部进程执行和环境管理的 API。               |
| FileUpload    | 您的 servlet 和 Web 应用程序的文件上传功能。                 |
| Geometry      | 空间和坐标。                                                 |
| Imaging       | 纯 Java 图像库。                                             |
| IO            | I/O 实用程序的集合。                                         |
| JCI           | Java 编译器接口                                              |
| JCS           | Java缓存系统                                                 |
| Jelly         | 基于 XML 的脚本和处理引擎。                                  |
| Jexl          | 表达式语言，它扩展了 JSTL 的表达式语言。                     |
| Lang          | 为 java.lang 中的类提供额外的功能。                          |
| Logging       | 包装各种日志 API 实现。                                      |
| Math          | 轻量级、自包含的数学和统计组件。                             |
| Net           | 网络实用程序和协议实现的集合。                               |
| Numbers       | 数字类型（复数、四元数、分数）和实用程序（数组、组合）。     |
| Pool          | 通用对象池组件。                                             |
| RDF           | 可由 JVM 上的系统实现的 RDF 1.1 的通用实现。                 |
| RNG           | 随机数生成器的实现。                                         |
| Text          | Apache Commons Text 是一个专注于处理字符串的算法的库。       |
| Validator     | 在 xml 文件中定义验证器和验证规则的框架。                    |
| VFS           | 用于将文件、FTP、SMB、ZIP 等视为单个逻辑文件系统的虚拟文件系统组件。 |
| Weaver        | 提供一种简单的方法来增强（编织）已编译的字节码。             |

---

### Commons Sandbox

**Commons Sandbox**

Commons Sandbox是Java组件开发的工作区，在Sandbox中Commons的贡献者协作和检验那些被未列入Commons Proper的项目。Sandbox项目在Commons成员的支持下晋升为Commons Proper项目；大量的开发者协作强化Sandbox项目，直到它们匹配推广的标准。

可在Commons Sandbox项目页面上查看当前Commons Sandbox项目的列表。

---

### Commons Dormant

**Commons Dormant**

Commons Dormant是一个当前处于非活动状态的组件库。用户也可以使用这些组件，但必须自己进行组件的构建。一般而言，这些组件不会在近期发布。

可在Commons Dormant项目页面上查看当前Commons Dormant项目的列表。