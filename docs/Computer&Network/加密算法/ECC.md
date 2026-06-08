# ECC非对称加解密算法

## 1. 算法概述

**ECC**（Elliptic Curve Cryptography，椭圆曲线密码学）是一类基于**椭圆曲线离散对数问题（ECDLP）**的公钥密码体制。它由 Neal Koblitz 和 Victor Miller 于 1985 年各自独立提出。

与 RSA 相比，ECC 在相同安全强度下使用**更短的密钥**，从而获得更好的性能和更小的存储开销。如今，ECC 已广泛应用于 TLS、SSH、加密货币等领域，是**新项目公钥密码学的首选方案**。

### 1.1 基本参数

| 参数 | 值 |
|------|-----|
| 算法类型 | 非对称加密/签名/密钥协商 |
| 常用密钥长度 | 256 / 384 / 521 位 |
| 数学基础 | 椭圆曲线离散对数问题 |
| 主要标准 | NIST P-256, Curve25519, secp256k1 |
| 安全状态 | ✅ **当前推荐，性能优于 RSA** |

### 1.2 ECC 与 RSA 密钥长度对比

| ECC 密钥长度 | RSA 等效密钥长度 | 对称加密等效 | 密钥大小比 |
|-------------|----------------|-------------|-----------|
| 256 位 | 3072 位 | 128 位 | RSA 的 1/12 |
| 384 位 | 7680 位 | 192 位 | RSA 的 1/20 |
| 521 位 | 15360 位 | 256 位 | RSA 的 1/30 |

**这意味着**：ECC-256 只需 32 字节的密钥就能达到 RSA-3072（384 字节）的安全强度。在带宽受限、存储受限的场景（移动端、IoT、区块链）中优势明显。

## 2. 算法原理（简述）

### 2.1 核心思想

ECC 的安全性基于椭圆曲线上的**离散对数问题**：

```
已知：基点 G、公钥 Q = k × G（k 次点加法）
求解：私钥 k = ?

"正向计算"（已知 k 和 G 求 Q）很容易
"反向求解"（已知 G 和 Q 求 k）极其困难
```

椭圆曲线定义在有限域上：`y² ≡ x³ + ax + b (mod p)`，曲线上的点构成一个群，支持"点加法"运算。

### 2.2 密钥对

- **私钥**：一个随机大整数 k
- **公钥**：椭圆曲线上的一个点 Q = k × G（G 为标准基点）

### 2.3 三大应用方案

| 方案 | 用途 | 说明 |
|------|------|------|
| **ECDH** | 密钥协商 | 双方交换公钥，各自计算出相同的共享密钥 |
| **ECDSA** | 数字签名 | 椭圆曲线版的 DSA 签名算法 |
| **EdDSA** | 数字签名 | 基于 Edwards 曲线的确定性签名（Ed25519） |

> 开发者不需要理解椭圆曲线的数学细节（点加法、标量乘法的具体实现），只需知道选择哪条曲线、调用哪个 API。

## 3. 曲线选择指南（开发者重点）

### 3.1 主流曲线对比

| 曲线名称 | 位数 | 推荐场景 | 说明 |
|----------|------|----------|------|
| **Ed25519** | 255 | 数字签名（首选） | 快速、确定性、抗侧信道 |
| **X25519** | 255 | 密钥协商（首选） | 高性能、设计安全 |
| **NIST P-256** | 256 | 政府/合规场景 | FIPS 认证，TLS 广泛使用 |
| **NIST P-384** | 384 | 高安全需求 | 192 位安全强度 |
| **secp256k1** | 256 | 区块链 | 比特币、以太坊标准 |
| **Ed448** | 448 | 超高安全需求 | 224 位安全强度 |

### 3.2 选型决策

```
新项目签名 → Ed25519
新项目密钥协商 → X25519
需要 FIPS 合规 → P-256
区块链相关 → secp256k1
需要 192+ 位安全 → P-384 或 Ed448
```

### 3.3 为什么推荐 Curve25519 系列？

| 优势 | 说明 |
|------|------|
| 常数时间实现 | 设计上避免时序攻击 |
| 无需随机数（EdDSA） | 避免 k 值重用导致私钥泄露 |
| 性能优异 | 签名/验签速度快于 P-256 |
| 密钥简短 | 公钥 32 字节，签名 64 字节 |
| 安全曲线参数 | 参数选择透明，无后门疑虑 |

## 4. 安全性分析

### 4.1 当前安全状态

对于 256 位曲线，最优攻击复杂度约为 2¹²⁸，当前完全不可行。

### 4.2 实现中的致命陷阱

| 陷阱 | 后果 | 防护 |
|------|------|------|
| **k 值重用** | 私钥直接泄露 | 使用 EdDSA（确定性）或 RFC 6979 |
| 无效曲线攻击 | 私钥泄露 | 验证点在曲线上（标准库已处理） |
| 侧信道泄露 | 私钥泄露 | 使用 Curve25519（常数时间设计） |
| 弱随机数 | 私钥泄露 | 使用 `crypto/rand` |

> **🚨 k 值重用的致命后果**：在 ECDSA 中，如果两次签名使用了相同的随机数 k，攻击者可以直接计算出私钥。2010 年索尼 PS3 被破解正是因为固定 k 值。这是选择 EdDSA（确定性签名，不需要随机 k）的重要原因。

### 4.3 量子计算威胁

与 RSA 相同，ECC 也会被 Shor 算法破解。由于 ECC 密钥更短，理论上比 RSA 更容易受到量子攻击。后量子时代需迁移到格基密码等方案（NIST ML-KEM / ML-DSA）。

## 5. 开发者实践指南

### 5.1 使用要点

| 要点 | 说明 |
|------|------|
| 签名首选 Ed25519 | 确定性签名，无 k 值重用风险 |
| 密钥协商首选 X25519 | 常数时间实现，抗侧信道 |
| 不要自选曲线参数 | 使用标准曲线，不要自定义 a、b、p |
| 验证对方公钥 | 确保收到的公钥在曲线上（标准库通常已处理） |
| ECDH 后要 KDF | 共享密钥不能直接用作对称密钥，需经 KDF 派生 |

### 5.2 ECDH + AES-GCM 典型流程

```
密钥协商后加密通信：
1. Alice 和 Bob 各生成 X25519 密钥对
2. 交换公钥
3. 各自计算 ECDH 共享密钥 S
4. 通过 HKDF 从 S 派生 AES-256 密钥 K
5. 使用 AES-256-GCM + K 加密通信数据
```

### 5.3 各语言使用方式

| 语言 | Ed25519 签名 | X25519 密钥协商 |
|------|-------------|----------------|
| Go | `crypto/ed25519` | `crypto/ecdh` |
| Java | `EdDSA` (JDK 15+) | `XDH` (JDK 11+) |
| Python | `cryptography` 库 | `cryptography` 库 |
| Node.js | `crypto.sign('ed25519', ...)` | `crypto.diffieHellman(...)` |
| Rust | `ed25519-dalek` | `x25519-dalek` |

## 6. Go 语言实现

### 6.1 Ed25519 签名/验签（推荐）

```go
package main

import (
	"crypto/ed25519"
	"crypto/rand"
	"encoding/hex"
	"fmt"
)

func main() {
	fmt.Println("=== Ed25519 签名/验签 ===")

	// 生成密钥对
	publicKey, privateKey, err := ed25519.GenerateKey(rand.Reader)
	if err != nil {
		panic(err)
	}

	fmt.Printf("公钥(hex): %s\n", hex.EncodeToString(publicKey))
	fmt.Printf("公钥长度: %d 字节\n", len(publicKey))

	// 签名（确定性，无需随机数）
	message := []byte("Hello, Ed25519 Signature!")
	signature := ed25519.Sign(privateKey, message)

	fmt.Printf("消息: %s\n", message)
	fmt.Printf("签名长度: %d 字节\n", len(signature))

	// 验签
	valid := ed25519.Verify(publicKey, message, signature)
	fmt.Printf("验签结果: %v\n", valid)

	// 篡改消息后验签
	tampered := []byte("Hello, Ed25519 Signature?")
	valid = ed25519.Verify(publicKey, tampered, signature)
	fmt.Printf("篡改后验签: %v (预期 false)\n", valid)
}
```

### 6.2 ECDH 密钥协商

```go
package main

import (
	"crypto/ecdh"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"fmt"
)

func main() {
	fmt.Println("=== X25519 ECDH 密钥协商 ===")

	// Alice 生成密钥对
	alicePrivate, err := ecdh.X25519().GenerateKey(rand.Reader)
	if err != nil {
		panic(err)
	}
	alicePublic := alicePrivate.PublicKey()

	// Bob 生成密钥对
	bobPrivate, err := ecdh.X25519().GenerateKey(rand.Reader)
	if err != nil {
		panic(err)
	}
	bobPublic := bobPrivate.PublicKey()

	// 双方计算共享密钥
	aliceShared, err := alicePrivate.ECDH(bobPublic)
	if err != nil {
		panic(err)
	}

	bobShared, err := bobPrivate.ECDH(alicePublic)
	if err != nil {
		panic(err)
	}

	fmt.Printf("Alice 共享密钥: %s\n", hex.EncodeToString(aliceShared[:16])+"...")
	fmt.Printf("Bob   共享密钥: %s\n", hex.EncodeToString(bobShared[:16])+"...")
	fmt.Printf("密钥一致: %v\n", hex.EncodeToString(aliceShared) == hex.EncodeToString(bobShared))

	// 实际使用时，应通过 KDF 派生对称密钥
	// 这里简单用 SHA-256 演示
	aesKey := sha256.Sum256(aliceShared)
	fmt.Printf("派生 AES 密钥: %s\n", hex.EncodeToString(aesKey[:16])+"...")
}
```

### 6.3 ECDSA 签名（合规场景）

```go
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"fmt"
)

func main() {
	fmt.Println("=== ECDSA P-256 签名/验签 ===")

	// 生成密钥对（P-256 曲线）
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		panic(err)
	}
	publicKey := &privateKey.PublicKey

	// 签名
	message := []byte("Hello, ECDSA Signature!")
	hash := sha256.Sum256(message)

	signature, err := ecdsa.SignASN1(rand.Reader, privateKey, hash[:])
	if err != nil {
		panic(err)
	}

	fmt.Printf("消息: %s\n", message)
	fmt.Printf("签名长度: %d 字节\n", len(signature))

	// 验签
	valid := ecdsa.VerifyASN1(publicKey, hash[:], signature)
	fmt.Printf("验签结果: %v\n", valid)
}
```

## 7. 实际应用场景

| 场景 | 使用的 ECC 方案 | 说明 |
|------|----------------|------|
| TLS 1.3 | X25519 + Ed25519/ECDSA | 密钥协商 + 证书签名 |
| SSH 密钥 | Ed25519 | `ssh-keygen -t ed25519` |
| 比特币/以太坊 | secp256k1 + ECDSA | 交易签名 |
| Signal/WhatsApp | X25519 + Ed25519 | 端到端加密 |
| WireGuard VPN | Curve25519 | 密钥协商 |
| FIDO2/WebAuthn | P-256 ECDSA | 无密码认证 |
| iOS/macOS 签名 | P-256 ECDSA | 应用签名 |

## 8. 总结

| 维度 | 评价 |
|------|------|
| 安全性 | ⭐⭐⭐⭐⭐ 256 位即达 128 位安全强度 |
| 性能 | ⭐⭐⭐⭐⭐ 远优于同等安全级别的 RSA |
| 密钥大小 | ⭐⭐⭐⭐⭐ 极为紧凑（公钥 32 字节） |
| 生态 | ⭐⭐⭐⭐ 主流语言和协议全面支持 |
| 未来性 | ⭐⭐ 受量子计算威胁（与 RSA 相同） |

**开发者决策指南**：

- **数字签名** → Ed25519（首选）或 ECDSA P-256（合规需求）
- **密钥协商** → X25519（首选）或 ECDH P-256（合规需求）
- **区块链** → secp256k1
- **需要加密** → ECDH 协商密钥 + AES-256-GCM 加密数据
- **RSA 迁移** → Ed25519 替代 RSA 签名，X25519+AES 替代 RSA 加密
