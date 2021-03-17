# Caching using Redis

[go-redis/cache](https://github.com/go-redis/cache) library implements a cache using Redis as a
key/value storage. It uses [MessagePack](https://github.com/vmihailenco/msgpack) to marshal values.

You can install go-redis/cache with:

```shell
go get github.com/go-redis/cache/v8
```

go-redis/cache accepts an interface to communicate with Redis and thus supports all types of Redis
clients that go-redis provides.

```go
rdb := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
})

mycache := cache.New(&cache.Options{
    Redis: rdb,
})

obj := new(Object)
err := mycache.Once(&cache.Item{
    Key:   "mykey",
    Value: obj, // destination
    Do: func(*cache.Item) (interface{}, error) {
        return &Object{
            Str: "mystring",
            Num: 42,
        }, nil
    },
})
if err != nil {
    panic(err)
}
```

You can also use local in-process storage to cache the small subset of popular keys. go-redis/cache
comes with [TinyLFU](https://github.com/dgryski/go-tinylfu), but you can use any other
[cache algorithm](https://github.com/vmihailenco/go-cache-benchmark) that implements the interface.

```go
mycache := cache.New(&cache.Options{
    Redis:      rdb,
    // Cache 10k keys for 1 minute.
    LocalCache: cache.NewTinyLFU(10000, time.Minute),
})
```
