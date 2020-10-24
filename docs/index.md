---
template: main.html
---

# Getting started

## Installation

go-redis supports 2 last Go versions and requires support for
[Go modules](https://github.com/golang/go/wiki/Modules). So make sure to initialize a Go module:

```shell
go mod init github.com/my/repo
```

And then install redis/v8 (note _v8_ in the import; omitting it is a popular mistake):

```shell
go get github.com/go-redis/redis/v8
```

## Connecting to Redis Server

To connect to a Redis database:

```go
import "github.com/go-redis/redis/v8"

rdb := redis.NewClient(&redis.Options{
    Addr:     "localhost:6379",
    Password: "", // no password set
    DB:       0,  // use default DB
})
```

Another popular way is using a connection string:

```go
opt, err := redis.ParseURL("redis://localhost:6379/<db>")
if err != nil {
    panic(err)
}

rdb := redis.NewClient(opt)
```

## Executing commands

To execute a command:

```go
val, err := rdb.Get(ctx, "key").Result()
if err != nil {
    if err == redis.Nil {
        fmt.Println("key does not exists")
        return
    }
    panic(err)
}
fmt.Println(val)
```

Alternatively you can access a value and an error separately:

```go
get := rdb.Get(ctx, "key")
if err := get.Err(); err != nil {
    if err == redis.Nil {
        fmt.Println("key does not exists")
        return
    }
    panic(err)
}
fmt.Println(get.Val())
```

When appropriate commands provide helper methods:

```go
// Shortcut for get.Val().(string) with error handling.
s, err := get.Text()

num, err := get.Int()

num, err := get.Int64()

num, err := get.Uint64()

num, err := get.Float32()

num, err := get.Float64()

flag, err := get.Bool()
```

## Executing unsupported commands

To execute arbitrary/custom command:

```go
val, err := rdb.Do(ctx, "get", "key").Result()
if err != nil {
    if err == redis.Nil {
        fmt.Println("key does not exists")
        return
    }
    panic(err)
}
fmt.Println(val.(string))
```

## Pipelining

To execute several commands in a single write/read pipeline:

```go
var incr *redis.IntCmd
_, err := rdb.Pipelined(ctx, func(pipe redis.Pipeliner) error {
    incr = pipe.Incr(ctx, "pipelined_counter")
    pipe.Expire(ctx, "pipelined_counter", time.Hour)
    return nil
})
if err != nil {
    panic(err)
}

// The value is available only after pipeline is executed.
fmt.Println(incr.Val())
```

Alternatively you can create and execute pipeline manually:

```go
pipe := rdb.Pipeline()

incr := pipe.Incr(ctx, "pipeline_counter")
pipe.Expire(ctx, "pipeline_counter", time.Hour)

_, err := pipe.Exec(ctx)
if err != nil {
    panic(err)
}

// The value is available only after Exec.
fmt.Println(incr.Val())
```

To wrap commands with `multi` and `exec` commands, use `TxPipelined` / `TxPipeline`.

## Transactions and Watch

To watch for changes in keys and commit a transaction only if keys remain unchanged:

```go
const maxRetries = 1000

// Increment transactionally increments key using GET and SET commands.
increment := func(key string) error {
    // Transactional function.
    txf := func(tx *redis.Tx) error {
        // Get current value or zero.
        n, err := tx.Get(ctx, key).Int()
        if err != nil && err != redis.Nil {
            return err
        }

        // Actual operation (local in optimistic lock).
        n++

        // Operation is commited only if the watched keys remain unchanged.
        _, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
            pipe.Set(ctx, key, n, 0)
            return nil
        })
        return err
    }

    for i := 0; i < maxRetries; i++ {
        err := rdb.Watch(ctx, txf, key)
        if err == nil {
            // Success.
            return nil
        }
        if err == redis.TxFailedErr {
            // Optimistic lock lost. Retry.
            continue
        }
        // Return any other error.
        return err
    }

    return errors.New("increment reached maximum number of retries")
}
```

## PubSub

go-redis allows to publish messages and subscribe to channels. It also automatically handles
reconnects.

To publish a message:

```go
err := rdb.Publish(ctx, "mychannel1", "payload").Err()
if err != nil {
    panic(err)
}
```

To subscribe to a channel:

```go
// There is no error because go-redis automatically reconnects on error.
pubsub := rdb.Subscribe(ctx, "mychannel1")
```

To receive a message:

```go
for {
    msg, err := pubsub.ReceiveMessage(ctx)
    if err != nil {
        panic(err)
    }

    fmt.Println(msg.Channel, msg.Payload)
}
```

But the simplest way is using a Go channel:

```go
ch := pubsub.Channel()

for msg := range ch {
    fmt.Println(msg.Channel, msg.Payload)
}
```
