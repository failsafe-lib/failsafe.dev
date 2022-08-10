---
layout: default
title: Retrofit
---

# Retrofit
{: .no_toc }

1. TOC
{:toc}

The Failsafe [Retrofit][retrofit-lib] integration allows Failsafe [policies] to be composed around Retrofit calls.

## Setup

Add the latest [failsafe][maven] and [failsafe-retrofit][maven-retrofit] dependencies to your project.

## Usage

Create a [FailsafeCall][FailsafeCall-retrofit] that composes a policy around a Retrofit [Call][retrofit-Call]:

```java
Call call = client.newCall(request);
RetryPolicy<Response<User>> retryPolicy = RetryPolicy.ofDefaults();
FailsafeCall<User> failsafeCall = FailsafeCall.with(retryPolicy).compose(call);

// Execute with retries
Response<User> response = failsafeCall.execute();
```

Failure handling works just as it does with any Failsafe execution.

### Async Execution

Async execution can also be performed for a [FailsafeCall][FailsafeCall-retrofit], which returns a [CompletableFuture]:

```java
CompletableFuture<Response<User>> future = failsafeCall.executeAsync();
```

### Policy Composition

Multiple policies can be [composed][policy-composition] around a Retrofit [Call][retrofit-Call]:

```java
FailsafeCall<User> failsafeCall = FailsafeCall.with(fallback)
  .compose(retryPolicy)
  .compose(circuitBreaker)
  .compose(call);
```

The same statement can also be written as:

```java
FailsafeCall<User> failsafeCall = FailsafeCall.with(fallback, retryPolicy, circuitBreaker)
  .compose(call);
```

See the [policy-composition] docs for more details.

### Cancellation

When a [FailsafeCall][FailsafeCall-retrofit] is cancelled, the underlying Retrofit [Call][retrofit-Call] is also cancelled:

```java
failsafeCall.cancel();
```

This is also true when cancellation is performed via an async execution's future:

```java
CompletableFuture<Response<User>> future = failsafeCall.executeAsync();
future.cancel(false);
```

### Failsafe Executor

A [FailsafeExecutor] configured with [event listeners][event-listeners] or an [ExecutorService][executorservice-configuration] can be used to create a [FailsafeCall][FailsafeCall-okhttp]:

```java
FailsafeExecutor<Response<User>> failsafe = Failsafe.with(retryPolicy)
  .with(executorService)
  .onSuccess(e -> log.info("Found user {}", e.getResult()))
  .onFailure(e -> log.error("Failed to find user", e.getException()));
FailsafeCall<User> failsafeCall = FailsafeCall.with(failsafe).compose(call);
```

{% include common-links.html %}