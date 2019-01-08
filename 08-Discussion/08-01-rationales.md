# 8. Discussion
## 8.1 Rationales

### 8.1.1 Why was the c5.18xlarge chosen in the Argon2 model?
Several other instance classes, including the z1d.12xlarge were considered. In general, high-end instances had similar levels of performance once the number of vCPUs were taken into account, and the c5.18xlarge had the best performance per dollar of anything tested. Taking this instance at its lowest available price seems to be likely to get at least a rough picture of what an adversary can do today with easily-available resources.

The adversary in the model is presumed to perform 100x more cost-effectively than the c5.18xlarge instance, to ensure that the adversary's capabilities are overestimated even if there is a more efficient option available today.

### 8.1.2 Why encode the filesystem config tag and page length in the FSID?
Peers receiving a filesystem for the first time need certain bootstrap info. The swarm protocol relies on parties agreeing on page size in advance. Although this is in the filesystem config, the filesystem config itself must be transmitted through the protocol. By encoding the page size directly into the ID, naive peers can immediately join in the swarm. By also basing the ID on the config tag, the configuration file can be validated to prove that it corresponds to the intended FSID.

### 8.1.3 Why use symmetric encryption to obfuscate ephemeral keys instead of something like Elligator2?
As of this writing, Elligator2 support is difficult to find. The symobf technique is similar to the methodology used in the i2p project's NTCP2 protocol.

### 8.1.4 Why not use ephemeral keys for the DHT protocol?
The DHT protocol is stateless, and also very size constrained. At present, DHT messages are limited to a maximum size of 508 bytes. Using an ephemeral key would consume additional space in the packet, and require a significant increase in the CPU load created by the protocol due to the extra Diffie-Hellman operation.

### 8.1.5 What is the purpose of encrypting RevisionTags?
Seed peers must be able to receive RevisionTags so they can be relayed to read-access peers. If the revision tags were not encrypted, the seed peers would gain unnecessary insight into the structure of the filesystem.

### 8.1.6 Why does taggedEncrypt do encrypt-then-MAC instead of an AEAD cipher?
The resulting tag would not be verifiable by seed peers, who do not possess the symmetric key used to produce the ciphertext. The MAC is calculated using a key derived from the seed root. Using an AEAD cipher in addition to this MAC would be an additional CPU cost, when authenticity and integrity are already guaranteed.

### 8.1.7 Why does taggedEncrypt use both an encrypt-then-MAC hash and an asymmetric signature?
A peer processing an incoming page or chunk must be able to provde three things, using knowledge of the seed key and public write key:

1. The text has not been tampered with.
2. The text was created by someone with knowledge of the private write key.
3. The text belongs to this filesystem, and not another filesystem that the write key may have been used for.

The asymmetric signature proves both 1) and 2), whereas the authenticated hash proves 1) and 3). Furthermore, the authenticated hash can be validated much more quickly, offering a reduced-cost way to perform integrity checking over a large batch of data.

### 8.1.8 Why not support de-duplication?
The use of page- and inode-specific derived keys with random salts provides resistance against the chosen plaintext capabilities granted in the threat models (e.g. `PageEncryptionOracle`).

Later versions may support page-independent keys with deterministic IVs as an option for users who desire de-duplication.

### 8.1.9 What if I want to revoke someone's access to a filesystem, or change the passphrase?
Filesystem passphrases are immutable. One option is to migrate the data in the filesystem to one with a new passphrase. This passphrase would then need to be distributed to the authorized parties. The means for doing this are beyond the scope of this document.

### 8.1.10 Why are side channel attacks excluded from the models?
Side channel attacks are very difficult to model. Although design decisions can make a system more or less resistant to some side channel attacks, no design can possibly foil all of them. Major classes of security issues come down to hardware, implementation and physical security concerns.

Certain real-world adversaries possess capabilities that go beyond the security model here and can easily break it. For instance, EasySafe has no means of protecting you if you are imprisoned and tortured until you reveal your passphrase. EasySafe cannot protect you from adversaries that participate in the thriving clandestine market for undisclosed exploits that will allow them access to your system. It also can't protect you if your adversary has access to a secret backdoor built into your CPU by the manufacturer.
