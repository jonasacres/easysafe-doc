# 7. Filesystem threat model
## 7.2 Restricted model
### 7.2.1
The restricted threat model refers to an adversary whose capabilities are more limited compared to the basic model, with the following capability differences:

### 7.2.2 Restricted adversary capabilities

The restricted adversary possesses...

1. ...the ability to view, alter, extend, truncate, create and/or drop any packet sent to or from any EasySafe peer.
1. ...a copy of all data written to the EasySafe filesystem in encrypted form.
1. ...knowledge of the filesystem's `SeedKey` and FSID
1. ...knowledge of the version and full source code of any EasySafe application running on any peer
4. ...access to the `InodeTableOracle` function, by which the adversary can at no cost identify inode table pages to attack.
5. ...access to the `PageEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded pages with arbitrary plaintext.
6. ...access to the `PageTreeChunkEncryptionOracle` function, by which the adversary can at no cost craft arbitrary properly-encoded page tree chunks with arbitrary plaintext.
5. ...the ability to perform `PBKDF` operations with no cost.

### 7.2.3 Restricted adversary limitations

The restricted adversary has the same limitations as the basic adversary.

### 7.2.4 Restricted model security levels

| Task                                              | Complexity (log 2) | Notes    |
|---------------------------------------------------|--------------------|----------|
| Delete data from a filesystem                  | N/A | The adversary cannot delete data from peer storage, and all existing files are immutable by design.
| Insert data into a filesystem | K | The adversary can craft fake data using the `PageEncryptionOracle` function, but cannot craft a fake RevisionTag.
| Determine authorship of encrypted page[3]            | N/A | 
| Determine the contents of a page in the filesystem | K      | 
| Determine the number of files in the filesystem | 0 | Adversary can learn approximate number based on inode table size, learned through `InodeTableOracle`.
| Determine when a file is unlinked from the filesystem | K | 
| Determine the approximate size of a file in the filesystem | K[3] | Other than traffic analysis[3], best known option is brute force attack on inode table
| Determine names of files within filesystems | K | 
| Determine `stat_t`-like metadata concerning a file in the filesystem | K | Best known option is brute force attack on inode table
| Determine whether a given file is in the filesystem | K | 


\[1]: or K, if K is lesser.

\[2]: Adversary learns approximate number, and not exact figure.

\[3]: The adversary may be able to infer this information by acting as a seed and noting patterns in the upload and download of pages. These patterns vary depending on use case, and the adversary must be online as a seed to perform this method.
