---
layout: default
title: Standalone Execution
---

# Standalone Execution

While the [Failsafe] class automatically performs executions, including retries, according to the configured policies, the [Execution] class can be used for situations where you need to control executions and retries yourself. 

An [Execution] can be used to record results and determine whether the execution is considered complete according to the configured policies, else if you should retry:

```java
Execution execution = new Execution(retryPolicy);
while (!execution.isComplete()) {
  try {
    doSomething();
    execution.complete();
  } catch (ConnectException e) {
    execution.recordFailure(e);
  }
}
```

An [Execution] is also useful for integrating with APIs that have their own retry mechanism:

```java
Execution execution = new Execution(retryPolicy);

// On failure
if (execution.canRetryOn(someFailure))
  service.scheduleRetry(execution.getWaitTime().toNanos(), TimeUnit.MILLISECONDS);
```

See the [RxJava example][RxJava] for a more detailed implementation.

{% include common-links.html %}