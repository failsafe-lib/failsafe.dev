---
layout: default
title: Policies
---

# Policies
{: .no_toc }

1. TOC
{:toc}

## Failure Handling

Failsafe policies determine which execution [results or failures][FailurePolicy] to handle and how to handle them. By default, policies handle any `Exception` that is thrown. But policies can also be configured to handle more specific failures or conditions:

```java
policyBuilder
  .handle(ConnectException.class, SocketException.class)
  .handleIf(failure -> failure instanceof ConnectException);
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

The process for executing a policy composition begins with Failsafe calling the outer-most policy. That policy in turn calls the next inner policy, and so on, until the user-provided `Runnable` or `Supplier` is reached. A result or failure is returned back through the policy layers, and handled if needed by any policy along the way.

Each policy makes its own decision to allow an execution attempt to proceed and how to handle an execution result or failure. For example, a `RetryPolicy` may retry an execution, which calls the next inner policy again, or it may return the result or failure. A `CircuitBreaker` may throw an exception before an execution attempt even makes it to the `Supplier`.

### Example Execution

Consider the following policy composition execution:

<img src="/assets/images/composition.png">

- Failsafe calls the `Fallback`
- `Fallback` calls the `RetryPolicy`
- `RetryPolicy` calls the `CircuitBreaker`
- `CircuitBreaker` rejects the execution if the breaker is open, else calls the `Supplier`
- `Supplier` executes and returns a result or throws a failure
- `CircuitBreaker` records the result as either a success or failure, based on its configuration, possibly changing the state of the breaker, then returns the result or failure
- `RetryPolicy` records the result as either a success or failure, based on its configuration, and either retries or returns the result or failure
- `Fallback` handles the result or failure according to its configuration and returns a fallback result if needed
- Failsafe returns the final result or failure to the caller

### Composition Recommendations

A typical policy composition might place a `Fallback` as the outer-most policy, followed by a `RetryPolicy`, `CircuitBreaker`, and a `Timeout` as the inner-most policy:

```java
Failsafe.with(fallback, retryPolicy, circuitBreaker, timeout)
```

That said, it really depends on how the policies are being used, and different compositions make sense for different use cases. 

## Supported Policies

Read about the built-in policies that Failsafe supports:

- [Retry][retry]
- [Circuit Breaker][circuit-breakers]
- [Timeout][timeouts]
- [Fallback][fallbacks]

{% include common-links.html %}