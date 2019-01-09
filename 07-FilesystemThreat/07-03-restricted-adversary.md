# 7. Filesystem threat model
## 7.2 Restricted adversary model
### 7.2.1
The restricted adversary model refers to an adversary whose capabilities are more limited compared to the basic model.

These capabilities are intended to be a superset of the cabilities of a real-world attacker who has:

* Complete control over the networking of one or more peers
* Possession of the FSID, `SeedKey`, and all encrypted data
* The ability to conduct chosen-plaintext attacks, without control over the RNG output of the peer encrypting the data

### 7.2.2 Restricted adversary capabilities

The restricted adversary possesses...

1. ...the ability to view, alter, extend, truncate, create and/or drop any packet sent to or from any EasySafe peer.
1. ...a copy of all the encrypted pages and page tree chunks that compose the filesystem.
1. ...knowledge of the filesystem's `SeedKey` and FSID
1. ...knowledge of the version and full source code of any EasySafe application running on any peer
4. ...access to the `InodeTableOracle` function, by which the adversary can at no cost identify inode table pages to attack.
5. ...access to the `PageEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded pages with arbitrary plaintext.
6. ...access to the `PageTreeChunkEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded page tree chunks with arbitrary plaintext.
5. ...the ability to perform `PBKDF` operations with no cost.

### 7.2.3 Oracle definitions
The following oracles are provided to the restricted adversary:

```
// allow adversary to craft an arbitrary encrypted page; distinguisher is randomly selected and not provided to adversary
PageEncryptionOracle(pageNum, pageText)

// allow adversary to craft an arbitrary encrypted page tree chunk; distinguisher is randomly selected and not provided to adversary
PageTreeChunkEncryptionOracle(chunkNum, chunkText)
```

Additionally, the restricted adversary also has access to the `InodeTableOracle` of 7.1.3.

### 7.2.4 Restricted adversary limitations

The restricted adversary has the same limitations as the basic adversary.
