---
layout: default
title: Frequently Asked Questions
---

# Frequently Asked Questions
{: .no_toc }

1. TOC
{:toc}

## How does Failsafe use threads?

Failsafe requires the use of separate threads for a few different situations:

- Async executions and retries
- Async `Fallback` calls
- `Timeout` checks

For async executions, async retries, and async fallbacks, you can supply your own `ScheduledExecutorService`, `Executor`, or `Scheduler` via `FailsafeExecutor.with()` which Failsafe will use. If no executor is configured, Failsafe will use the `ForkJoinPool.commonPool` for these purposes. 

`Timeout` checks are always performed using the `ForkJoinPool.commonPool` so that they are not blocked if the configured executor or Scheduler are fully utilized.

For async work that needs to be scheduled with a delay, Failsafe will either use a `ScheduledExecutorService` or `Scheduler` if one has been configured, else it will create a single threaded `ScheduledExecutorService` internally. This scheduler delegates executions to the configured executor or `commonPool`, will only be created if needed, and is shared across all Failsafe instances that don't configure a scheduler.

If no executor or scheduler is configured and the `ForkJoinPool.commonPool` has a parallelism of 1 (which occurs when the number of processors is <= 2), Failsafe will create and use an internal `ForkJoinPool` with a parallelism of 2 instead. This is necessary to support concurrent execution and `Timeout` checks. As with the internal `ScheduledExecutorService`, this  `ForkJoinPool` will only be created if needed, and will be shared across all Failsafe instances that don't configure an executor.

## Why is TimeoutExceededException not being handled?

It's common to use a `RetryPolicy` or `Fallback` together with a `Timeout`:

```java
Failsafe.with(fallback, retryPolicy, timeout).get(this::connect);
```

When you do, you may want to make sure that the `RetryPolicy` or `Fallback` are configured to handle `TimeoutExceededException`.

By default, a policy will handle all `Exception` types. But if you configure specific [result or failure handlers][FailurePolicyBuilder], then it may not recognize a `TimeoutExceededException` as a failure and may not handle it. Ex:

```java
// Only handles ConnectException, not TimeoutExceededException
retryPolicy.handle(ConnectException.class);
```

If you have specific failure handling configuration and also want to handle `TimeoutExceededException`, be sure to configure it.

## Why is CircuitBreakerOpenException not being handled?

As with `TimeoutExceededException` described above, if you configure specific [result or failure handlers][FailurePolicyBuilder] you may need to ensure that `CircuitBreakerOpenException` is configured to be handled.

{% include common-links.html %}