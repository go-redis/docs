---
template: main.html
---

# Tracing

go-redis supports tracing out-of-the-box using [OpenTelemetry](https://opentelemetry.io/) API. To
enable commands instrumentation, use the following code:

```go
import (
    "github.com/go-redis/redis/v8"
    "github.com/go-redis/redisext"
)

rdb := rdb.NewClient(&rdb.Options{...})

rdb.AddHook(redisext.OpenTelemetryHook{})
```

For Redis Cluster or Ring use:

```go
rdb.ForEachShard(func(shard *redis.Client) error {
    shard.AddHook(redisext.OpenTelemetryHook{})
    return nil
})
```

This is how span looks at Uptrace.dev which is an OpenTelemetry backend that supports
[distributed traces, logs, and errors](https://uptrace.dev/1/groups?system=db%3Aredis).

![Redis trace and spans](img/redis-span.png)
