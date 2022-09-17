---
layout: default
title: Policies
---

# Policies
{: .no_toc }

1. TOC
{:toc}

## Failure Handling

Failsafe policies add resilience by detecting failures and handling them. Each policy determines which execution [results or exceptions][FailurePolicyBuilder] to consider as failures and how to handle them. By default, policies handle any `Exception` that is thrown. But policies can also be configured to handle more specific exceptions or conditions:

```java
policyBuilder
  .handle(ConnectException.class, SocketException.class)
  .handleIf(e -> e instanceof ConnectException);
```

They can also be configured to handle specific results or result conditions:

```java
policyBuilder
  .handleResult(null)
  .handleResultIf(result -> result == null);  
```

If multiple handle methods are configured, they are logically OR'ed. The default `Exception` handling condition is only replaced by another condition that handles exceptions. A condition that only handles results will not replace the default `Exception` handler.

## Policy Composition

Policies can be composed in any way desired, including multiple policies of the same type. Policies handle execution results in reverse order, similar to the way that function composition works. For example, consider:

```java
Failsafe.with(fallback)
  .compose(retryPolicy)
  .compose(circuitBreaker)
  .compose(timeout)
  .get(supplier);
```

The same statement can also be written as:

```java
Failsafe.with(fallback, retryPolicy, circuitBreaker, timeout).get(supplier);
```

These result in the following composition when executing the `supplier` and handling its result:

```
Fallback(RetryPolicy(CircuitBreaker(Timeout(Supplier))))
```

### Executing a Policy Composition

The process for executing a policy composition begins with Failsafe calling the outer-most policy. That policy in turn calls the next inner policy, and so on, until the user-provided `Runnable` or `Supplier` is reached. A result or exception is returned back through the policy layers, and handled if needed by any policy along the way.

Each policy makes its own decision to allow an execution attempt to proceed and how to handle an execution result or exception. For example, a `RetryPolicy` may retry an execution, which calls the next inner policy again, or it may return the result or exception. A `CircuitBreaker` may throw an exception before an execution attempt even makes it to the `Supplier`.

### Example Execution

Consider the following policy composition execution:

<img src="/assets/images/composition.png">

- Failsafe calls the `Fallback`
- `Fallback` calls the `RetryPolicy`
- `RetryPolicy` calls the `CircuitBreaker`
- `CircuitBreaker` rejects the execution if the breaker is open, else calls the `Supplier`
- `Supplier` executes and returns a result or throws an exception
- `CircuitBreaker` records the result as either a success or failure, based on its [configuration](#failure-handling), possibly changing the state of the breaker, then returns or throws
- `RetryPolicy` records the result as either a success or failure, based on its [configuration](#failure-handling), and either retries or returns the result or exception
- `Fallback` handles the result or exception according to its configuration and returns a fallback result or exception if needed
- Failsafe returns the final result or exception to the caller

### Composition and Exception Handling

While policies handle all `Exception` instances by default, it's common to configure a policy to handle more specific exceptions, as [described above](#failure-handling):

```java
retryPolicyBuilder.handle(ConnectException.class);
```

But when doing so for a policy that is composed around other policies, you may want to also configure an outer policy to handle exceptions thrown by any inner policies, depending on your use case:

```java
retryPolicyBuilder.handle(CircuitBreakerOpenException.class, TimeoutExceededException.class);
```

### Composition Recommendations

A typical policy composition might place a `Fallback` as the outer-most policy, followed by a `RetryPolicy`, a `CircuitBreaker` or `RateLimiter`, a `Bulkhead`, and a `Timeout` as the inner-most policy:

```java
Failsafe.with(fallback, retryPolicy, circuitBreaker, bulkhead, timeout)
```

That said, it really depends on how the policies are being used, and different compositions make sense for different use cases.

## Supported Policies

Read about the built-in policies that Failsafe supports:

- [Retry][retry]
- [Circuit Breaker][circuit-breakers]
- [Rate Limiter][rate-limiters]
- [Timeout][timeouts]
- [Bulkhead][bulkheads]
- [Fallback][fallbacks]

{% include common-links.html %}