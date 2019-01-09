# EasySafe attack sketch
## Determine names of files within filesystem

Adversary model: Extreme

Complexity: (64 + F)

## Method
The model assumes all files have F bits of entropy. The model also grants the basic adversary significant chosen-plaintext capabilities in the form of `SuperPageEncryptionOracle`.

The adversary attempts to brute force directory structures via `SuperPageEncryptionOracle` and checks to see if the resulting ciphertext exists on any peer. Per the model assumption, this has a complexity of F bits, plus the 64-bit distinguisher.

