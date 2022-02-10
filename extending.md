---
layout: default
title: Extending Failsafe
---

# Extending Failsafe

Failsafe provides a Service Provider Interface (SPI) that allows you to implement a [custom Scheduler][custom-schedulers] or [Policy] which you can plug into Failsafe.

## Custom Policies

Failsafe [Policy] implementations contain the necessary configuration to handle executions in a certain way. The actual execution handling is done by a corresponding [PolicyExecutor] implementation, which each Policy provides. The PolicyExecutor is responsible for performing any pre-execution behavior and post-execution handling of a result or exception. 

The [PolicyExecutor class][policy-executor-impl] along with the existing PolicyExecutor [implementations][policy-executor-impls] are a good reference for creating additional implementations.

{% include common-links.html %}