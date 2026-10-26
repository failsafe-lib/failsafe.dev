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

### Expiring Access Tokens

An OkHttp `Call` retains its original `Request`. Failsafe executes that call on the first attempt and clones it for retries, so an `Authorization` header set when building the request also reuses the original token. To obtain a valid token after any policy delay, set the header in an OkHttp application interceptor:

```java
static OkHttpClient authenticatedClient(Supplier<String> accessTokenSupplier) {
  return new OkHttpClient.Builder()
    .addInterceptor(chain -> {
      Request request = chain.request().newBuilder()
        .header("Authorization", "Bearer " + accessTokenSupplier.get())
        .build();
      return chain.proceed(request);
    })
    .build();
}
```

Here, `Supplier` is `java.util.function.Supplier`. Supply an application-owned callback that returns a currently valid token, refreshing it through your authentication provider when needed. The interceptor runs for each Failsafe attempt that reaches OkHttp, including asynchronous attempts:

```java
OkHttpClient client = authenticatedClient(accessTokenSupplier);
Call call = client.newCall(request);
FailsafeCall failsafeCall = FailsafeCall.with(retryPolicy, circuitBreaker).compose(call);
```

Keep the same Failsafe execution across retries so its retry budget and other policies remain in effect. Refreshing the token does not require rebuilding the `FailsafeCall` or its executor. If refresh depends on a server's `401` challenge, use an OkHttp `Authenticator` to handle that challenge; retrying a response alone does not refresh credentials.

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
