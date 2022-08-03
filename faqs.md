---
layout: default
title: Frequently Asked Questions
---

# Frequently Asked Questions

1. TOC
{:toc}

## How does Failsafe use threads?

Failsafe requires the use of separate threads for a few different situations:

- Async executions and retries
- Async `Fallback` calls
- `Timeout` checks
- Async `RateLimiter` and `Bulkhead` permit waiting

For async executions, retries, fallbacks, and rate limiter or bulkhead waiting, you can supply your own `ScheduledExecutorService`, `ExecutorService`, or `Scheduler` via `FailsafeExecutor.with()` which Failsafe will use. If none of these is configured, Failsafe will use the `ForkJoinPool.commonPool` for async scheduling. 

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
retryPolicyBuilder.handle(ConnectException.class);
```

If you have specific failure handling configuration and also want to handle `TimeoutExceededException`, be sure to configure it.

## Why is CircuitBreakerOpenException not being handled?

As with `TimeoutExceededException` described above, if you configure specific [result or failure handlers][FailurePolicyBuilder] you may need to ensure that `CircuitBreakerOpenException` is configured to be handled.

## How to I throw an exception when retries are exceeded?

When a `RetryPolicy` is exceeded, the last execution result or exception is returned or thrown. In the case that a result was returned but an exception is desired, the best approach is to wrap a RetryPolicy in a Fallback that converts a failed result into an exception:

```java
// Retry on a null result
RetryPolicy<Connection> retryPolicy = RetryPolicy.<Connection>builder()
  .handleResult(null)
  .build();
// Fallback on a null result to a ConnectException
Fallback<Connection> fallback = Fallback.<Connection>builderOfException(e -> {
    return new ConnectException("Connection failed after 3 attempts");
  })
  .handleResult(null)
  .build();

Failsafe.with(fallback).compose(retryPolicy).get(this::getConnection);
```

{% include common-links.html %}