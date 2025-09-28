# Encoding & Hashing Size Comparison

---

## 📌 Common Hash Algorithms and Output Sizes

| Algorithm | Output Size (bits) | Output Size (bytes) | Hex Length (chars) | Base64 Length (chars) |
|-----------|---------------------|----------------------|---------------------|------------------------|
| **MD5** | 128 bits | 16 bytes | 32 | 24 |
| **SHA-1** | 160 bits | 20 bytes | 40 | 28 |
| **SHA-256** | 256 bits | 32 bytes | 64 | 44 |
| **SHA-384** | 384 bits | 48 bytes | 96 | 64 |
| **SHA-512** | 512 bits | 64 bytes | 128 | 88 |

👉 Notes:  
- Hex length = `2 × bytes`.  
- Base64 length = `4 × ceil(bytes / 3)`.  

---

## 📌 Encoding Size Formulas

Let **N = number of input bytes**.

| Representation | Formula (characters) | Encoded Size (bytes) | Notes |
|----------------|-----------------------|-----------------------|-------|
| **Raw** | — | `N` | Pure binary form |
| **Hex** | `2 × N` | `2 × N` | Each byte → 2 hex chars |
| **Base64** | `4 × ceil(N / 3)` | `4 × ceil(N / 3)` | 3 bytes → 4 chars |
| **Base32** | `8 × ceil(N / 5)` | `8 × ceil(N / 5)` | 5 bytes → 8 chars |
| **Base58** | `≈ ceil(N × log(256) / log(58))` | ≈ `1.37 × N` | Compact; excludes confusing chars (0, O, I, l, etc.) |
| **Base85 (Ascii85/Z85)** | `5 × ceil(N / 4)` | `5 × ceil(N / 4)` | 4 bytes → 5 chars (~25% overhead) |

---

## 📌 Example: MD5 Hash (16 bytes = 128 bits)

| Representation | Formula Applied | Characters | Encoded Size |
|----------------|-----------------|------------|--------------|
| **Raw** | `N = 16` | — | 16 bytes |
| **Hex** | `2 × 16 = 32` | 32 | 32 bytes |
| **Base64** | `4 × ceil(16 / 3) = 24` | 24 | 24 bytes |
| **Base32** | `8 × ceil(16 / 5) = 32` | 32 | 32 bytes |
| **Base58** | `ceil(16 × log₂(256) / log₂(58)) = ceil(128 / 5.857) ≈ 22` | 22 | 22 bytes |
| **Base85** | `5 × ceil(16 / 4) = 20` | 20 | 20 bytes |

---

## 📌 Compactness Ranking (MD5 = 16 bytes)

1. **Base58** → 22 bytes (smallest ASCII-safe form)  
2. **Base85** → 20 bytes  
3. **Base64** → 24 bytes  
4. **Hex / Base32** → 32 bytes (largest)  
