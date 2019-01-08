# 4. Distributed hash table
## 4.2 Temporal lookup keys
The `DHTLookup` function requires a temporal lookup key as an argument. This temporal lookup key is a time-dependent derivative of the `SeedKey`.

```
// EpochSeconds = # seconds elapsed since 1970-01-01 00:00:00 UTC
timeslice = floor(EpochSeconds / (60*60*3))
temporalSeedId = deriveSubkey(rootKey=SeedKey, name="temporal-seed-key", salt=timeslice(64))
```

The temporal lookup keys are designed to rotate every 3 hours. This is done so that Sybil attacks against an ID will require an ongoing investment of effort, and to limit the duration for which DHT peers can observe the number of lookups for a given ID.
