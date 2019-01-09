# EasySafe attack sketch
## Read contents of encrypted page in filesystem

Adversary model: Extreme

Complexity: 64 + F

## Method
The page is encrypted using a combination of the FSID (known to the attacker), and the 64-bit distinguisher, page number and file contents (not known to the attacker). The adversary does not have knowledge of the `FSKey` needed to decipher the encrypted page.

The adversary does possess access to a chosen plaintext oracle in the form of the `SuperPageEncryptionOracle` capability. This can be used to encrypt trial candidates of page 0 of the file with arbitrary distinguishers. The attacker can then check if the resulting page already exists in the filesystem.

Both the guess for the distinguisher and the page contents must match an existing page exactly. Since the distinguisher has 64 bits of entropy, and the file is assumed to have F bits, there is therefore 64 + F bits of secret material that must be brute-forced by the attacker via the oracle.

Given the model assumption that all files in the filesystem have F bits of entropy, the adversary may be best-served to brute force the inode table, and therefore learn the distinguishers of all files. This assumption is unrealistic since in practice, the adversary must guess each distinguisher in order to guess the contents of the inode table.
