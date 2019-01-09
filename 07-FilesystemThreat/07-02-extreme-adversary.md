# 7. Filesystem threat model

## 7.1 Extreme adversary model model

### 7.1.1 Description
A threat model describing an adversary with strong capabilities is defined here as the "Extreme threat model."

The extreme adversary model refers to an adversary with comprehensive control over the network and storage operations of all peers, as well as access to very potent chosen-plaintext attacks and considerable insight into the structure of the archive and the identities of peers.

These capabilities are intended to be a superset of the capabilities of a real-world attacker who has:

* Complete control over the networking of one or more peers
* Read-write access to the storage devices of one or more peers
* Possession of the FSID, `SeedKey`, and all encrypted pages
* The ability to influence the RNG output of one or more peers, combined with the ability to conduct chosen-plaintext attacks using that peer

### 7.1.2 Extreme adversary capabilities

The extreme adversary is assumed to possess...

1. ...the ability to view, alter, extend, truncate, create and/or drop any packet sent to or from any EasySafe peer.
2. ...knowledge of all key material used to encrypt network communications, including static keypairs, ephemeral keypairs, preshared secrets and any keys derived from aforementioned material.\[2]
1. ...the ability to view, alter, extend, truncate, create and/or delete any file directly created by EasySafe on any host filesystem anywhere in realtime.\[1]
2. ...knowledge of the contents of every file directly created by EasySafe on any host filesystem at any point in history.
1. ...knowledge of the filesystem's `SeedKey` and FSID
1. ...knowledge of the version and full source code of any EasySafe application running on any peer
3. ...knowledge of the identities of each peer
4. ...access to the `InodeTableOracle` function, by which the adversary can at no cost identify inode table pages to attack.
5. ...access to the `SuperPageEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded pages with arbitrary plaintext, and also fix the 64-bit random `distinguisher`.
6. ...access to the `SuperPageTreeChunkEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded page tree chunks with arbitrary plaintext, and also fix the 64-bit random `distinguisher`.
5. ...the ability to perform `PBKDF` operations with no cost.

### 7.1.3 Oracle definitions

```
// allow adversary to determine if a given tag belongs to an inode table
InodeTableOracle(tag, contents):
  isInodeTable // set true <=> the file belongs to an inode table
  pageNum // set to page number of this inode table page if isInodeTable, else undefined
  refTag // set to the refTag of the inode table this page belongs to if isInodeTable, else undefined
  return [isInodeTable, refTag, pageNum]

// allow adversary to craft an arbitrary encrypted page for a pre-specified distinguisher
SuperPageEncryptionOracle(distinguisher, pageNum, pageText)

// allow adversary to craft an arbitrary page tree chunk
SuperPageTreeChunkEncryptionOracle(distinguisher, chunkNum, chunkText)

```

\[1]: This does not extend to virtual memory dumps created by the OS that may include sensitive key material. Nothing in this threat model should be taken to indicate that EasySafe provides security against an adversary with physical or privileged access to a peer's system while it contains confidential material in plaintext.

\[2]: This does not apply to key material related to the filesystem. Specifically, the extreme adversary does not have knowledge of the passphrase, `RootKey`, `FSKey`, or any keys derived from this material.

### 7.1.4 Extreme adversary limitations

The adversary's capabilities are limited to those things listed above. The following non-comprehensive list of limitations is provided to minimize confusion.

The adversary does not possess...

1. ...advance knowledge of the read or write passphrase, `WritePrivateKey`, `RootKey` or `FSKey`.
2. ...the ability to read the contents of EasySafe's memory.
3. ...a technique for attacking any of the cryptographic primitives in less average time than a brute force attack.
4. ...information about the filesystem other than what is stated in the definition, written to disk or sent over the network. The adversary has knowledge of the sequence of operations, but not their exact timing.[3]

\[3] In reality, even casual observers almost certainly possess information about timing. Real adversaries may also possess other information such as sound, power utilization or EM radiation, any of which could enable a side channel attack. This restriction is intended to exclude matters of implementation and physical security from the analysis. Due care is taken to mitigate these attacks in implementation. Such implementation discussion is beyond the scope of this document.
