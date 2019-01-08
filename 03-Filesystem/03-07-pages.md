# 3. Filesystem
## 3.7 Pages

### 3.7.1 Plaintext structure
Files are divded into individual pages of fixed length (`PAGE_SIZE`) for storage. The final page of a file may be shorter than the `PAGE_SIZE`. The final page is padded during encryption so that all encrypted pages have the same length.

### 3.7.2 Ciphertext
The plaintext is encrypted, resulting in a ciphertext and page tag.

```
FSID // From filesystem
Distinguisher // From inode
PageNum // Index of page into file
PagePlaintext // Constructed per table above

pageKey = deriveSubkey(parentKey=FSKey, name="Page", salt=FSID || Distinguisher(64) || PageNum(16))
PageTag, PageCiphertext = taggedEncrypt(symKey=pageKey, authKey=SeedRoot, signPrivateKey=WritePrivateKey, plaintext=PagePlaintext)
```

### 3.7.3 Storage
The page is stored at a location determined by its tag.

```
PagePath = hashpath(PageTag)
```

### 3.7.4 RefTag calculation

If a file consists of exactly one page, then the resulting tag of that page is used to create the RefTag stored in the file's inode, with type `REF_TYPE_INDIRECT`. Otherwise, the page tag is stored inside of a page tree, defined in the following section.
