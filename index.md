---
layout: default
title: Fault tolerance and resilience patterns for the JVM
---

{: .callout }
For Golang projects, [Failsafe-go] has been released! See the [project website][failsafe-go] for more info.

# Overview

Failsafe is a lightweight, zero-dependency library for handling failures in Java 8+. It has a concise API for handling everyday use cases and the flexibility to handle everything else. Failsafe works by wrapping executable logic with one or more resilience [policies], which can be combined and [composed][policy-composition] as needed.

## Setup

Add the latest [Failsafe dependency][maven] to your project.

## Getting Started

To see how Failsafe works, we'll create a [retry policy][retry] that defines which failures to handle and when retries should be performed:

```java
RetryPolicy<Object> retryPolicy = RetryPolicy.builder()
  .handle(ConnectException.class)
  .withDelay(Duration.ofSeconds(1))
  .withMaxRetries(3)
  .build();
```

We can then execute a `Runnable` or `Supplier` *with* retries:

```java
// Run with retries
Failsafe.with(retryPolicy).run(() -> connect());

// Get with retries
Connection connection = Failsafe.with(retryPolicy).get(() -> connect());
```

### Asynchronous Execution

Executing a `Runnable` or `Supplier` [asynchronously][async-execution] *with* a policy is simple:

```java
// Run with retries asynchronously
CompletableFuture<Void> future = Failsafe.with(retryPolicy).runAsync(() -> connect());

// Get with retries asynchronously
CompletableFuture<Connection> future = Failsafe.with(retryPolicy).getAsync(() -> connect());
```

### Composing Policies

Multiple [policies] can be arbitrarily composed to add additional layers of resilience or to handle different failures in different ways:

```java
var fallback = Fallback.of(this::connectToBackup);
var circuitBreaker = CircuitBreaker.ofDefaults();
var timeout = Timeout.of(Duration.ofSeconds(10));

// Get with fallback, retries, circuit breaker, and timeout
Failsafe.with(fallback, retryPolicy, circuitBreaker, timeout).get(this::connect);
```

Order does matter when composing policies. See the [policy composition][policy-composition] overview for more details.

### Failsafe Executor

Policy compositions can also be saved for later use via a [FailsafeExecutor]:

```java
FailsafeExecutor<Object> executor = Failsafe.with(retryPolicy).compose(circuitBreaker);
executor.run(this::connect);
```

## Further Reading

Read more about [policies] and how they're used, then explore some of Failsafe's other features in the site menu.

{% include common-links.html %}

[failsafe-go]: https://failsafe-go.dev