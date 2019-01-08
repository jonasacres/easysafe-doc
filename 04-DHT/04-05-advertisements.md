# 4. Distributed Hash Table
## 4.5 Advertisements
### 4.5.1 Advertisement content
Advertisements are structured as follows:

| Advertisement |
|-|
| Public mutable section (unencrypted) |
| Public immutable section (unencrypted) |
| Private section (encrypted) |

### 4.5.2 Public mutable section
The public mutable section contains plaintext information that may be altered by the peer storing the advertisement. It contains:

| Public mutable section |
|-|
| IP address (string)

This section is mutable so that the storing peer can set the IP address to the observed IP address of the sender, as a means to overcome scenarios in network address translation where the sender may not know their own public IP address. The storing peer must also be able to ensure that the advertisement lists the sender's actual IP address, as a means of discouraging denial of service attacks.

### 4.5.3 Public immutable section
The public immutable section contains plaintext information that may not be altered by the peer storing the advertisement.

| Public immutable section |
|-|
| AdSalt (32-bit) |

##### Salt
The `AdSalt` is a random 32-bit value. It must be unique to the advertisement for a given `LookupToken`. Storing peers ignore `DHTSet` requests for any advertisement with an `AdSalt` matching one that they already possess for a given `LookupToken`.

### 4.5.4 Private section

The private section is encrypted using a key derived from the `LookupToken`, `AdSalt` and `SeedKey`.

| Private section plaintext |
|---|
| Port (16-bit)
| FSID
| Public static key

Advertisements also contain an encrypted FSID field, calculated as follows:

### 4.5.5 Key derivation

The private section is encrypted with a key derived as follows:

```
PrivateSectionKey = deriveKey(rootKey=SeedKey, name="dht-advertisement", salt=LookupToken || AdSalt)
```

### 4.5.6 Private section encryption

The private section ciphertext is produced by encrypting the plaintext with the `PrivateSectionKey`:

```
PrivateSectionPlaintext // constructed per table above

PrivateSectionCiphertext = symmetricEncryptAEAD(key=PrivateSectionkey, nonce=zeros(SYM_IV_LEN), plaintext=PrivateSectionPlaintext)
```

### 4.5.7 Assembled advertisement

The completed advertisement is assembled by concatenating the `PublicMutableSection`, `PublicImmutableSection` and `PrivateSection`:

```
AdText = PublicMutableSection || PublicImmutableSection || PrivateSection
```