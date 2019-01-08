# 2. Conventions

## 2.1. Operators

### 2.1.1 Concatenation operator

The `||` operator denotes concatenation. Example:

`"abc" || "def"   // "abcdef"`

When the `||` operator is applied to an integer, the integer is to be serialized in big-endian byte order. The size of the integer field in bits is explicitly specified in all cases. For example:

```
x = 1234
y = "abc" || x(32)   // y = { 'a', 'b', 'c', 0x00, 0x00, 0x12, 0x34 }
```

### 2.1.2 Size operator

The operation `|x|` on a byte string `x` returns the number of bytes in `x`. Example:

```
x = "abc"    // |x| = 3
```

### 2.1.3 Range operator

Given a byte array `V`, the operation `V[a ... b]` produces the subarray beginning with `V[a]`, and continuing to (but not including) `V[b]`. It follows that `|V[a ... b]|` is equal to `b - a` bytes.

### 2.1.4 Bitshift operator

The operation `x << y` on a bit string `x` causes `x` to be shifted left by `y` bits. The `y` least significant bits of the result are set to 0.

### 2.1.5 XOR operator

The operation `x ^ y` is understood to be equivalent to `x XOR y`.
