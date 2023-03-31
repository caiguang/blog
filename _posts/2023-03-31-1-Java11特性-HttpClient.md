---
title: JDK11新特性-HttpClient
date: 2023-3-31
category: java
tags: java
excerpt: JDK9 的时候，Java 提供了一个新的 Http 请求工具 HttpClient，经过了 JDK10 的再次预览，最终在 JAVA11 中作为正式功能提供使用，同时也完全替换了仅有阻塞模式的`HttpURLConnection`。
---

 JDK9 的时候，Java 提供了一个新的 Http 请求工具 HttpClient，经过了 JDK10 的再次预览，最终在 JAVA11 中作为正式功能提供使用，同时也完全替换了仅有阻塞模式的`HttpURLConnection`。

# HttpClient 简介

作为 JDK11 中正式推出的新 Http 连接器，支持的功能还是比较新的，主要的特性有：



- 完整支持 HTTP 2.0 或者 HTTP 1.1
- 支持 HTTPS/TLS
- 有简单的阻塞使用方法
- 支持异步发送，异步时间通知
- 支持 WebSocket
- 支持响应式流



HTTP2.0 其他的客户端也能支持，而 HttpClient 使用 CompletableFuture 作为异步的返回数据。WebSocket 的支持则是 HttpClient 的优势。响应式流的支持是 HttpClient 的一大优势。



而 HttpClient 中的 NIO 模型、函数式编程、CompletableFuture 异步回调、响应式流让 HttpClient 拥有极强的并发处理能力，所以其性能极高，而内存占用则更少。



HttpClient 的主要类有:



- java.net.http.HttpClient
- java.net.http.HttpRequest
- java.net.http.HttpResponse
- java.net.http.WebSocket（本文就不介绍这个了）



细节会在后文介绍，但是 WebSocket 用的比较少，本文就略过了。

# 核心使用

HttpClient 的核心类主要就是 HttpClient、HttpRequest 以及 HttpResponse，它们都是位于 java.net.http 包下，接下来对他们进行一下介绍。

## HttpClient

HttpClient 类是最核心的类，它支持使用建造者模式进行复杂对象的构建，主要的参数有：



- Http 协议的版本 (HTTP 1.1 或者 HTTP 2.0)，默认是 2.0。
- 是否遵从服务器发出的重定向
- 连接超时时间
- 代理
- 认证



```java
//可以用参数调整
HttpClient client = HttpClient.newBuilder()
                         .version(Version.HTTP_1_1)
                         .followRedirects(Redirect.NORMAL)
                         .connectTimeout(Duration.ofSeconds(20))
                         .proxy(ProxySelector.of(new InetSocketAddress("proxy.example.com", 8080)))
                         .authenticator(Authenticator.getDefault())
                         .build();
                         
//也可以直接全部默认的便捷创建
HttpClient clientSimple = HttpClient.newHttpClient();

```



当创建了 HttpClient 实例后，可以通过其发送多条请求，不用重复创建。

## HttpRequest

HttpRequest 是用语描述请求体的类，也支持通过建造者模式构建复杂对象，主要的参数有：



- 请求地址
- 请求方法:GET,POST,DELETE 等（默认是 GET）
- 请求体 (按需设置，例如 GET 不用 body,但是 POST 要设置)
- 请求超时时间（默认）
- 请求头



```java
//使用参数组合进行对象构建，读取文件作为请求体
HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://www.baidu.com"))
                .timeout(Duration.ofSeconds(20))
                .header("Content-type","application/json")
                .POST(HttpRequest.BodyPublishers.ofFile(Paths.get("data.json")))
                .build();
//直接GET访问
HttpRequest requestSimple = HttpRequest.newBuilder(URI.create("http://www.baidu.com")).build();

```



HttpRequest 是一个不可变类，可以被多次发送。

## HttpResponse

HttpResponse 没有提供外部可以创建的实现类，它是一个接口，从 client 的返回值中创建获得。接口中的主要方法为：



```java
public interface HttpResponse<T> {
        public int statusCode();
        public HttpRequest request();
        public Optional<HttpResponse<T>> previousResponse();
        public HttpHeaders headers();
        public T body();
        public URI uri();
        public Optional<SSLSession> sslSession();
        public HttpClient.Version version();
    }

```



HttpResponse 的返回内容与常识一致，这里就不展开介绍了。

## 信息发送

HttpClient 中可以使用同步发送或者异步发送。



**同步 send()**



同步发送后，请求会一直阻塞到收到 response 为止。



```java
final HttpResponse<String> send = client.send(httpRequest, HttpResponse.BodyHandlers.ofString());
System.out.println(send.body());

```



其中 send 的第二个参数是通过`HttpResponse.BodyHandlers`的静态工厂来返回一个可以将 response 转换为目标类型`T`的处理器（handler），本例子中的类型是 String。`HttpResponse.BodyHandlers.ofString()`的实现方法为：



```java
public static BodyHandler<String> ofString() {
            return (responseInfo) -> BodySubscribers.ofString(charsetFrom(responseInfo.headers()));
}

```



其中，BodySubscribers.ofString() 的方法实现是：



```java
public static BodySubscriber<String> ofString(Charset charset) {
    Objects.requireNonNull(charset);
    return new ResponseSubscribers.ByteArraySubscriber<>(
           bytes -> new String(bytes, charset)
    );
}

```



可以看到最终是返回了一个 ResponseSubscribers ，而 Subscribers 则是我们之前《JDK9 响应式编程》中讨论过的订阅者。这个构造方法的入参`Function<byte[],T>定义了订阅者中的`finisher`属性，而这个属性将在响应式流完成订阅的时在`onComplete()`方法中调用。



**异步 sendAsync()**



异步请求发送之后，会立刻返回 CompletableFuture，然后可以使用 CompletableFuture 中的方法来设置异步处理器。



```java
client.sendAsync(httpRequest, HttpResponse.BodyHandlers.ofString())
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println)
        .join();

```



而就如同 JDK 中响应式流中发布者的 submit()方法与 offer()方法一样，HttpClient 中的 send()方法知识 sendAsync 方法的特例，在 send()方法中是先调用 sendAsync()方法，然后直接阻塞等待响应结束再返回，部分核心代码为：



```java
    @Override
    public <T> HttpResponse<T>
    send(HttpRequest req, BodyHandler<T> responseHandler)
        throws IOException, InterruptedException
    {
        CompletableFuture<HttpResponse<T>> cf = null;

        // if the thread is already interrupted no need to go further.
        // cf.get() would throw anyway.
        if (Thread.interrupted()) throw new InterruptedException();
        try {
            cf = sendAsync(req, responseHandler, null, null);
            return cf.get();
        } catch (InterruptedException ie) {
            if (cf != null )
                cf.cancel(true);
            throw ie;
        }
      ...

```



**响应式流**



HttpClient 作为 Request 的发布者 (publisher),将 Request 发布到服务器，作为 Response 的订阅者 (subscriber),从服务器接收 Response。而上文中我们在 send()的部分发现，调用链的最底端返回的是一个`ResponseSubscribers`订阅者。



当然,就如同`HttpResponse.BodyHandlers.ofString()`,HttpClient 默认提供了一系列的默认订阅者，用语处理数据的转换：



```java
HttpRequest.BodyPublishers::ofByteArray(byte[])
HttpRequest.BodyPublishers::ofByteArrays(Iterable)
HttpRequest.BodyPublishers::ofFile(Path)
HttpRequest.BodyPublishers::ofString(String)
HttpRequest.BodyPublishers::ofInputStream(Supplier<InputStream>)

HttpResponse.BodyHandlers::ofByteArray()
HttpResponse.BodyHandlers::ofString()
HttpResponse.BodyHandlers::ofFile(Path)
HttpResponse.BodyHandlers::discarding()

```



所以在 HttpClient 的时候我们也可以自己创建一个实现了`Flow.Subscriber<List<ByteBuffer>>`接口的订阅者，用于消费数据。响应式流完整的简单的例子如下：



```java
public class HttpClientTest {

    public static void main(String[] args) throws IOException, InterruptedException {
        final HttpClient client = HttpClient.newHttpClient();
        final HttpRequest httpRequest = HttpRequest.newBuilder(URI.create("http://www.baidu.com")).build();

        HttpResponse.BodySubscriber<String> subscriber = HttpResponse.BodySubscribers.fromSubscriber(new StringSubscriber(),StringSubscriber::getBody);
        client.sendAsync(httpRequest,responseInfo -> subscriber)
                .thenApply(HttpResponse::body)
                .thenAccept(System.out::println)
                .join();
    }

    static class StringSubscriber implements Flow.Subscriber<List<ByteBuffer>>{
        Flow.Subscription subscription;
        List<ByteBuffer> response = new ArrayList<>();
        String body;
        public String getBody(){
            return body;
        }

        @Override
        public void onSubscribe(Flow.Subscription subscription) {
            this.subscription = subscription;
            subscription.request(1);
        }

        @Override
        public void onNext(List<ByteBuffer> item) {
            response.addAll(item);
            subscription.request(1);
        }

        @Override
        public void onError(Throwable throwable) {
            System.err.println(throwable);
        }

        @Override
        public void onComplete() {
            byte[] data = new byte[response.stream().mapToInt(ByteBuffer::remaining).sum()];
            int offset = 0;
            for(ByteBuffer buffer:response){
                int remain = buffer.remaining();
                buffer.get(data,offset,remain);
                offset += remain;
            }
            body = new String(data);
        }
    }
}

```

# 最后

HttpClient 是 JDK11 正式上线的高性能 Http 客户端。其底层基于响应式流，通过上层封装还提供了异步信息发送、同步信息发送，以及其他完成的 HTTP 协议内容。在进行响应式编程的方面，HttpClient 也是一个十分优秀的参照目标。