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

Cancellation will cause any async execution retries and timeout attempts to stop. Interruption will cause the execution thread's [interrupt][interrupts] flag to be set.

## Cooperative Cancellation

Executions can cooperate with a cancellation by checking `ExecutionContext.isCancelled()`:

```java
Failsafe.with(timeout).getAsync(ctx -> {
  while (!ctx.isCancelled())
    doWork();
});
```

## Cooperative Interruption

Execution [interruption][interrupt] will cause certain blocking calls to unblock and may throw [InterruptedException] within your execution.

Non-blocking executions can cooperate with [interruption][interrupt] by periodically checking `Thread.isInterrupted()`:

```java
Failsafe.with(timeout).getAsync(()-> {
  while (!Thread.isInterrupted())
    doBlockingWork();
});
```

## Limitations

Since the [async integration][async-integration] methods involve external threads, which Failsafe has no knowledge of, these executions cannot be directly cancelled or interrupted by Failsafe. These executions can still [cooperate with cancellation][cooperative-cancellation] as described above, though they cannot cooperate with interruption.

## Interruption Support

Failsafe adds interruption support for any [ForkJoinPool][] that is configured as an [executor][withExecutorService], including the [common ForkJoinPool][common-pool] which is used by default. This means asynchronous tasks which are not normally interruptable outside of Failsafe can become interruptable when using Failsafe.

{% include common-links.html %}