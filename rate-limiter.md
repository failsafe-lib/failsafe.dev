---
layout: default
title: Rate Limiter
---

# Rate Limiter
{: .no_toc }

1. TOC
{:toc}

[Rate limiters][RateLimiter] allow you to control the rate of executions as a way of preventing system overload. Failsafe provides two types of rate limiting: *smooth* and *bursty*, which are discussed below.

## How It Works

When the number of executions through the rate limiter exceeds the configured max per time period, further executions will either fail with `RateLimitExceededException` or will wait until permitted.

## Smooth Rate Limiter

A *smooth* rate limiter permits a max number of executions per time period, using a leaky bucket approach to spread out executions at an even rate. Creating a smooth [RateLimiter] is straightforward:

```java
// Permits 100 executions per second
RateLimiter<Object> limiter = RateLimiter.smoothBuilder(100, Duration.ofSeconds(1)).build();
```

The rate at which individual executions are permitted is based on the given `maxExecutions` and `period`. Alternatively, you can directly specify the max rate of individual executions:

```java
// Permits an execution every 10 millis
RateLimiter<Object> limiter = RateLimiter.smoothBuilder(Duration.ofMillis(10)).build();
```

Smooth rate limited executions are permitted with no delay up to the max rate, and are always performed at or below the max rate, avoiding potential bursts.

## Bursty Rate Limiter

A *bursty* rate limiter uses a fixed window approach to permit a max number of executions for individual time periods. Creating a bursty [RateLimiter] is also straightforward:

```java
// Permits 10 executions every 1 second
RateLimiter<Object> limiter = RateLimiter.burstyBuilder(10, Duration.ofSeconds(1)).build();
```

Executions are permitted with no delay up to the given `maxExecutions` for the current period. When a new period begins, the number of permitted executions is reset to the configured `maxExecutions`. This may cause bursts of executions when a new time period begins. Larger time periods may cause larger bursts.

## Waiting

By default, when a [RateLimiter] is exceeded, further executions will immediately fail with `RateLimitExceededException`. A rate limiter can also be configured to wait for execution permission if it can be achieved within a max wait time:

```java
// Wait up to 1 second for execution permission
builder.withMaxWaitTime(Duration.ofSeconds(1));
```

Actual wait times for a rate limiter can vary depending on busy the rate limiter is. Wait times will grow if more executions are consistently attempted than the rate limiter permits. Since synchronous executions will block while waiting on a rate limiter, a `maxWaitTime` should be chosen carefully to avoid blocking too many threads. Async executions will not block while waiting on a rate limiter.

## Event Listeners

[RateLimiter] supports that standard [policy listeners][policy-listeners] that indicate when an execution attempt through the rate limiter succeeds or fails.

## Best Practices

A rate limiter can and *should* be shared across code that accesses common dependencies. This ensures that if the rate limit is exceeded, all executions that share the same dependency and use the same rate limiter will either wait or fail until executions are permitted again. For example, if multiple connections or requests are made to the same external server, typically they should all go through the same rate limiter.

## Standalone Usage

A [RateLimiter] can also be manually operated in a standalone way:

```java
if (rateLimiter.tryAcquirePermit()) {
  doSomething();
}
```

You can also integrate a standalone rate limiter with an external scheduler to wait for a permit asynchronously:

```java
Duration permitWaitTime = rateLimiter.reservePermit();
executor.schedule(this::sendRequest, permitWaitTime);
```

## Performance

Failsafe's internal [RateLimiter] implementation is efficient, with _O(1)_ time and space complexity.

{% include common-links.html %}