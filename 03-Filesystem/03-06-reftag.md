# 3. Filesystem
## 3.6 RefTag

A RefTag is an identifier allowing retrieval of a file from a filesystem. It is structured as follows:

| Field | Size | Description
|-------|--------|----|
| tag   | HASH_LEN | Context-dependent on refType
| numPages | 8 | Number of pages in file. Does not include page tree chunks.
| refType | 1 | Reference type, determines meaning of `tag` field. See below for acceptable values.
| reserved | 3 | Reserved for future use.

The `refType` field takes on values from the following table:

| Value | Symbol | tag field semantics | Used when...
|-------|--------|---------------------|----|
| 0     | REF_TYPE_IMMEDIATE | Tag field contains literal file contents, zero-padded to HASH_LEN | File size < HASH_LEN
| 1     | REF_TYPE_INDIRECT | Tag field contains tag of only page in file. | HashLen <= File size <= PAGE_SIZE
| 2     | REF_TYPE_2INDIRECT | Tag field contains tag of root page tree chunk. | File size > PAGE_SIZE

The "Page" and "Page Tree" structures are described in the following sections.
