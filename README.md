
# EasySafe

EasySafe is a distributed, encrypted filesystem. The filesystem can be read and written to based on knowledge of a passphrase. Peers hosting the filesystem can also be located on a decentralized network using the passphrase.

This document explains the cryptography used to encrypt files, locate peers and communicate with them.

This document is currently in an unstable draft status, and reflects pre-release software. It is therefore incomplete and highly subject to change.

## The good stuff

I want to know...

| ... | Chapter
|--|--|
| Which algorithms are used? | [2.2](02-Conventions/02-02-crypto.md#22-cryptographic-function-specifications) |
| What is feasibility of brute force attacks on the passphrase? | [6.2](06-PBKDF/06-02-threat-model.md#6-passphrase-based-key-derivation) |
| What is the threat model for attacks against the filesystem? | [7.1](07-FilesystemThreat/07-01-threat-model.md#71-overview-of-threat-model)
| How are keys derived? | [3.2](03-Filesystem/03-02-key-derivations.md#3-filesystem) |
| How are files encrypted? | [3.1](03-Filesystem/03-01-tagged-encryption.md#2-filesystem), [3.7](03-Filesystem/03-07-pages.md#3-filesystem) |
| How are DHT messages encrypted? | [4.3](04-DHT/04-03-message-crypto.md#4-distributed-hash-table) |
| How do network peers exchange keys? | [5.2](05-P2P/05-02-handshake.md#5-peer-to-peer-swarming) |
| How are network messages encrypted? | [5.3](05-P2P/05-03-messaging.md#5-peer-to-peer-swarming) |

## Table of contents

### Chapter 1: About
| Chapter | Title | Description
|-|-|-|
| 1.1 | [About this document](01-Document/01-01-about.md) | Author, copyright and IP information

### Chapter 2: Conventions
| Chapter | Title | Description
|-|-|-|
| 2.1 | [Operators](02-Conventions/02-01-operators.md) | Definitions of operators used throughout the documentation
| 2.2 | [Cryptographic function specifications](02-Conventions/02-02-crypto.md) | Definitions of cryptographic functions used throughout the documentation
| 2.3 | [Constants](02-Conventions/02-03-constants.md) | Definitions of constants used throughout the documentation
| 2.4 | [Functions](02-Conventions/02-04-functions.md) | Definitions of functions used throughout the documentation

### Chapter 3: Filesystem
| Chapter | Title | Description
|-|-|-|
| 3.1 | [Tagged Encryption](03-Filesystem/03-01-tagged-encryption.md) | Encryption methodology used to store individual encrypted files
| 3.2 | [Key Derivations](03-Filesystem/03-02-key-derivations.md) | Explanation of how keys are derived for use throughout EasySafe
| 3.3 | [Configuration](03-Filesystem/03-03-config.md) | Encryption methodology used to produce filesystem-specific configuration file
| 3.4 | [FSID](03-Filesystem/03-04-fsid.md) | Methodology for deriving unique filesystem identifier used to locate filesystem on decentralized network
| 3.5 | [Inode Table](03-Filesystem/03-05-inode-table.md) | Discussion of filesystem inode table
| 3.6 | [RefTag](03-Filesystem/03-06-reftag.md) | Discussion of RefTag object, used in encrypted EasySafe data structures
| 3.7 | [Pages](03-Filesystem/03-07-pages.md) | How plaintext files are converted to pages and encrypted
| 3.8 | [Page Trees](03-Filesystem/03-08-page-trees.md) | Data structure used to locate large files in EasySafe
| 3.9 | [Revision Tags](03-Filesystem/03-09-revision-tags.md) | Discussion of Revision Tag object, used to describe filesystem states

### Chapter 4: Distributed hash table
| Chapter | Title | Description
|-|-|-|
| 4.1 | [Protocol](04-DHT/04-01-protocol.md) | Overview of DHT protocol
| 4.2 | [Temporal lookup keys](04-DHT/04-02-temporal-lookup-keys.md) | Discussion of short-lived filesystem-specific identifier sought in DHT
| 4.3 | [Message-level cryptography](04-DHT/04-03-message-crypto.md) | How messages between DHT peers are encrypted
| 4.4 | [Lookup tokens](04-DHT/04-04-lookup-tokens.md) | Peer-specific token used to prevent unauthorized DHT lookups
| 4.5 | [Advertisements](04-DHT/04-05-advertisements.md) | Data structure used in DHT search results

### Chapter 5: Peer-to-peer Swarm
| Chapter | Title | Description |
|-|-|-|
| 5.1 | [Protocol](05-P2P/05-01-protocol.md) | Peer-to-peer messaging protocol for exchanging filesystem data
| 5.2 | [Handshake](05-P2P/05-02-handshake.md) | Noise-based protocol for establishing secure connection between peers
| 5.3 | [Messaging](05-P2P/05-03-messaging.md) | Message-level cryptography for swarming protocol

### Chapter 6: Passphrase-based Key Derivation
| Chapter | Title | Description
|-|-|-|
| 6.1 | [Parameters](06-PBKDF/06-01-parameters.md) | Explanation of selection process for Argon2 parameters
| 6.2 | [Threat model](06-PBKDF/06-02-threat-model.md) | Calculation of life expectancy for passphrases of various strengths against model adversary

### Chapter 7: Filesystem Threat Model
| Chapter | Title | Description
|-|-|-|
| 7.1 | [Overview of Threat Model](07-FilesystemThreat/07-01-threat-model.md#71-overview-of-threat-model) | Threat model and summary of analysis
| 7.2 | [Extreme Adversary](07-FilesystemThreat/07-02-extreme-adversary.md) | Analysis of filesystem security guarantees assuming an adversary with very strong capabilities
| 7.3 | [Restricted Adversary](07-FilesystemThreat/07-03-restricted-adversary.md) | Analysis of filesystem security guarantees assuming an adversary with a lesser set of capabilities compared to the extreme model
| 7.4 | [Blind Adversary](07-FilesystemThreat/07-04-blind-adversary.md) | Analysis of filesystem security guarantees assuming an adversary with possession of encrypted files and no special knowledge

### Chapter 8: Rationales
| Chapter | Title | Description
|-|-|-|
| 8.1 | [Rationales](08-Discussion/08-01-rationales.md) | FAQ-like discussion of various design decisions
