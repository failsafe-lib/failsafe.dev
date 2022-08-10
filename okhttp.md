---
layout: default
title: OkHttp
---

# OkHttp
{: .no_toc }

1. TOC
{:toc}

The Failsafe [OkHttp][okhttp-lib] integration allows Failsafe [policies] to be composed around OkHttp calls.

## Setup

Add the latest [failsafe][maven] and [failsafe-okhttp][maven-okhttp] dependencies to your project.

## Usage

Create a [FailsafeCall][FailsafeCall-okhttp] that composes a policy around an OkHttp [Call][okhttp-Call]:

```java
Call call = client.newCall(request);
RetryPolicy<Response> retryPolicy = RetryPolicy.ofDefaults();
FailsafeCall failsafeCall = FailsafeCall.with(retryPolicy).compose(call);

// Execute with retries
Response response = failsafeCall.execute();
```

Failure handling works just as it does with any Failsafe execution.

### Async Execution

Async execution can also be performed for a [FailsafeCall][FailsafeCall-okhttp], which returns a [CompletableFuture]:

```java
CompletableFuture<Response> future = failsafeCall.executeAsync();
```

### Policy Composition

Multiple policies can be [composed][policy-composition] around an OkHttp [Call][okhttp-Call]:

```java
FailsafeCall failsafeCall = FailsafeCall.with(fallback)
  .compose(retryPolicy)
  .compose(circuitBreaker)
  .compose(call);
```

The same statement can also be written as:

```java
FailsafeCall failsafeCall = FailsafeCall.with(fallback, retryPolicy, circuitBreaker).compose(call);
```

See the [policy-composition] docs for more details.

### Cancellation

When a [FailsafeCall][FailsafeCall-okhttp] is cancelled, the underlying OkHttp [Call][okhttp-Call] is also cancelled:

```java
failsafeCall.cancel();
```

This is also true when cancellation is performed via an async execution's future:

```java
CompletableFuture<Response> future = failsafeCall.executeAsync();
future.cancel(false);
```

### Failsafe Executor

A [FailsafeExecutor] configured with [event listeners][event-listeners] or an [ExecutorService][executorservice-configuration] can be used to create a [FailsafeCall][FailsafeCall-okhttp]:

```java
FailsafeExecutor<Response> failsafe = Failsafe.with(retryPolicy)
  .with(executorService)
  .onSuccess(e -> log.info("Found user {}", e.getResult()))
  .onFailure(e -> log.error("Failed to find user", e.getException()));
FailsafeCall failsafeCall = FailsafeCall.with(failsafe).compose(call);
```

{% include common-links.html %}