# EasySafe attack sketch
## Insert data into a filesystem

Adversary model: Blind

Complexity: K + P

## Method
The attacker brute forces candidate passphrases and tries them against each page until the configuration file is discovered. The adversary can then create a blank inode table containing the new data.