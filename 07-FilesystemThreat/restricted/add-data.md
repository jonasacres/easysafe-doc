# EasySafe attack sketch
## Insert data into a filesystem

Adversary model: Restricted

Complexity: K

## Method
The attacker is able to craft fake pages and page tree chunks using the `PageEncryptionOracle` and `PageTreeChunkEncryptionOracle` capabilities. The attacker could use these to craft false files and page trees. It is still necessary to place the RefTags for these falso files into an inode, but the adversary does not have the means to decrypt the inode table.

The adversary can craft a false inode table as well. However, this inode table will only contain the attacker's false data, since the attacker does not know the content of the genuine inodes.

Such a revision might be automatically merged in with a genuine inode table if distributed on the network. This would require the adversary to construct a revision tag. Knowledge of the `WriteKey` is needed to create a revision tag, which the adversary does not possess, and no equivalent oracle is provided in the model.

Therefore, the attacker must attempt a brute force of the `WriteKey`. This has complexity `K`, per the assumptions of the model.

