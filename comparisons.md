---
layout: default
title: Comparisons
---

# Comparisons
{: .no_toc }

1. TOC
{:toc}

## Failsafe vs Resilience4j

Failsafe and Resilience4j are similar libraries in what they aim to offer users. They do provide different user experiences though, and also have some different features that are worth noting:

### General

- Failsafe has zero dependencies. Resilience4j depends on Vavr.
- Failsafe provides a [uniform interface][FailsafeExecutor] for performing executions against any policy combination. Resilience4j supports varying execution decorators for different policies.
- Failsafe offers a [uniform ExecutionContext][execution-context] for any execution.

### Async Support

- Failsafe offers [async execution][async-execution] support for and any [Runnable], [Supplier], or [CompletionStage], and any policy combination. Resilience4j offers async support for some policies via decorated CompletionStages.
- Failsafe offers [async integration][async-integration] support.
- Failsafe offers [configurable executors][executorservice-configuration] and [custom schedulers][custom-schedulers].
- Failsafe offers [cancellation and interruption][execution-cancellation] support for all sync and async executions. Resilience4j supports cancellation via decorated CompletionStages.
- Failsafe provides [interruption support][interruption-support] for executions that run in a [ForkJoinPool] such as CompletableFutures. Resilience4j decorated CompletionStages cannot be interrupted.

### Policies

- Failsafe provides a uniform interface for [failure handling][FailurePolicyBuilder] and [event handling][PolicyListeners] configuration. Resilience4j provides distinct configurations for each policy.
- Resilience4j offers Bulkhead and Cache policies, which Failsafe currently does not offer.
- Failsafe circuit breakers and rate limiters can be used as standalone constructs.

### Other Features

- Failsafe offers [standalone execution][standalone-execution] support.
- Failsafe provides [strong typing][strong-typing] across all of its APIs.
- Failsafe offers an extensible [SPI][extending] for implementing custom policies.
- Resilience4j has a built-in named policy registry.
- Resilience4j offers modules for integration with some 3rd party libraries.

### References:

- [https://resilience4j.readme.io](https://resilience4j.readme.io)
- [https://javadoc.io/doc/io.github.resilience4j](https://javadoc.io/doc/io.github.resilience4j)

## Failsafe vs Hystrix

Failsafe is intended to be a lightweight, general purpose library for handling any type of execution. Hystrix is more oriented around the execution of remote requests and offers additional features for that purpose. A few differences worth noting:

### General

- Failsafe has zero dependencies. Hystrix has several external dependencies including Archais, Guava, ReactiveX, and Apache Commons Configuration.
- Failsafe offers several [policies][supported-policies] in addition to circuit breakers, which you can create and compose as needed. Hystrix is primarily an implementation of the [circuit breaker][fowler-circuit-breaker] and bulkhead patterns.
- Failsafe allows executable logic via simple lambda expressions, [Runnables][Runnable], and [Suppliers][Supplier]. Hystrix requires that executable logic be placed in a [HystrixCommand] implementation.

### Async Support

- Failsafe supports [configurable thread pools][withExecutorService] in addition to the [common pool][common-pool]. In Hystrix, asynchronous commands are executed on internally managed thread pools for particular dependencies.
- In Failsafe, asynchronous executions can be observed via [event listeners][event-listeners] and the resulting [CompletableFuture]. In Hystrix, asynchronous executions can be [observed](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#observe--) using RxJava [Observables](http://reactivex.io/RxJava/javadoc/rx/Observable.html).

### Policies

- Failsafe supports both _count based_ and _time based_ circuit breakers, with the ability to set any time window. Hystrix supports only _time based_ circuit breakers, recording execution results per second, [by default][num-buckets], and basing open/close decisions on the last second's results.
- Failsafe circuit breakers support configurable success thresholds. Hystrix only performs a single execution when in half-open state to determine whether to close a circuit.
- Failsafe circuit breakers can be shared across different executions, so that if a failure occurs, all executions against that component will be halted by the circuit breaker.
- Failsafe circuit breakers can be used and operated as [standalone][circuit-breaker-standalone] constructs.

### References

- [https://github.com/Netflix/Hystrix/wiki/How-it-Works](https://github.com/Netflix/Hystrix/wiki/How-it-Works)

[HystrixCommand]: https://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html
[num-buckets]: https://github.com/Netflix/Hystrix/wiki/Configuration#metricsrollingstatsnumbuckets

{% include common-links.html %}