# EasySafe attack sketch
## Determine names of files within filesystem

Adversary model: Blind

Complexity: K + P

## Method
The attacker must brute force the passphrase and learn the FSID through trial and error by attempting each candidate passphrase against every encrypted file to determine if that file is the configuration file.

The attacker does not possess any revision tags. So the attacker repeats this trial-and-error method against all pages of the filesystem to locate inode tables. The adversary then traverses the directory trees of each revision to learn the filenames.