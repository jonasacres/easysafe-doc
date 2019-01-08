# 2. Conventions
## 2.3 Basic constants
The following constants will be referred to by name throughout the document:

| Constant name    | Value | Unit | Description
|------------------|-------|------|----|
| HASH_LEN         | 64              | bytes | Output length of Hash() function
| SYM_KEY_LEN      | 32              | bytes | Symmetric key size
| SYM_IV_LEN       | 16              | bytes | Symmetric initialization vector size in unauthenticated mode (equivalent to block size)
| SYM_AEAD_IV_LEN  | 16              | bytes | Symmetric initialization vector size in authenticated mode
| SYM_AEAD_TAG_LEN | 16              | bytes | Tag size for symmetric encryption in authenticated mode
| ASYM_SIG_LEN     | 64              | bytes | Output length of AsymmetricSign() function
| PAGE_SIZE        | 65536 (default) | bytes | Configurable fixed page size for filesystem.
