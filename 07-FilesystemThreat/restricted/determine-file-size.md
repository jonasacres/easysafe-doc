# EasySafe attack sketch
## Determine the specific size of a file in the filesystem

Adversary model: Restricted

Complexity: K

Caveat: The attacker knows the file size cannot exceed the size of the archive, less the size of inode tables and the configuration file.

## Method
The attacker brute forces the passphrase and gauges the file size from the inode table.