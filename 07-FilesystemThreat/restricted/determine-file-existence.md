# EasySafe attack sketch
## Determine whether a given file is in the filesystem

Adversary model: Restricted

Complexity: K

## Method
The adversary's chosen plaintext oracles are insufficient to mount a meaningful, as the restricted adversary's `PageEncryptionOracle` does not possess the ability to dictate the random 64-bit distinguisher used in the file encryption, nor does the oracle reveal which distinguisher was used.

Thus, the adversary would need to make many requests to the oracle until either a duplicate of an existing page was discovered (proving the first page of the file is in the filesystem), or 2^64 distinct results had been gathered (showing the file is not in the filesystem).

Per the coupon collector's problem, this would take more than 2^266 attempts. Since K <= 256, the adversary is better off attempting a brute force attack of the passphrase.