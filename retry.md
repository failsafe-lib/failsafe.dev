---
layout: default
title: Retry
---

# Retry Policy
{: .no_toc }

1. TOC
{:toc}

[Retry policies][RetryPolicy] will retry failed executions a certain number of times, with an optional delay between attempts. If an execution fails after retries have been exceeded, the last result or exception is returned or thrown. If additional handling or an alternative result is needed, additional policies, such as a [fallbacks], can be [composed][policy-composition] around a [RetryPolicy].

Creating a [RetryPolicy] is straightforward, for example:

```java
// Retry on a ConnectException up to 3 times with a 1 second delay between attempts
RetryPolicy<Object> retryPolicy = RetryPolicy.builder()
  .handle(ConnectException.class)
  .withDelay(Duration.ofSeconds(1))
  .withMaxRetries(3)
  .build();
```

## Max Attempts

By default, a [RetryPolicy] will allow a maximum of 3 execution attempts. You can configure a different max number of [attempts][max-attempts]:

```java
builder.withMaxAttempts(3);
```

Or a max number of [retries][max-retries]:

```java
builder.withMaxRetries(2);
```

You can also disable the default max attempt limit:

```java
builder.withMaxAttempts(-1);
```

## Max Duration

In addition to max attempts, you can also add a [max duration][max-duration] for an execution, after which retries will stop if the max attempts haven't already been reached.

```java
builder.withMaxDuration(Duration.ofMinutes(5));
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

Or a [computed delay][computed-delay] based on an execution result or exception.

### Jitter

You can also combine a random [jitter factor][jitter-factor] with a delay:

```java
builder.withJitter(.1);
```

Or a [time based jitter][jitter-duration]:

```java
builder.withJitter(Duration.ofMillis(100));
```

To [cancel or interrupt][execution-cancellation] running executions, see the [Timeout][timeouts] policy.

## Aborts

You can also specify which results, exceptions, or conditions to [abort retries][abort-retries] on:

```java
builder
  .abortWhen(true)
  .abortOn(NoRouteToHostException.class)
  .abortIf(result -> result == true)
```

## Failure Handling

A [RetryPolicy] can be configured to handle only [certain results or exceptions][failure-handling], in combination with any of the configuration described above:

```java
builder
  .handle(ConnectException.class)
  .handleResult(null);
```

## Event Listeners

In addition to the standard [policy listeners][policy-listeners], a [RetryPolicy] can notify you when an execution attempt fails or before a retry is to be attempted:

```java
builder
  .onFailedAttempt(e -> log.error("Connection attempt failed", e.getLastException()))
  .onRetry(e -> log.warn("Failure #{}. Retrying.", e.getAttemptCount()));
```

It can notify you when an execution fails and the max retries are [exceeded][retries-exceeded]:

```java
builder.onRetriesExceeded(e -> log.warn("Failed to connect. Max retries exceeded."));
```

When an async retry is scheduled to be attempted after the configured delay:

```java
builder.onRetryScheduled(e -> log.info("Connection retry scheduled {}.", e.getException()));
```

Or when retries have been aborted:

```java
builder.onAbort(e -> log.warn("Connection aborted due to {}.", e.getException()));
```


{% include common-links.html %}