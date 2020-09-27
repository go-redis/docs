---
template: main.html
---

# Redis Cluster

To connect to a [Redis Cluster](https://redis.io/topics/cluster-tutorial):

```go
import "github.com/go-redis/redis/v8"

rdb := redis.NewClusterClient(&redis.ClusterOptions{
    Addrs: []string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},

    // To route commands by latency or randomly, enable one of the following.
    //RouteByLatency: true,
    //RouteRandomly: true,
})
```

To iterate over shards:

```go
err := rdb.ForEachShard(ctx, func(ctx context.Context, shard *redis.Client) error {
    return shard.Ping(ctx).Err()
})
if err != nil {
    panic(err)
}
```

To iterate over master nodes, use `ForEachMaster`. To iterate over slave nodes, use `ForEachSlave`.

To change options for a specific shard:

```go
rdb := redis.NewClusterClient(&redis.ClusterOptions{
    NewClient: func(opt *redis.Options) *redis.NewClient {
        user, pass := userPassForAddr(opt.Addr)
        opt.Username = user
        opt.Password = pass

        return redis.NewClient(opt)
    },
})
```
