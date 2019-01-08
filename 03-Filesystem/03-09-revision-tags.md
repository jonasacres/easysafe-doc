# 3. Filesystem
## 3.9. RevisionTag

A RevisionTag provides an entry point into the filesystem, similar to a git commit hash. A RevisionTag is not a hash, and contains readable information. To prevent seed-only peers from learning information about the structure of the filesystem by observing RevisionTags, the content of the RevisionTag is obfuscated.

### 3.9.1 Plaintext

The plaintext of the RevisionTag contains the following fields:

| Field | Length (bytes) | Description |
|-------|----------------|-------------|
| refTag | HASH_LEN + 12 | RefTag produced by inode table |
| parentTag | 8 | First 8 bytes of RevisionTag of parent[1]
| height | 64 | 1 + height of parent |

[1] If a revision has multiple parents, the parentTag is calculated by hashing the concationation of the parent revtags in sorted order. If a revision has no parents, the blank RevisionTag is used.

[2] If a revision has multiple parents, the greatest parent height is used. If a revision has no parents, the height is set to 1.

### 3.9.2 Obfuscator

An obfuscator is derived from the plaintext as follows:

```
Obfuscator = HKDF(ikm=plaintext, length=16, salt=FSKey, info="revision-tag-obfuscator")
```

The obfuscator derives an obfuscation key:

```
ObfuscationKey = deriveSubkey(parentKey=FSKey, name="obfuscation-key", salt=Obfuscator)
```

##### Obfuscator collisions
The obfuscator is derived deterministically from the plaintext to ensure that a given RefTag always produces the same RevisionTag. The obfuscator is again combined with the `FSKey` to ensure that only peers possessing knowledge of the `FSKey` can decipher the RefTag.

There is a tradeoff between the length of the obfuscator (and therefore efficiency of representation), and the probability of two RevisionTags producing identical obfuscators. With a 128-bit obfuscator, this happens on average once per 2^64 RevisionTags. When RevisionTags share a common obfuscator, their confidentiality is lost. This will allow the adversary to learn the refTag, parentTag and height. In turn, this provides the adversary with at least one page file whose `Distinguisher` field is known, simplifying brute force attacks against the `FSKey`. This capability is modeled as the `InodeTableOracle` in the threat model later in this document.

If a filesystem produces a new revision every nanosecond, it will take an average of over 584 years to produce such a collision. This is deemed acceptable for most use cases. Nevertheless, the complexity is low enough that this operation is referred to as "obfuscation" rather than "encryption."

### 3.9.3 Ciphertext
The obfuscation key is used to encrypt the plaintext:

```
RevTagPlaintext // constructed according to table above
RevisionTagCiphertext = SymmetricEncrypt(key=ObfuscationKey, iv=zeros(SYM_IV_LEN), data=RevTagPlaintext)
```

### 3.9.4 Signature
The resulting ciphertext is then signed using the `WritePrivateKey`. This is done to allow seed-only peers to authenticate RevisionTags without knowledge of the `FSKey`.

```
RevisionTagSignature = AsymmetricSign(key=WritePrivateKey, data=RevisionTagCiphertext)
```

### 3.9.5 Assembled RevisionTag
The obfuscator, ciphertext and signature are combined to produce the RevisionTag:

```
RevisionTag = obfuscator || ciphertext || signature
```

### 3.9.6 Revision tag distribution
RevisionTags are stored locally as entry points into the filesystem. Additionally, each filesystem revision contains a special file referred to as the "Revision Info" which contains a list of the RefTags of each parent revision.

RevisionTags are shared on the peer-to-peer network, including to seed-only peers.
