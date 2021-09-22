---
layout: default
title: Async Execution
---

# Async Execution
{: .no_toc }

1. TOC
{:toc}

In addition to synchronous execution, Failsafe provides several ways to perform asynchronous executions or integrate with async code.

## Simple Async Execution

Async executions can be performed by using the [runAsync] or [getAsync] methods, which return a [CompletableFuture]:

```java
CompletableFuture<Connection> future = Failsafe.with(retryPolicy).getAsync(this::connect);
```

## CompletionStage Integration

Code that returns a [CompletionStage] can be integrated with Failsafe by using the [getStageAsync] method, which accepts a [CompletionStage][CompletionStage] [Supplier][Supplier] and returns a new [CompletableFuture] with failure handling built-in:

```java
Failsafe.with(retryPolicy)
  .getStageAsync(this::connectAsync)
  .thenAccept(System.out::println));
```

## Async Integration

Failsafe can also integrate with threads that are outside of its control. The [runAsyncExecution], [getAsyncExecution] and [getStageAsyncExecution] methods provide an [AsyncExecution] object that can be used to record execution results from another thread:

```java
Failsafe.with(retryPolicy)
  .getAsyncExecution(execution -> service.connect().whenComplete((result, failure) -> {
    if (execution.complete(result, failure))
      log.info("Connected");
    else if (!execution.retry())
      log.error("Connection attempts failed", failure);
  }));
```

## Executor Configuration

By default, Failsafe performs async executions using the [ForkJoinPool]'s [common pool][common-pool], but you can also configure a specific [ScheduledExecutorService], [ExecutorService], or [Executor] to use:

```java
Failsafe.with(policy).with(executor).getAsync(this::connect);
```

## Custom Schedulers

Failsafe can integrate with libraries that use their own schedulers for async executions, such as [Akka](https://akka.io) or [Vert.x](https://vertx.io), by supporting a custom [Scheduler]:

```java
Failsafe.with(policy).with(akkaScheduler).getAsync(this::connect);
```

See the [Vert.x example][Vert.x] of a custom scheduler implementation.

{% include common-links.html %}