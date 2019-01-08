# 3. Filesystem
## 3.5 Inode table

EasySafe's filesystem relies on an inode table to locate file data. Each file in the filesystem is indexed by an inode within the table. This inode contains basic information beyond the scope of this document, similar to that described in a POSIX `stat_t` structure. Additionally, the inode contains a 64-bit randomly-generated `distinguisher` field which is constant for the life of the file, and a RefTag.

Inode contents include:

| Field | Description |
|-------|-------------|
| stat  | POSIX `stat_t`-like data structure containing file metadata
| distinguisher | Random 64-bit integer; constant for lifetime of file
| reftag | RefTag object, described below. Allows retrieval of file.

The inode table is stored in the same manner as any other file in the filesystem, described below. The inode table has a fixed `distinguisher` of zero. The `reftag` of the inode table is expressed as a `RevisionTag`, described below.
