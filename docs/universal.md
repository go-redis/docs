# Universal Redis client

UniversalClient is an abstract client which - based on the provided options - represents either a
`ClusterClient`, a `FailoverClient`, or a single-node `Client`. This can be useful for testing
cluster-specific applications locally or having different clients in different environments.

`NewUniversalClient` returns a new multi client. The type of the returned client depends on the
following conditions:

1.  If the MasterName option is specified, a sentinel-backed FailoverClient is returned.
2.  if the number of Addrs is two or more, a ClusterClient is returned.
3.  Otherwise, a single-node Redis Client is returned.

For example:

```go
// rdb is *redis.Client.
rdb := NewUniversalClient(&redis.UniversalOptions{
    Addrs: []string{":6379"},
})

// rdb is *redis.ClusterClient.
rdb := NewUniversalClient(&redis.UniversalOptions{
    Addrs: []string{":6379", ":6380"},
})

// rdb is *redis.FailoverClient.
rdb := NewUniversalClient(&redis.UniversalOptions{
    Addrs: []string{":6379"},
    MasterName: "mymaster",
})
```
