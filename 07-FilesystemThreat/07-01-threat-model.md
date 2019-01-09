# 7. Filesystem Threat Model
## 7.1 Overview of threat model

### 7.1.1 Model design
The model presented here describes a variety of tasks to be performed in the filesystem, such as reading an encrypted file or determining the number of files in the filesystem. Each task is something that is not possible to any party, or is only possible with knowledge of the appropriate passphrase or key. Thus, each task requires some form of attack.

The model also defines a set of adversaries, each with formally-specified capabilities. For each defined task, a sketch of an attack is presented describing the best-known method for an attacker to complete the task in the lowest possible time. This time is listed as the "complexity" of an attack, and is defined as the base-2 logarithm of the maximum number of steps required by any portion of the attack.

Finally, the model provides a set of assumptions about the filesystem, to be used in the attack sketches to describe the difficulty of various basic attacks such as brute forces.

A table is presented on this page showing the complexity of each attack for each task and adversary, with links to the relevant attack sketches.

### 7.1.2 Nature of security claims
Due to the nature of this methodology, it is not possible to make absolute claims on the minimum complexity of each attack. The attack sketches presented here are not formal security proofs providing mathematical arguments for a minimum complexity for each attack. Instead, the sketches demonstrate a method for performing an attack with a certain maximum complexity.

These results are presented to claim that these are the best known attacks available against EasySafe's security, and to encourage further review.

It is possible that better attacks can be formulated. The reader is encouraged to suggest superior attacks.


### 7.1.3 Adversaries
| Adversary name | Chapter | Description
|----------------|---------|-------------
| Extreme        | 7.2     | Universal network and storage access with extensive chosen-plaintext capabilities
| Restricted     | 7.3     | Attacker with knowledge of `FSID` and `SeedKey` and moderate chosen-plaintext capabilities
| Blind          | 7.4     | Attacker possesses encrypted pages, but has no further information about them

### 7.1.4 Filesystem assumptions

1. The read and write passphrase each have K bits of entropy.
2. Each file in the filesystem contains F bits of entropy.
3. The number of EasySafe encrypted files on the storage device (pages, page tree chunks and configuration file) is 2^P.

### 7.1.5 Tasks

| Task | Blind | Restricted | Extreme
|--|--|--|--
| Determine the contents of a page in the filesystem | [K+P](blind/read-page-contents#easysafe-attack-sketch.md) | [K](restricted/read-page-contents.md#easysafe-attack-sketch) | [64 + F](extreme/read-page-contents.md#easysafe-attack-sketch) |
| Determine whether a given file is in the filesystem | [K+P](blind/determine-file-existence.md#easysafe-attack-sketch) | [K](restricted/determine-file-existence.md#easysafe-attack-sketch) | [K](extreme/determine-file-existence.md#easysafe-attack-sketch) |
| Delete data from the filesystem | [Infeasible](blind/delete-data) | [Infeasible](restricted/delete-data) | [0](extreme/delete-data) |
| Insert data into the filesystem | [K + P](blind/add-data.md#easysafe-attack-sketch) | [K](restricted/add-data.md#easysafe-attack-sketch) | [K](extreme/add-data.md#easysafe-attack-sketch) |
| Modify existing data in the filesystem | TODO | TODO | TODO
| Determine authorship of encrypted page | [Infeasible](blind/determine-authorship.md#easysafe-attack-sketch) | [0*](restricted/determine-authorship.md#easysafe-attack-sketch) | [0*](extreme/determine-authorship.md#easysafe-attack-sketch) |
| Determine the number of files in the filesystem | [K + P](blind/determine-file-count.md#easysafe-attack-sketch) | [0*](restricted/determine-file-count.md#easysafe-attack-sketch) | [0*](extreme/add-data.md#easysafe-attack-sketch) |
| Determine when a file is unlinked from the filesystem | [K + P](blind/determine-unlink.md#easysafe-attack-sketch) | [K](restricted/determine-unlink.md#easysafe-attack-sketch) | [K](extreme/determine-unlink.md#easysafe-attack-sketch) |
| Determine the specific size of a file in the filesystem | [K + P](blind/determine-file-size.md#easysafe-attack-sketch) | [K](restricted/determine-file-size.md#easysafe-attack-sketch) | [F + 64](extreme/determine-file-size.md#easysafe-attack-sketch) |
| Determine names of files within filesystems | [K + P](blind/determine-filename.md#easysafe-attack-sketch) | [K](restricted/determine-filename.md#easysafe-attack-sketch) | [64 + F](extreme/determine-filename.md#easysafe-attack-sketch) |

Note that for all tasks, there is a maximum complexity based on the passphrase entropy K. In the Blind model, the maximum complexity is K + P. In the Restricted and Extreme models, the maximum complexity is K, due to these adversaries having access to the `InodeTableOracle`.

If the stated complexity is less than the maximum complexity, then the maximum complexity should be assumed as the true complexity of the attack.