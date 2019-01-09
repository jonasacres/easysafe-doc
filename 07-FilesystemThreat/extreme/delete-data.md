# EasySafe attack sketch
## Delete data from a filesystem

Adversary model: Extreme

Complexity: 0

Caveat: Attacker does not know what is being deleted, unless what is deleted is an inode table or config file.

## Method
Additional assumption: EasySafe peers do not have any backups or other storage that the adversary lacks write access to.

The attacker is assumed in the model to have complete access to the storage of all peers. The attacker uses this capability to delete arbitrary page, chunk or config files from all peers. This can be combined with the attacker's `InodeTableOracle` capability to delete specific revisions from the filesystem.

The attacker could also instead delete the configuration file. For non-default archives, this could prevent all future access to the archive, unless peers 