# EasySafe attack sketch
## Determine whether a given file is in the filesystem

Adversary model: Blind

Complexity: K+P

## Method
The attacker must brute force the passphrase and learn the FSID through trial and error by attempting each candidate passphrase against every encrypted file to determine if that file is the configuration file.

The attacker does not possess any revision tags. So the attacker repeats this trial-and-error method against all pages of the filesystem to locate inode tables. The adversary now has the ability to read the contents of every file, and can determine if the file is included.