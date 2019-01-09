# EasySafe attack sketch
## Determine the specific size of a file in the filesystem

Adversary model: Extreme

Complexity: min(K, 64 + F)

## Method
Since all files are assumed to have F bits of entropy, the adversary can use the `SuperPageEncryptionOracle` capability to attempt brute force attacks of either the inode table or the file itself.

The attacker combines guesses of the 64-bit distinguisher and file plaintext, and computes the ciphertext of each through `SuperPageEncryptionOracle`. The adversary also has knowledge of all encrypted files stored in any EasySafe peer, and therefore knows if the resulting ciphertext already exists in the filesystem.