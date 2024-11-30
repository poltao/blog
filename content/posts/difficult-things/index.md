+++
title = "你遇到的最困难的x件事"
description = "本文是一篇指向性非常强的文章，讨论了工作中遇到的较为困难的x件事情。"
date = 2024-11-29
[taxonomies]
tags= ["Interview"]
+++

## 1. 使用错误的线程模型

这是一个由于使用 `CompletableFuture.runAsync` 发起 Dubbo 调用而导致的问题。由于 `CompletableFuture.runAsync` 在未指定线程池参数时默认会使用 `ForkJoinPool.commonPool`，而 `ForkJoinPool` 的线程模型是基于工作窃取（work-stealing）的，线程会主动寻找并执行任务，这种模型适用于计算密集型任务，如并行计算、递归分解等。所以在 Dubbo 调用时，拦截器会拒绝服务，导致调用失败，频繁告警（内部框架逻辑，Dubbo 本身并无这个限制）。

```java
public class AsyncRun {
    // 普通的线程池
    private static final Executor normalExecutor = new ThreadPoolExecutor(
            10, // 线程池中的线程数
            10, // 池中允许的最大线程数
            10L, // 当线程数大于核心线程数时，这是多余空闲线程在终止前等待新任务的最大时间
            TimeUnit.SECONDS, // keepAliveTime参数的时间单位
            new SynchronousQueue<>() // 在任务执行前用于保存任务的队列，此队列将仅保存由execute方法提交的Runnable任务
    );

    // 基于工作窃取线程模型的线程池
    private static final Executor workStealingPool = ForkJoinPool.commonPool();

    public static void main(String[] args) {
        List<String> list = Lists.newArrayList("1", "2", "3", "4", "5", "6", "7", "8", "9", "10");

        List<String> resultList = new CopyOnWriteArrayList<>();
        CompletableFuture<?>[] tasks = Lists.partition(list, 2).stream()
                .map(o -> CompletableFuture.runAsync(() -> {
                    resultList.add(StringUtils.join(o, "-"));
                }))
                .toArray(CompletableFuture[]::new);
        CompletableFuture.allOf(tasks).join();
        for (String s : resultList) {
            System.out.println(s);
        }
        // close pool
        ((ThreadPoolExecutor) normalExecutor).shutdown();
        ((ForkJoinPool) workStealingPool).shutdown();
    }
}
```

上面的例子，简单演示了一波，使用 `CompletableFuture.runAsync` 批量运行多个异步任务的场景。在实际的业务场景中，我们可能会使用 `CompletableFuture.runAsync` 发起 Dubbo 调用，这时候就会遇到上面提到的问题。而解决这个问题的方法也很简单，只需要指定线程池参数即可。

```java
CompletableFuture<?> future = CompletableFuture.runAsync(() -> {
            for (String s : list) {
                System.out.println(s);
            }
        }, normalExecutor);
future.join();
```

## 2. 使用 Log4j2 的打印大对象导致 CPU 飙高

在使用 Log4j2 记录日志时，通常会使用 Jackson 的 `ObjectMapper` 将对象序列化为 JSON 字符串，然后再打印到日志文件中。但是，当对象过大时，序列化的过程会消耗大量的 CPU 资源，导致 CPU 飙高。

```java
public class Log4j2Test {
    private static final Logger logger = LogManager.getLogger(Log4j2Test.class);

    @Data
    @AllArgsConstructor
    public static class Value {
        private String name;
        private int age;
    }

    public static String toJson(Object obj) {
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            return objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            logger.error("toJson error", e);
        }
        return null;
    }

    public static void main(String[] args) {
        List<Value> list = Lists.newArrayList(new Value("张三", 18), new Value("李四", 20));
        logger.info("list:{}", toJson(list));
    }
}
```

如上面代码所示，当 `list` 中的元素过多时，序列化的过程会消耗大量的 CPU 资源，导致 CPU 飙高。所以日常开发中，我们应该避免在日志中打印大对象。
最后对于 Java 程序，可以使用[async-profiler](https://github.com/async-profiler/async-profiler)这个低开销的采样分析器，来定位 CPU 飙高的问题。`async-profiler` 是一个功能强大的 Java 分析工具，通过低开销的采样技术和 HotSpot 特定 API，解决了传统分析器的 Safepoint 偏差问题。它不仅能够分析 Java 线程，还能监控非 Java 线程，并提供多维度的性能分析，适用于各种基于 HotSpot JVM 的 Java 运行时环境。

## 3. Java Dubbo 服务使用 Sentinel 进行限流及排队等待策略

在分布式系统中，服务限流是保障系统稳定性和可用性的重要手段之一。Java Dubbo 服务可以通过集成 Alibaba 开源的 [Sentinel 框架](https://github.com/alibaba/Sentinel)，实现对发起调用方的限流，并在达到限流值时，采用排队等待策略，以避免系统过载。

## 4. 使用 redis Pipeline 方案，减少网络延迟和提高吞吐量

- **批量请求-响应**：管道操作允许客户端一次性发送多个命令到 Redis 服务器，然后一次性接收所有命令的响应。这种方式减少了网络往返次数，从而降低了网络延迟。
- **减少网络开销**：通过减少网络往返次数，管道操作显著降低了网络开销，提高了整体性能。

```java
Jedis jedis = new Jedis("localhost", 6379);
Pipeline pipeline = jedis.pipelined();

pipeline.get("key1");
pipeline.get("key2");
pipeline.get("key3");

List<Object> responses = pipeline.syncAndReturnAll();
for (Object response : responses) {
    System.out.println(response);
}

jedis.close();
```

通过合理选择和使用这两种操作方式，可以在不同的场景下优化 Redis 的性能和响应速度。
