---
layout: default
title: Standalone Execution
---

# Standalone Execution

While the [Failsafe] class automatically performs executions, including retries, according to the configured policies, the [Execution] class can be used for situations where you need to control executions and retries yourself. 

An [Execution] can be used to record results and determine whether the execution is considered complete according to the configured policies, else if you should retry:

```java
Execution<Connection> execution = Execution.of(retryPolicy);
while (!execution.isComplete()) {
  try {
    execution.recordResult(connect());
  } catch (ConnectException e) {
    execution.recordException(e);
  }
}
```

An [Execution] is also useful for integrating with APIs that have their own retry mechanism:

```java
Execution<Connection> execution = Execution.of(retryPolicy);

// On exception
execution.recordException(connectException);
if (!execution.isComplete())
  service.scheduleRetry(execution.getDelay());
```

See the [RxJava example][rxjava-example] for a more detailed implementation.

{% include common-links.html %}