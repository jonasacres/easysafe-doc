# EasySafe attack sketch
## Determine whether a given file is in the filesystem

Adversary model: Extreme

Complexity: 64

## Method
The model grants the adversary strong chosen plaintext capabilities via the `SuperPageEncryptionOracle`. The attacker uses this to encrypt the given file with brute-force guesses of the 64-bit distinguisher, and check if the resulting ciphertext is held by any peer.