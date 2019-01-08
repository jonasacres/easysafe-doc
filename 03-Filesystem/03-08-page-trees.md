# 3. Filesystem
## 3.8 Page trees
Individual pages can only be read if their page tag is known. When a file consists of only a single tag, the tag is stored directly in the inode table for the file. For files containing multiple tags, the tags are stored in a B-tree structure known as a "page tree". Each node of the page tree is known as a "page tree chunk."

### 3.8.1 Page tree chunk plaintext

Each chunk is a list of tags. For branch nodes, these tags reflect the tags of the node's children. For leaf nodes, these tags reflect page tags belonging to the file.

| Chunk |
|---|
| tag0 |
| tag1 |
| ... |

Each chunk has as many tags as can be fit into `PAGE_SIZE` bytes. If the filesystem `PAGE_SIZE` is not a multiple of `HASH_LEN`, then the chunk is padded during encryption so that the resulting ciphertexts match the lengths of ordinary pages.

If a chunk does not have enough child tags to fill `PAGE_SIZE` bytes, then each missing tag is set to `zeros(HASH_LEN)`.

### 3.8.2 Page tree chunk ciphertext

```
FSID // From filesystem
Distinguisher // From inode
ChunkNum // Index of chunk in tree
ChunkPlaintext // Constructed per table above

chunkKey = deriveSubkey(parentKey=FSKey, name="Chunk", salt=FSID || Distinguisher(64) || ChunkNum(16))
chunkTag, PageCiphertext = taggedEncrypt(symKey=chunkKey, authKey=SeedRoot, signPrivateKey=WritePrivateKey, plaintext=ChunkPlaintext)
```

### 3.8.3 Storage
The chunk is stored at a location determined by its tag.

```
ChunkPath = hashpath(ChunkTag)
```

### 3.8.4 RefTag calculation
When page trees are used, the file reftag is constructed with type `REF_TYPE_2INDIRECT` using the tag of the root chunk (that is, the chunk with index 0).
