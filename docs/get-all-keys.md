# Get all keys

Redis allows iterating over all keys using the [scan](https://redis.io/commands/scan) command:

```go
var cursor uint64
for {
    var keys []string
    var err error
    keys, cursor, err = rdb.Scan(ctx, cursor, "key*", 0).Result()
    if err != nil {
        panic(err)
    }

    for _, key := range keys {
        fmt.Println("key", key)
    }

    if cursor == 0 { // no more keys
        break
    }
}
```

go-redis allows to simplify the code above to:

```go
iter := rdb.Scan(ctx, 0, "key*", 0).Iterator()
for iter.Next(ctx) {
    fmt.Println(iter.Val())
}
if err := iter.Err(); err != nil {
    panic(err)
}
```

With [Redis Cluster](cluster.md) and [Redis Ring](ring.md) you need to scan each cluster node
separately:

```go
err := rdb.ForEachMaster(ctx, func(ctx context.Context, rd *redis.Client) error {
    iter := rdb.Scan(ctx, 0, "key*", 0).Iterator()

    ...

    return iter.Err()
})
if err != nil {
    panic(err)
}
```
