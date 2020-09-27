---
template: main.html
---

# Redis Ring

Ring is a Redis client that uses consistent hashing to distribute keys across multiple Redis servers
(shards). It's safe for concurrent use by multiple goroutines.

Ring monitors the state of each shard and removes dead shards from the ring. When a shard comes
online it is added back to the ring. This gives you maximum availability and partition tolerance,
but no consistency between different shards or even clients. Each client uses shards that are
available to the client and does not do any coordination when shard state is changed.

Ring should be used when you need multiple Redis servers for caching and can tolerate losing data
when one of the servers dies. Otherwise you should use Redis Cluster.

To create a Ring cluster that consists of 3 shards:

```go
import "github.com/go-redis/redis/v8"

rdb := redis.NewRing(&redis.RingOptions{
    Addrs: map[string]string{
        "shard1": ":7000",
        "shard2": ":7001",
        "shard3": ":7002",
    },
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

To change options for a specific shard:

```go
rdb := redis.NewRing(&redis.RingOptions{
    NewClient: func(opt *redis.Options) *redis.NewClient {
        user, pass := userPassForAddr(opt.Addr)
        opt.Username = user
        opt.Password = pass

        return redis.NewClient(opt)
    },
})
```

Ring uses
[Rendezvous](https://medium.com/@dgryski/consistent-hashing-algorithmic-tradeoffs-ef6b8e2fcae8) to
distribute keys over shards. To change default consistent hash implementation:

```go
import "github.com/golang/groupcache/consistenthash"

ring := redis.NewRing(&redis.RingOptions{
    NewConsistentHash: func() {
        return consistenthash.New(100, crc32.ChecksumIEEE)
    },
})
```
