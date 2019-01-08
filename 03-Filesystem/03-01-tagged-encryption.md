# 2. Filesystem
## 2.1 Tagged encryption

The following methodology is used to perform the `taggedEncrypt` function, used several times in this document. This function is used as the basis for all file encryption in EasySafe.

### 2.1.2 Arguments

The `taggedEncrypt` function has the following arguments:
```
taggedEncrypt(symRootKey, authKey, signPrivateKey, plaintext)
```

| Argument | Description
|--|--|
| SymRootKey | Key material used to derive encryption key
| KeySalt | Additional salt to use in encryption key derivation
| AuthKey | Key used to authenticate resultant ciphertext
| SignPrivateKey | Private key used to sign resultant ciphertext
| Plaintext | Plaintext to be encrypted

### 2.1.3 Result

The `taggedEncrypt` function produces a two-tuple of the form (tag, ciphertext):

| Result | Description
|-|-|
| Tag | Authenticated hash of ciphertext using supplied `AuthKey` |
| SignedCiphertext | Signed ciphertext, encrypted using material from `SymRootKey` and signed with `SignPrivateKey`

### 2.1.4 Methodology

#### Plaintext padding
The plaintext is first padded to the fixed page size.

```
PaddedPlaintext = pad(data=Plaintext, len=PAGE_SIZE) // must have: |Plaintext| <= PAGE_SIZE
```

#### Key derivation
File-specific keys are derived based on the

```
keyMaterial = HDKF(ikm=KeySalt, length=2*SYM_KEY_LENGTH, salt=symRootKey, info="tagged-encryption-key-material")
TextKey = keyMaterial[0 ... SYM_KEY_LENGTH]
SaltKey = keyMaterial[SYM_KEY_LENGTH ... 2*SYM_KEY_LENGTH]
```

#### File salt
A plaintext-specific salt is derived using the `SaltKey`:

```
PTSalt = HMAC(key=SaltKey, data=PaddedPlaintext)
```

#### Encryption key derivation
The `PTSalt` is used to derive an encryption key:

```
EncryptionKey = deriveSubkey(parentKey=symRootKey, name="tagged-encryption-key", salt=PTSalt)
```

#### Unsigned ciphertext
The encryption key is used to produce unauthenticated ciphertext:

```
RawCiphertext = SymmetricEncrypt(key=EncryptionKey, nonce=zeros(SYM_IV_LEN), plaintext=PaddedPlaintext)
```

#### Signature
The raw ciphertext is signed and combined with the `PTSalt` to produce the final ciphertext:

```
signature = AsymmetricSign(key=SignPrivateKey, data=RawCiphertext)
SignedCiphertext = PTSalt || RawCiphertext || signature
```

#### Tag
The ciphertext is authenticated using the auth key.

```
Tag = HMAC(key=authKey, data=Ciphertext)
```
