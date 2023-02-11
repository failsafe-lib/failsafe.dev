---
layout: default
title: Strong Typing
---

# Strong typing

Failsafe APIs are typed based on the expected execution result. For generic policies that are used for various executions, the result type may just be `Object`:

```java
RetryPolicy<Object> retryPolicy = RetryPolicy.ofDefaults();
```

But for other policies you might declare a more specific result type:

```java
RetryPolicy<HttpResponse> retryPolicy = RetryPolicy.<HttpResponse>builder()
  .handleResultIf(response -> response.getStatusCode() == 500)
  .onFailedAttempt(e -> log.warn("Failed attempt: {}", e.getLastResult().getStatusCode()))
  .build();
```

This allows Failsafe to ensure that the same result type used for the policy is returned by the execution and available in [event listeners][event-listeners]:

```java
HttpResponse response = Failsafe.with(retryPolicy)
  .onSuccess(e -> log.info("Success: {}", e.getResult().getStatusCode()))  
  .get(this::sendHttpRequest);
```

It also ensures that when multiple policies are composed, they all share the same result type:

```java
CircuitBreaker<HttpResponse> circuitBreaker = CircuitBreaker.ofDefaults();
Failsafe.with(retryPolicy, circuitBreaker).get(this::connect);
```

{% include common-links.html %}