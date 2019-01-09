# 7. Filesystem threat model
## 7.3 Blind adversary
### 7.3.1
The blind adversary model refers to an adversary whose capabilities are similar to those of someone who has recovered EasySafe without possessing any special information about it, such as its `SeedKey` or FSID.

This would be the case in incidents where an adversary recovers a hard disk belonging to an EasySafe peer, but does not have any particular knowledge about it.

These capabilities are intended to be a superset of the cabilities of a real-world attacker who has:

* Possession of all encrypted data

### 7.2.2 Blind adversary capabilities

The blind adversary possesses...
1. ...a copy of all the encrypted pages and page tree chunks that compose the filesystem.
2. ...knowledge of the version and full source code of the version of EasySafe used to produce each encrypted file.
3. ...the ability to perform `PBKDF` operations with no cost.

