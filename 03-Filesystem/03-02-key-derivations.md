# 3. Filesystem
## 3.2 Key derivations
### 3.2.1 Overview
Every EasySafe filesystem is defined by a read passphrase and a write passphrase. By default, these passphrases are identical, but this may be overridden by the user.

```
ReadPassphrase // given by user
WritePassphrase // given by user; defaults to ReadPassphrase

readRawKey = PBKDF(ReadPassphrase)
RootKey = deriveSubkey(parentKey=readRawKey, name="easysafe-read-key", salt=zeros(0))
SeedKey = deriveSubkey(parentKey=readRawKey, name="easysafe-seed-key", salt=zeros(0))

writeRawKey = PBKDF(WritePassphrase)
WriteKey = deriveSubkey(parentKey=writeRawKey, name="easysafe-write-key", salt=zeros(0))
WritePublicKey = AsymmetricDerivePublic(WritePrivateKey)
```

### 3.2.2 Key responsibilities
Possession of these keys acts as authorization for various tasks.

| Key | Possessor can... |
|-----|-----|
| WritePublicKey | ...verify data belongs to filesystem |
| SeedKey | ...participate in p2p network for filesystem
| RootKey | ...read filesystem contents |
| WritePrivateKey | ...add to filesystem |

It is presumed that a user given the `WritePrivateKey` also has knowledge of the `RootKey`. The converse is not true. A user given the `RootKey` is not presumed to have knowledge of the `WritePrivateKey`, unless the read and write passphrases are identical.

### 3.2.3 FS Key
Every EasySafe filesystem has an additional 256-bit key referred to as the `FSKey`. This key is stored in the configuration file, encrypted using the `RootKey`. By default, the `FSKey` is equal to the `RootKey`. In non-default filesystems (explained below), the `FSKey` is randomly generated.

### 3.2.4 Seed-only peers
A peer possessing knowledge of the `SeedKey`, but not the `RootKey`, `FSKey` or `WritePrivateKey`, is referred to as a "seed-only peer." These peers are able to participate in the peer-to-peer network to send and receive encrypted data, and possess sufficient information to authenticate that date. However, these peers are unable to read the contents of the encrypted filesystem.
