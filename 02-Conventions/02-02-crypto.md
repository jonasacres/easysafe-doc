# 2. Conventions
## 2.2. Cryptographic function specifications

EasySafe relies on peer-reviewed, publicly-disclosed algorithms for all cryptography. Those algorithms are listed below. Each algorithm has a name, by which it is referred throughout this document.

| Name | Algorithm | Length (bits) |
|------|--------------------|--------------|
| Hash(data) | Blake2b            | 512         
| FastHash(key, data) | SipHash-2-4 | 64
| HMAC(key, data)| As specified in RFC 2104           | 
| HKDF(ikm, length, salt, info) | As specified in RFC 5869     | 
| SymmetricEncrypt(key, iv, plaintext) | ChaCha20 | 256 key / 96 nonce
| SymmetricEncryptAEAD(key, iv, plaintext) | ChaCha20-Poly1305 | 256 key / 96 nonce / 128 tag
| AsymmetricSharedSecret(privateKey, publicKey) | X25519 | 256
| AsymmetricSign(privateKey, data) | Ed25519 | 512
| AsymmetricVerify(publicKey, signature, data) | Ed25519 | 256
| AsymmetricDerivePublic(privateKey) | Ed25519 | 256
| PBKDF(passphrase) | Argon2 | 256

#### 2.2.3 Argon2 settings

The parameters chosen for all Argon2 operations are as follows:

| Argon2 Setting | Value                  |
|----------------|------------------------|
| mode           | Argon2i                |
| salt           | "easysafe-argon2-salt" |
| parallelism    | 16 threads             |
| tagLength      | 256 bits               |
| memorySize     | 1 GiB                  |
| iterations     | 40                     |
| version        | 13                     |


