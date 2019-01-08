# 4. Distributed hash table
## 4.4 Lookup tokens
When performing the `DHTLookup` operation, each individual request contains a receiver-specific lookup token. This token is constructed as follows:

```
NetworkID // fixed constant, specific to the DHT
RemotePubKey // X25519 static public key of the receiving peer
TemporalSeedId // Temporal seed ID used as argument to DHTLookup

lookupKey = deriveSubkey(rootKey=SeedRoot, name="dht-lookup", salt=NetworkID)
LookupToken = HMAC(key=lookupKey, TemporalSeedId || RemotePubKey)
```

Because the `LookupToken` depends on the remote peer's public key, the token is specific to each receiving peer. When performing a `DHTSet` operation, requestors include the `LookupToken` for the remote peer with the advertisement to be stored. The `LookupToken` is also included in `DHTLookup` requests, and the remote peer will only respond with advertisements stored with the same token. This prevents a peer from performing `DHTLookup` operations on temporal lookup keys harvested from `DHTLookup` operations that the peer has received.