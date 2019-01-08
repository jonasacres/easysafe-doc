# 7. Filesystem threat model

## 7.1 Basic model

### 7.1.1 Description
The basic threat model refers to an adversary with strong capabilities, including knowledge of keys used for data-in-transit. Additional threat models will consider additional guarantees that can be made when an adversary possesses fewer capabilities. The basic threat model is intended to consider the security guarantees offered by the file system, disregarding any network-level security.

### 7.1.2 Basic adversary capabilities

The basic adversary is assumed to possess...

1. ...the ability to view, alter, extend, truncate, create and/or drop any packet sent to or from any EasySafe peer.
1. ...the ability to view, alter, extend, truncate, create and/or delete any file directly created by EasySafe on any host filesystem anywhere in realtime.\[1]
2. ...knowledge of the contents of every file directly created by EasySafe on any host filesystem at any point in history.
1. ...knowledge of the filesystem's `SeedKey` and FSID
1. ...knowledge of the version and full source code of any EasySafe application running on any peer
2. ...knowledge of all key material used to encrypt network communications, including static keypairs, ephemeral keypairs, preshared secrets and any keys derived from aforementioned material.\[2]
3. ...knowledge of the identities of each peer
4. ...access to the `InodeTableOracle` function, by which the adversary can at no cost identify inode table pages to attack.
5. ...access to the `SuperPageEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded pages with arbitrary plaintext.
6. ...access to the `SuperPageTreeChunkEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded page tree chunks with arbitrary plaintext.
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

// allow adversary to craft an arbitrary encrypted page; distinguisher is randomly selected and not provided to adversary
PageEncryptionOracle(pageNum, pageText)

// allow adversary to craft an arbitrary encrypted page tree chunk; distinguisher is randomly selected and not provided to adversary
PageTreeChunkEncryptionOracle(chunkNum, chunkText)

```

\[1]: This does not extend to virtual memory dumps created by the OS that may include sensitive key material. Nothing in this threat model should be taken to indicate that EasySafe provides security against an adversary with physical or privileged access to a peer's system while it contains confidential material in plaintext.

\[2]: This does not apply to key material related to the filesystem. Specifically, the basic adversary does not have knowledge of the passphrase, `RootKey`, `FSKey`, or any keys derived from this material.

### 7.1.4 Basic adversary limitations

The adversary's capabilities are limited to those things listed above. The following non-comprehensive list of limitations is provided to minimize confusion.

The adversary does not possess...

1. ...advance knowledge of the read or write passphrase, `WritePrivateKey`, `RootKey` or `FSKey`.
2. ...the ability to read the contents of EasySafe's memory.
3. ...a technique for attacking any of the cryptographic primitives in less average time than a brute force attack.
4. ...information about the filesystem other than what is stated in the definition, written to disk or sent over the network. The adversary has knowledge of the sequence of operations, but not their exact timing.[3]

\[3] In reality, even casual observers almost certainly possess information about timing. Real adversaries may also possess other information such as sound, power utilization or EM radiation, any of which could enable a side channel attack. This restriction is intended to exclude matters of implementation and physical security from the analysis. Due care is taken to mitigate these attacks in implementation. Such implementation discussion is beyond the scope of this document.

### 7.1.5 Basic filesystem assumptions

The following assumptions are made of the structure of the filesystem.

1. The passphrase key contains K bits of entropy, K <= 256.
2. The filesystem has P files containing pages or page tree chunks unrelated to the inode table.
3. Any page within the filesystem possesses a minimum of F bits of entropy.

### 7.1.6 Basic security levels
EasySafe provides the following guarantees against the basic adversary. The adversary faces the following complexities:

| Task                                              | Minimum complexity (log 2) | Notes    |
|---------------------------------------------------|--------------------|----------|
| Delete data from a filesystem                  | 0 | The adversary can delete arbitrary pages from all peers, but does not have knowledge of what is being deleted beyond whether it is an inode table without performing a separate and more costly attack.
| Insert data into a filesystem | K | The adversary can craft fake data using the `SuperPageEncryptionOracle` function, but cannot craft a fake RevisionTag.
| Determine authorship of encrypted page            | 0 | Adversary has capability to observe first peer to transmit page
| Determine the contents of a page in the filesystem | (64 + F)\[1]      | Adversary brute forces guesses file contents and `distinguisher` through `SuperPageEncryptionOracle`
| Determine the number of files in the filesystem | 0\[2] | Adversary can learn approximate number based on inode table size, learned through `InodeTableOracle`. |
| Determine when a file is unlinked from the filesystem | (64 + F)\[1,3] | Best known option is brute force of root directory, assuming file is in root directory (worst case)
| Determine the specific size of a file in the filesystem | K[3] | Other than traffic analysis[3], best known option is brute force attack on inode table
| Determine names of files within filesystems | (64 + F)\[1] | Brute force of root directory
| Determine `stat_t`-like metadata concerning a file in the filesystem | K | Best known option is brute force attack on inode table
| Determine whether a given file is in the filesystem | (64 + P)\[1] | Adversary combines `SuperPageEncryptionOracle` with a brute-force attack on 64-bit distinguisher against each page observed in the filesystem


\[1]: or K, if K is lesser.

\[2]: Adversary learns approximate number, and not exact figure.

\[3]: The adversary may be able to infer this information by observing how often certain pages are downloaded. These patterns vary depending on use case.

