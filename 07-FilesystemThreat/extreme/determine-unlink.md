# EasySafe attack sketch
## Determine when a file is unlinked from the filesystem

Adversary model: Extreme

Complexity: 64 + F

## Method
Page files are not unlinked from the host filesystem when they are unlinked within a revision of the EasySafe encrypted filesystem. Thus, there is no outward evidence that a file has been unlinked.

The adversary may notice that certain encrypted pages are requested less often, as the adversary has the capability to view all requests made by all EasySafe peers. The request patterns of peers are not included in the model, and so this model cannot make concrete statements as to the complexity of this vector.

The model assumes that any file in the filesystem has F bits of entropy. Directories are files within EasySafe, and if a file is unlinked from the filesystem, it is removed from its parent directory. The adversary may attempt a brute force attack via the `SuperPageEncryptionOracle` of the directory containing the file believed to have been deleted. 

The attacker conducts this attack by combining guesses of the directory contents (F bits) with guesses of the distinguisher (64 bits) and checking the output of `SuperPageEncryptionOracle` to see if the result is included in the filesystem. This results in an attack with 64 + F bits of complexity.

