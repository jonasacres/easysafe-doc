# 4. Distributed Hash Table

## 4.1 DHT protocol

### 4.1.1 Purpose of the DHT
EasySafe allows filesystem peers to discover one another by means of a distributed hash table (DHT). This DHT allows peers with knowledge of seed key to discover "advertisements" listing FSIDs and connection information (such as IP address and port number) for peers sharing filesystems matching that seed key.

The particulars of the DHT protocol are beyond the scope of this document. Discussion in the present document is limited to the cryptographic aspects of this protocol.

## 4.1.2 DHT functions
The DHT can be understood abstractly as a service providing the following functions:

| Function | Result | Description
|----------|--------|-------------
| DHTLookup(temporalLookupKey) | Zero or more Advertisements | Locate Advertisements for connectable peers sharing a given seed ID
| DHTSet(peer, temporalSeedId, ad) | None | Request a remote peer to store an advertisement for a given seed ID
