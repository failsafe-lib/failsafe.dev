---
layout: default
title: Fallback
---

# Fallback
{: .no_toc }

1. TOC
{:toc}

[Fallbacks][Fallback] allow you to provide an alternative result for a failed execution. They can also be used to suppress exceptions and provide a default result:

```java
Fallback<Object> fallback = Fallback.of(defaultResult);
```

Throw a custom exception:

```java
Fallback<Object> fallback = Fallback.ofException(e -> new CustomException(e.getException()));
```

Or compute an alternative result such as from a backup resource:

```java
Fallback<Object> fallback = Fallback.of(this::connectToBackup);
```

A [CompletionStage] can be supplied as a fallback:

```java
Fallback<Object> fallback = Fallback.ofStage(this::connectToBackup);
```

For computations that block, a Fallback can be configured to run asynchronously:

```java
Fallback<Object> fallback = Fallback.builder(this::blockingCall).withAsync().build();
```

## Failure Handling

[Fallbacks][Fallback] can be configured to handle only [certain results or exceptions][failure-handling]:

```java
builder
  .handle(ConnectException.class)
  .handleResult(null);
```

When using a Fallback in combination with another policy, it's common to configure both to handle the same failures.

## Event Listeners

[Fallbacks][Fallback] support event listeners that can tell you when the last execution attempt failed:

```java
builder.onFailedAttempt(e -> log.error("Connection failed", e.getException()))
```

When the fallback attempt failed:

```java
builder.onFailure(e -> log.error("Failed to connect to backup", e.getException()));
```

Or when the execution or fallback attempt succeeded:

```java
builder.onSuccess(e -> log.info("Connection established"));
```

{% include common-links.html %}
