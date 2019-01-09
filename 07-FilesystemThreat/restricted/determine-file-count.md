# EasySafe attack sketch
## Determine number of files in filesystem

Adversary model: Restricted

Complexity: 0*

Caveat: Adversary is not certain of exact number, and only learns upper bound. Users can manipulate this upper bound at will.

## Method
Every file possesses an inode, listed in the inode table. The adversary is granted the ability to identify the number of pages in each revision of the inode table through the `InodeTableOracle` capability. The adversary is also assumed to possess all pages, including inode table pages.

The inode table has fixed-length inodes. By observing the number of pages in an inode table, and counting the number of inodes that can fit inside, the attacker can estimate the number of inodes in the filesystem.

Note that the adversary does not get an exact number, since the final page of the inode table may contain any number of inodes between 1 and the maximum number that fit per page. The inode table may also contain deleted inodes that have not yet been reissued.

Thus, the adversary learns an estimate that serves as an upper bound on the number of inodes genuinely in the filesystem.

This attack can be mitigated by creating many decoy files, or frequently unlinking files from the filesystem. This mitigation does not prevent the attacker from establishing a maximum for the number of files in the filesystem.
