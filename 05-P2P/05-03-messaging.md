# 5. Peer-to-peer Swarming
## 5.2 Messaging

### 5.2.1 Message structure
The plaintext structure of the messages exchanged by peers is beyond the scope of this document. A given plaintext message is encrypted using the CipherState objects produced per the Noise specification. This includes authenticated encryption proving message integrity.

Messages are transmitted as follows:

| Serialized message |
|-|
| Obfuscated Length (16-bit) |
| Ciphertext |

### 5.2.2 Ciphertext length obfuscation

The `FastHash` function is used to obfuscate the contents of length fields. This is done to prevent a casual observer from distinguishing a sample of EasySafe network traffic from random data by noting the regularity of length fields. This obfuscation works as follows:

```
Length // Ciphertext length, in bytes
sentByInitiator // True if message is to be sent from initiator to responder; false otherwise

if sentByInitiator:
  out = LenIvA = FastHash(LenKeyA, LenIvA)
else:
  out = LenIvB = FastHash(LenKeyB, LenIvB)
  
ObfuscatedLength = lenBytes ^ (out[0] << 8 | out[1]) // only use first 16 bits of output for XORing length field
```

#### Impracticality of concealing length
The purpose of obfuscating the ciphertext lengths is not to prevent a determined adversary from learning these lengths. In practice, an observer will most likely be able to infer the obfuscated lengths by observing the timing and structure of packets. The sole purpose of obfuscating the length fields to prevent a casual observer from distinguishiung EasySafe network traffic from random data.

### 5.2.3 Ciphertext

```
EncryptWithAd // Encryption function from appropriate CipherState, per Noise section 5.1
Plaintext // plaintext payload of message

ciphertext = EncryptWithAd(zeros(0), Plaintext) // EncryptWithAd defined in Noise section 5.1, using appropriate CipherState from handshake
Length = |ciphertext|(16) // used for ciphertext length obfuscation above
```

### 5.2.4 Rekeying
The `REKEY()` function defined by Noise (see sections 4.2, 5.1, 11.3 of Noise specification) is called after every call to `EncryptWithAd`.
