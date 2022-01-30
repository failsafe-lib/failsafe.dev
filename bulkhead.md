---
layout: default
title: Bulkhead
---

# Bulkhead
{: .no_toc }

1. TOC
{:toc}

[Bulkheads][Bulkhead] allow you to restrict concurrent executions as a way of preventing system overload. Creating a [Bulkhead] is straightforward:

```java
// Permits 10 concurrent executions
Bulkhead<Object> bulkhead = Bulkhead.of(10);
```

## How It Works

Executions are permitted in a bulkhead until it is full, meaning the max number of concurrent executions has been reached. Any further executions will either fail with `BulkheadFullException ` or will wait until permitted.

## Waiting

By default, when the max concurrent executions are exceeded, further executions will immediately fail with `BulkheadFullException`. A bulkhead can also be configured to wait for execution permission if it can be achieved within a max wait time:

```java
// Wait up to 1 second for execution permission
Bulkhead<Object> bulkhead = Bulkhead.builder(10)
  .withMaxWaitTime(Duration.ofSeconds(1))
  .build();
```

Fairness is guaranteed with waiting executions, meaning they're permitted in the order they're received. Actual wait times for a bulkhead can vary depending on how busy it is. Since synchronous executions will block while waiting on a bulkhead, a `maxWaitTime` should be chosen carefully to avoid blocking too many threads. Async executions will not block while waiting on a bulkhead.

## Event Listeners

[Bulkhead] supports that standard [policy listeners][policy-listeners] that indicate when an execution attempt through the bulkhead succeeds or fails.

## Best Practices

A bulkhead can and *should* be shared across code that accesses finite resources. This ensures that if the bulkhead is exceeded, all executions that access the same resource and use the same bulkhead will either wait or fail until executions are permitted again. For example, if multiple connections or requests are made to the same external server, you may route them through the same bulkhead.

## Standalone Usage

A [Bulkhead] can also be manually operated in a standalone way:

```java
if (bulkhead.tryAcquirePermit()) {
  try {
    doSomething();
  } finally {
    bulkhead.releasePermit();
  }
}
```

{% include common-links.html %}