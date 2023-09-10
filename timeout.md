---
layout: default
title: Timeout
---

# Timeout
{: .no_toc }

1. TOC
{:toc}

[Timeouts][Timeout] allow you to fail an execution with `TimeoutExceededException` if it takes too long to complete:

```java
Timeout<Object> timeout = Timeout.of(Duration.ofSeconds(10));
```

You can create a Timeout that [interrupts][interrupt] execution if it times out:

```java
Timeout<Object> timeout = Timeout.builder(Duration.ofSeconds(10)).withInterrupt().build();
```

If a cancellation is triggered by a `Timeout`, the execution is completed with `TimeoutExceededException`. See the [execution cancellation][execution-cancellation] page for more on cancellation and interruption.

## Timeouts with Retries

When a `Timeout` is [composed][policy-composition] _outside_ a `RetryPolicy`, a timeout occurrence will cancel any _inner_ retries:

```java
Failsafe.with(timeout).compose(retryPolicy).run(this::connect);
```

When a `Timeout` is [composed][policy-composition] _inside_ a `RetryPolicy`, a timeout occurrence will not automatically cancel any _outer_ retries:

```java
Failsafe.with(retryPolicy).compose(timeout).run(this::connect);
```

## Async Execution Timeouts

When an async executions times out, Failsafe still waits until the execution completes, either through interruption or naturally, before recording a `TimeoutExceededException`. This avoids the risk of retrying while the original execution is still running, which ultimately could lead to numerous abandoned executions running in numerous threads.

## Event Listeners

[Timeouts][Timeout] support the standard [policy listeners][PolicyListeners] which can notify you when a timeout is exceeded:

```java
builder.onFailure(e -> log.error("Connection attempt timed out", e.getException()));
```

Or when an execution completes and the timeout is not exceeded:

```java
builder.onSuccess(e -> log.info("Execution completed on time"));
```

## Limitations

Since the [async integration][async-integration] methods involve external threads, which Failsafe has no knowledge of, these executions cannot be directly cancelled or interrupted by a Timeout. But, these executions can still [cooperate][cooperative-cancellation] with a Timeout cancellation.

{% include common-links.html %}