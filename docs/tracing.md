---
template: main.html
---

# Tracing

go-redis supports tracing out-of-the-box using [OpenTelemetry](https://opentelemetry.io/) API. To
enable commands instrumentation, use the following code:

```go
import (
    "github.com/go-redis/redis/v8"
    "github.com/go-redis/redis/extra/redisotel"
)

rdb := rdb.NewClient(&rdb.Options{...})

rdb.AddHook(redisotel.TracingHook{})
```

For Redis Cluster and Ring use:

```go
rdb := redis.NewClusterClient(&redis.ClusterOptions{
    // ...

    NewClient: func(opt *redis.Options) *redis.Client {
        node := redis.NewClient(opt)
        node.AddHook(redisotel.TracingHook{})
        return node
    },
})

rdb.AddHook(redisotel.TracingHook{})
```

This is how span looks at Uptrace.dev which is an OpenTelemetry backend that supports
[distributed traces, logs, and errors](https://uptrace.dev/1/groups?system=db%3Aredis).

![Redis trace and spans](img/redis-span.png)
