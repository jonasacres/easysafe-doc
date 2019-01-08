# 5. Peer-to-peer Swarming
## 5.2 Handshaking

### 5.2.1 Noise pattern
Handshaking and message encryption follow the standards of the Noise specification, as of Revision 34.[TODO cite] The protocol name is `Noise_XKpsk4_25519_ChaChaPoly_BLAKE2b`.

The handshake is expressed in Noise pattern notation as follows:
```
// PSK is FSID
// Responder static key learned by initiator from DHT ad
// | denotes message payload, which is to be encrypted per Noise specification
XKpsk4:
  <- s
   ...
  -> e, es
  <- e, ee
  -> s, se | PayloadA
  <- psk   | PayloadB
```

### 5.2.2 XKpsk4 designation
The Noise specification does not contain a "XKpsk4" pattern. This key exchange follows the standard XKpsk3 pattern, with the exception that the psk is moved to a new fourth round.

This is because the responder may be listening for multiple filesystems using the same port and public static key. Since the FSID is used as a pre-shared key, the responder must receive information from the requestor in order to determine which FSID to use. The requestor cannot safely transmit this information until the payload of round 3 (the `PayloadA` section, described below).

In the standard "XKpsk3" pattern, the `psk` token would have been processed prior to encrypting `PayloadA`, meaning the responder would be unable to determine which PSK to use. Therefore the `psk` token is delayed until the next round.

### 5.2.3 Symmetric obfuscation of ephemeral public key
During the handshake, the initiator's public ephemeral key is transmitted in the clear. Ed25519 public keys are distinguishable from random. To prevent a casual observer from distinguishing EasySafe traffic from random, the initiator's public ephemeral key is obfuscated using a key derived from the responder's public static key:

```
  ResponderPublicStaticKey // public static key belonging to responder; known to requestor from ad
  InitiatorPublicEphemeralKey // public ephemeral key belonging to initiator
  
  random = RNG(32)
  obfKey = deriveSubkey(parentKey=ResponderPublicStaticKey, name="handshake-ephemeral-key-obfuscation", salt=random)
  ObfuscatedEphemeralKey = random || SymmetricEncryptAEAD(key=obfKey, iv=zeros(SYM_IV_LEN), data=InitiatorPublicEphemeralKey)
```

The `ObfuscatedEphemeralKey` is what is transmitted when processing the `e` token of round 1 of the Noise handshake.

### 5.2.4 Additional key material derivation
Length obfuscation (explained below) requires the derivation of additional key material during handshaking. This is done via a modification to the `Split()` function of Noise, as defined in section 5.2 of the specification:

```
Split(): Returns a pair of CipherState objects for encrypting transport messages. Executes the following steps, where zerolen is a zero-length byte sequence:

    Sets temp_k1, temp_k2 = HKDF(ck, zerolen, 2). // see Noise spec for details on Noise-specific HKDF semantics
    If HASHLEN is 64, then truncates temp_k1 and temp_k2 to 32 bytes.
    Creates two new CipherState objects c1 and c2.
    Calls c1.InitializeKey(temp_k1) and c2.InitializeKey(temp_k2).
    Returns the pair (c1, c2).
```

The `HKDF` call is changed to produce 3 outputs, the last of which is termed `akm_root`. The resulting function is termed `ModifiedSplit()` and is specified here:

```
ModifiedSplit(): Returns a pair of CipherState objects for encrypting transport messages. Executes the following steps, where zerolen is a zero-length byte sequence:

    Sets temp_k1, temp_k2, ask_root = HKDF(ck, zerolen, 3).
    If HASHLEN is 64, then truncates temp_k1 and temp_k2 to 32 bytes.
    Creates two new CipherState objects c1 and c2.
    Calls c1.InitializeKey(temp_k1) and c2.InitializeKey(temp_k2).
    Returns (c1, c2, ask_root).
```

`ask_root` is raw key material and not a Noise CipherState object like `c1` and `c2`. The purpose of this notation is simply to make clear that `ask_root` is an additional shared key calculated at the conclusion of the handshake, derived in the same HKDF operation as the key material used to create `c1` and `c2`.

### 5.2.5 Ciphertext length obfuscation keys
The `ask_root` is used to initialize four variables used for length obfuscation.

```
AskRoot // ask_root from ModifiedSplit() as described above

keyRaw = HKDF(ikm=askRoot, len=2*FAST_HASH_LEN, salt=zeros(0), info="obfuscated-length-key")
ivRaw = HKDF(ikm=askRoot, len=16, salt=zeros(0), info="obfuscated-length-iv")

LenKeyA = keyRaw[0 ... FAST_HASH_LEN]
LenKeyB = keyRaw[FAST_HASH_LEN ... 2*FAST_HASH_LEN]

LenIvA = ivRaw[0 ... 8]
LenIvB = ivRaw[8 ... 16]
```

The `LenKeyA`, `LenKeyB`, `LenIvA` and `LenIvB` fields are used in the Ciphertext Length Obfuscation section below.

### 5.2.6 Payloads
The additional data sections `PayloadA` and `PayloadB` are constructed as follows.

The initiator performs the following operations to calculate `PayloadA` at the end of round 3:
```
// HA refers to Noise handshake hash after the "se" symbol of round 3 has been processed
// FSID is FSID belonging to the filesystem initiator intends to share with responder

idHash = HMAC(key=HA, data=FSID)
proofA = calculateProof(0, HA)

PayloadA = JSON({ idHash:idHash, proof:proofA }) || zeros(1)
```

The responder performs the following operations to calculate `PayloadB` at the end of round 4:
```
// HB refers to Noise handshake hash after the "psk" symbol of round 4 has been processed
if proofA == calculateProof(0, HA):
  proofB = calculateProof(1, HB)
else:
  proofB = RNG(SYM_KEY_LEN)
PayloadB = JSON({ proof:proofB }) || zeros(1)
```

The `calculateProof` function is used by both peers:

```
calculateProof(index, hash):
  // LocalHostHasRootKey true <=> the peer executing this function possesses the RootKey
  if(LocalHostHasRootKey == false):
    return RNG(SYM_KEY_LEN)
  return deriveSubkey(parentKey=RootKey, name="handshake-proof", salt=index(1) || hash)
```

The `proofA` and `proofB` fields to to allow the peers to supply proof that they possess knowledge of the passphrase root key. These proofs are calculated so that the verifier must also possess knowledge of the passphrase root key to determine if they are legitimate, and are bound by way of the handshake hash to the specific connection to prevent replay attacks.

The purpose of this is to allow support for higher-level protocol messages that require each peer to have demonstrated read access to the filesystem contents.

Either peer may conceal the fact that it possesses the `RootKey` by sending random bytes in place of a proof. If both peers possess the `RootKey` and the requestor admits to this by sending a valid proof, the responder may choose whether to also admit knowledge by sending its own proof, or to conceal its knowledge by sending a random proof. Peers will not grant access to privileged messages without a valid proof, and so this will limit the responder's behavior in the connection to the requestor to those actions that would be acceptable for a seed-only peer.