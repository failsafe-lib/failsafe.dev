---
layout: default
title: Retry
---

# Retry Policy
{: .no_toc }

1. TOC
{:toc}

[Retry policies][RetryPolicy] express when retries should be performed for an execution.

## Attempts

By default, a [RetryPolicy] will perform a maximum of 3 execution attempts. You can configure a different max number of [attempts][max-attempts]:

```java
builder.withMaxAttempts(3);
```

Or a max number of [retries][max-retries]

```java
builder.withMaxRetries(2);
```

## Delays

By default, a [RetryPolicy] has no delay between attempts. You can configure a fixed delay:

```java
builder.withDelay(Duration.ofSeconds(1));
```

Or a delay that [backs off][backoff] exponentially:

```java
builder.withBackoff(1, 30, ChronoUnit.SECONDS);
```

A [random delay][random-delay] for some range:

```java
builder.withDelay(1, 10, ChronoUnit.SECONDS);
```

Or a [computed delay][computed-delay] based on an execution result or failure.

### Jitter

You can also combine a random [jitter factor][jitter-factor] with a delay:

```java
builder.withJitter(.1);
```

Or a [time based jitter][jitter-duration]:

```java
builder.withJitter(Duration.ofMillis(100));
```

## Duration

You can add a [max duration][max-duration] for an execution, after which retries will stop:

```java
builder.withMaxDuration(Duration.ofMinutes(5));
```

To [cancel or interrupt][execution-cancellation] running executions, see the [Timeout][timeouts] policy.

## Aborts

You can also specify which results, failures or conditions to [abort retries][abort-retries] on:

```java
builder
  .abortWhen(true)
  .abortOn(NoRouteToHostException.class)
  .abortIf(result -> result == true)
```

## Failure Handling

A [RetryPolicy] can be configured to handle only [certain results or failures][failure-handling], in combination with any of the configuration described above:

```java
builder
  .handle(ConnectException.class)
  .handleResult(null);
```

## Event Listeners

In addition to the standard [policy listeners][policy-listeners], a [RetryPolicy] can notify you when an execution attempt fails or before a retry is to be attempted:

```java
builder
  .onFailedAttempt(e -> log.error("Connection attempt failed", e.getLastFailure()))
  .onRetry(e -> log.warn("Failure #{}. Retrying.", e.getAttemptCount()));
```

It can notify you when an execution fails and the max retries are [exceeded][retries-exceeded]:

```java
builder.onRetriesExceeded(e -> log.warn("Failed to connect. Max retries exceeded."));
```

When an async retry is scheduled to be attempted after the configured delay:

```java
builder.onRetryScheduled(e -> log.info("Connection retry scheduled {}.", e.getFailure()));
```

Or when retries have been aborted:

```java
builder.onAbort(e -> log.warn("Connection aborted due to {}.", e.getFailure()));
```


{% include common-links.html %}