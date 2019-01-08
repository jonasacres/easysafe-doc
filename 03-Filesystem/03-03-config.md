# 3. Filesystem

## 3.2 Configuration file
## 3.2.1 Default filesystems
Each filesystem has a configuration file containing various parameters. For instance, the write passphrase can optionally be chosen independently of the read passphrase. There are default values for each configuration parameter. When the defaults are chosen for all parameters, the configuration file is deterministic from the passphrase. Such a filesystem is referred to as a "default filesystem." These filesystems can be located in the decentralized network solely with knowledge of their passphrase.

## 3.2.2 Configuration layout

The configuration file is laid out as follows:

|         |
|---------|
| Version hash |
| Salt         |
| Seed ciphertext |
| Secure ciphertext |
| Padding |
| Signature |

## 3.2.3 Version hash

The version hash is produced as follows:

```
VersionHash = HMAC(key=SeedRoot, data="easysafe-version:0")
```

## 3.2.4 Salt

The salt is generated using an authenticated hash of the following:

| SaltInput |
|-|
| Seed Section Plaintext Length (16 bits) |
| Seed Section Plaintext |
| Secure Section Plaintext Length (16 bits) |
| Secure Section Plaintext |

This information is then authenticated to produce the salt:
```
Salt = HMAC(key=SeedKey, data=SaltInput)
```

## 3.2.5 Seed ciphertext
The plaintext structure of the seed section is beyond the scope of this document. It includes `PAGE_SIZE` and `WritePublicKey`, as well as any other filesystem parameters that must be known to parties possessing the `SeedKey`.

The seed ciphertext key is derived as follows:

```
SeedCiphertextKey = deriveSubey(parentKey=SeedKey, name="SeedCiphertextKey", salt=VersionHash || Salt)
```

The key is then used to produce the secure ciphertext:

```
SeedCiphertext = SymmetricEncryptAEAD(key=SeedCiphertextKey, nonce=zeros(SYM_IV_LEN), data=SeedPlaintext)
```

## 3.2.6 Secure ciphertext
The plaintext structure of the secure section is beyond the scope of this document. The secure section includes the `FSKey`, and any filesystem parameters that should be known only to holders of the read passphrase. The secure ciphertext key is derived as follows:

```
SecureCiphertextKey = deriveSubey(parentKey=RootKey, name="SecureCiphertextKey", salt=VersionHash || Salt || SeedCiphertext)
```

The key is then used to produce the secure ciphertext:

```
SecureCiphertext = SymmetricEncryptAEAD(key=SecureCiphertextKey, nonce=zeros(SYM_IV_LEN), data=SecurePlaintext)
```

## 3.2.7 Padding

The padding section ensures that the overall length of the configuration file is equal to `PAGE_SIZE + ASYM_SIG_LEN`. Thus, it has a length of:

```
PaddingLen = PAGE_SIZE + ASYM_SIG_LEN - |VersionHash| - |Salt| - |SeedCiphertext| - |SecureCiphertext|
```

The padding may take any value, subject to the requirement that it appear statistically indistinguishable from random to a party that does not have knowledge of the read passphrase. By default, the padding is constructed deterministically as follows:

```
paddingKey = deriveSubkey(parentKey=RootKey, name="PaddingKey", salt=VersionHash || Salt || SeedCiphertext || SecureCiphertext)
Padding = SymmetricEncrypt(key=paddingKey, nonce=zeros(SYM_IV_LEN), data=zeros(PaddingLen))
```

## 3.2.8 Signature
A public key signature is calculated over the preceding sections using the `WritePrivateKey` as follows:

```
unsignedFile = VersionHash || Salt || SeedCiphertext || SecureCiphertext || Padding
Signature = AsymmetricSign(key=WritePrivateKey, data=unsignedFile)
```

## 3.2.9 Assembled configuration file
The above sections are concatenated together to form the completed configuration file:

```
ConfigFile = VersionHash || Salt || SeedCiphertext || SecureCiphertext || Padding || Signature
```
