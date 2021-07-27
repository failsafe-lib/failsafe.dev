---
layout: default
title: Execution Cancellation
---

# Execution Cancellation
{: .no_toc }

1. TOC
{:toc}

Failsafe supports cancellation and optional interruption of executions. Cancellation and interruption can be triggered by a [Timeout][timeouts] or through an async execution's [Future]:

```java
Future<Connection> future = Failsafe.with(retryPolicy).getAsync(this::connect);
future.cancel(shouldInterrupt);
```

Cancellation will cause any async execution retries and timeout attempts to stop. Interruption will cause the execution thread's [interrupt] flag to be set.

## Handling Cancellation

Executions can cooperate with a cancellation by checking `ExecutionContext.isCancelled()`:

```java
Failsafe.with(timeout).getAsync(ctx -> {
  while (!ctx.isCancelled())
    doWork();
});
```

## Handling Interruption

Execution [interruption][interrupt] will cause certain blocking calls to unblock and may throw [InterruptedException] within your execution.

Non-blocking executions can cooperate with [interruption][interrupt] by periodically checking `Thread.isInterrupted()`:

```java
Failsafe.with(timeout).getAsync(ctx -> {
  while (!Thread.isInterrupted())
    doBlockingWork();
});
```

## Interruption Support

Failsafe adds interruption support for any [ForkJoinPool][] that is configured as a [scheduler][schedulers], including the [common ForkJoinPool][common-pool] which is used by default. This means asynchronous tasks which are not normally interruptable outside of Failsafe can become interruptable when using Failsafe. 

The one exception is when using the [async API integration][async-api] methods. Since those methods depend on external executors, which Failsafe has no knowledge of, those external tasks cannot be cancelled or interrupted by Failsafe.

{% include common-links.html %}