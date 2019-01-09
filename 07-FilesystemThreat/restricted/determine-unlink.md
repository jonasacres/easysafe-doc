# EasySafe attack sketch
## Determine when a file is unlinked from the filesystem

Adversary model: Restricted

Complexity: K

## Method
The adversary brute forces the passphrase and reads the filesystem.

The adversary may notice that certain encrypted pages are requested less often, as the adversary has the capability to view all requests made by all EasySafe peers. The request patterns of peers are not included in the model, and so this model cannot make concrete statements as to the complexity of this vector.
