# ECC非对称加解密算法

## 1. 算法概述

**ECC**（Elliptic Curve Cryptography，椭圆曲线密码学）是一类基于**椭圆曲线离散对数问题（ECDLP）**的公钥密码体制。它由 Neal Koblitz 和 Victor Miller 于 1985 年各自独立提出。

与 RSA 相比，ECC 在相同安全强度下使用更短的密钥，从而获得更好的性能和更小的存储开销。如今，ECC 已广泛应用于 TLS、SSH、加密货币等领域。

### 1.1 基本参数

| 参数 | 值 |
|------|-----|
| 算法类型 | 非对称加密/签名/密钥协商 |
| 常用密钥长度 | 256 / 384 / 521 位 |
| 数学基础 | 椭圆曲线离散对数问题 |
| 核心操作 | 椭圆曲线上的点乘法 |
| 主要标准 | NIST P-256/P-384/P-521, Curve25519, secp256k1 |

### 1.2 ECC 与 RSA 密钥长度对比

| ECC 密钥长度 | RSA 等效密钥长度 | 对称加密等效 |
|-------------|----------------|-------------|
| 160 位 | 1024 位 | 80 位 |
| 224 位 | 2048 位 | 112 位 |
| 256 位 | 3072 位 | 128 位 |
| 384 位 | 7680 位 | 192 位 |
| 521 位 | 15360 位 | 256 位 |

ECC 256 位 ≈ RSA 3072 位的安全强度，但密钥仅为 RSA 的 1/12。

## 2. 数学基础

### 2.1 椭圆曲线定义

在有限域 GF(p) 上，椭圆曲线定义为满足以下方程的点集加上无穷远点 O：

```
y² ≡ x³ + ax + b (mod p)

其中：4a³ + 27b² ≠ 0 (mod p)（确保曲线无奇点）
```

### 2.2 椭圆曲线上的点运算

#### 点加法（P + Q）

给定曲线上两个不同点 P(x₁, y₁) 和 Q(x₂, y₂)：

```
斜率 λ = (y₂ - y₁) / (x₂ - x₁) mod p

R = P + Q:
  x₃ = λ² - x₁ - x₂ mod p
  y₃ = λ(x₁ - x₃) - y₁ mod p
```

#### 点倍乘（P + P = 2P）

```
斜率 λ = (3x₁² + a) / (2y₁) mod p

R = 2P:
  x₃ = λ² - 2x₁ mod p
  y₃ = λ(x₁ - x₃) - y₁ mod p
```

#### 标量乘法（k × P）

将点 P 与自身相加 k 次：`Q = k × P = P + P + ... + P (k 次)`

- 已知 k 和 P，计算 Q 很容易（使用"倍加-加"算法）
- 已知 P 和 Q，求 k **极其困难** → 这就是 ECDLP

### 2.3 椭圆曲线离散对数问题（ECDLP）

```
已知：基点 G、公钥 Q = k × G
求解：私钥 k = ?

对于适当大小的曲线（如 256 位），目前无已知的多项式时间算法可以求解 ECDLP。
```

### 2.4 常用椭圆曲线

| 曲线名称 | 位数 | 标准 | 应用场景 |
|----------|------|------|----------|
| NIST P-256 (secp256r1) | 256 | NIST/FIPS | TLS, 政府应用 |
| NIST P-384 (secp384r1) | 384 | NIST/FIPS | 高安全需求 |
| NIST P-521 (secp521r1) | 521 | NIST/FIPS | 极高安全需求 |
| Curve25519 | 255 | Bernstein | SSH, Signal, WireGuard |
| Curve448 | 448 | Goldilocks | 需要更高安全性 |
| secp256k1 | 256 | SECG | 比特币、以太坊 |
| Ed25519 | 255 | Bernstein | 数字签名 |

## 3. ECC 密码方案

### 3.1 ECDH（椭圆曲线 Diffie-Hellman 密钥协商）

```
Alice:
  私钥: a（随机整数）
  公钥: A = a × G

Bob:
  私钥: b（随机整数）
  公钥: B = b × G

密钥协商：
  Alice 计算: S = a × B = a × b × G
  Bob 计算:   S = b × A = b × a × G
  
共享密钥: S（双方计算结果相同）
```

### 3.2 ECDSA（椭圆曲线数字签名算法）

#### 签名过程

```
输入: 消息哈希 e, 私钥 d, 曲线参数 (G, n)

1. 选择随机数 k ∈ [1, n-1]
2. 计算 (x₁, y₁) = k × G
3. 计算 r = x₁ mod n（若 r = 0 则重新选 k）
4. 计算 s = k⁻¹(e + r·d) mod n（若 s = 0 则重新选 k）
5. 签名为 (r, s)
```

#### 验签过程

```
输入: 消息哈希 e, 签名 (r, s), 公钥 Q

1. 计算 u₁ = e·s⁻¹ mod n
2. 计算 u₂ = r·s⁻¹ mod n
3. 计算 (x₁, y₁) = u₁×G + u₂×Q
4. 验证: r ≡ x₁ (mod n) 则签名有效
```

### 3.3 EdDSA（Edwards 曲线数字签名算法）

EdDSA 是一种确定性签名方案，基于 Twisted Edwards 曲线：

- **Ed25519**：基于 Curve25519，用于签名
- **Ed448**：基于 Curve448，更高安全性

特点：
- **确定性**：不需要随机数（避免 k 值重用漏洞）
- **快速**：签名和验签性能极高
- **抗侧信道**：设计上避免了许多实现陷阱

### 3.4 ECIES（椭圆曲线集成加密方案）

用于加密的混合方案：

```
加密：
1. 生成临时密钥对: (k, R = k×G)
2. 计算共享秘密: S = k × Q_受方公钥
3. 通过 KDF 派生对称密钥: (K_enc, K_mac) = KDF(S)
4. 对称加密: C = AES_K_enc(M)
5. 计算 MAC: T = MAC_K_mac(C)
6. 输出: (R, C, T)

解密：
1. 计算共享秘密: S = d × R（d 为私钥）
2. 派生密钥: (K_enc, K_mac) = KDF(S)
3. 验证 MAC
4. 对称解密: M = AES_K_enc⁻¹(C)
```

## 4. 安全性分析

### 4.1 ECDLP 的攻击复杂度

| 攻击方法 | 复杂度 | 说明 |
|----------|--------|------|
| Baby-step Giant-step | O(√n) | 通用方法 |
| Pollard's rho | O(√n) | 最优通用攻击 |
| MOV 攻击 | 依赖嵌入度 | 将 ECDLP 转化为有限域 DLP |
| 特殊曲线攻击 | 多项式时间 | 仅对特定弱曲线有效 |

对于 256 位曲线，最优攻击复杂度约为 2¹²⁸，目前不可行。

### 4.2 实现中的安全隐患

| 隐患 | 后果 | 防护 |
|------|------|------|
| k 值重用 | 私钥泄露 | 使用确定性签名（RFC 6979）或 EdDSA |
| 无效曲线攻击 | 私钥泄露 | 验证点在曲线上 |
| 侧信道泄露 | 私钥泄露 | 常数时间实现 |
| 弱曲线选择 | 安全性降低 | 使用标准曲线 |

> **🚨 k 值重用的致命后果**
>
> 在 ECDSA 中，如果两次签名使用了相同的 k 值，攻击者可以直接计算出私钥。2010 年索尼 PS3 被破解正是因为固定 k 值。

### 4.3 量子计算威胁

与 RSA 类似，ECC 也会被 Shor 算法破解。由于 ECC 密钥更短，理论上比 RSA 更容易受到量子攻击。后量子时代需迁移到格基密码等方案。

## 5. 代码实现

### 5.1 Go 语言实现

```go
package main

import (
	"crypto/ecdh"
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"encoding/hex"
	"fmt"
	"math/big"
)

// ===== ECDSA 签名/验签 =====

func ecdsaDemo() {
	fmt.Println("=== ECDSA 签名/验签 ===")

	// 生成密钥对（P-256 曲线）
	privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		panic(err)
	}
	publicKey := &privateKey.PublicKey

	// 签名
	message := []byte("Hello, ECDSA Signature!")
	hash := sha256.Sum256(message)

	r, s, err := ecdsa.Sign(rand.Reader, privateKey, hash[:])
	if err != nil {
		panic(err)
	}

	fmt.Printf("消息: %s\n", message)
	fmt.Printf("签名 r: %s\n", r.Text(16)[:32]+"...")
	fmt.Printf("签名 s: %s\n", s.Text(16)[:32]+"...")

	// 验签
	valid := ecdsa.Verify(publicKey, hash[:], r, s)
	fmt.Printf("验签结果: %v\n", valid)
}

// ===== ECDH 密钥协商 =====

func ecdhDemo() {
	fmt.Println("\n=== ECDH 密钥协商 ===")

	// Alice 生成密钥对
	alicePrivate, err := ecdh.P256().GenerateKey(rand.Reader)
	if err != nil {
		panic(err)
	}
	alicePublic := alicePrivate.PublicKey()

	// Bob 生成密钥对
	bobPrivate, err := ecdh.P256().GenerateKey(rand.Reader)
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
}

func main() {
	ecdsaDemo()
	ecdhDemo()
}
```

### 5.2 Go 语言实现（Ed25519）

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

	// 签名
	message := []byte("Hello, Ed25519 Signature!")
	signature := ed25519.Sign(privateKey, message)

	fmt.Printf("消息: %s\n", message)
	fmt.Printf("签名(hex): %s\n", hex.EncodeToString(signature))
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

### 5.3 Python 实现

```python
from cryptography.hazmat.primitives.asymmetric import ec, ed25519
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.kdf.hkdf import HKDF

# ===== ECDSA 签名/验签 =====
def ecdsa_demo():
    print("=== ECDSA 签名/验签 ===")

    # 生成密钥对（P-256 曲线）
    private_key = ec.generate_private_key(ec.SECP256R1())
    public_key = private_key.public_key()

    # 签名
    message = b"Hello, ECDSA Signature!"
    signature = private_key.sign(message, ec.ECDSA(hashes.SHA256()))

    print(f"消息: {message.decode()}")
    print(f"签名长度: {len(signature)} 字节")
    print(f"签名(hex): {signature.hex()[:64]}...")

    # 验签
    try:
        public_key.verify(signature, message, ec.ECDSA(hashes.SHA256()))
        print("验签结果: 成功 ✓")
    except Exception:
        print("验签结果: 失败 ✗")

# ===== ECDH 密钥协商 =====
def ecdh_demo():
    print("\n=== ECDH 密钥协商 ===")

    # Alice
    alice_private = ec.generate_private_key(ec.SECP256R1())
    alice_public = alice_private.public_key()

    # Bob
    bob_private = ec.generate_private_key(ec.SECP256R1())
    bob_public = bob_private.public_key()

    # 计算共享密钥
    alice_shared = alice_private.exchange(ec.ECDH(), bob_public)
    bob_shared = bob_private.exchange(ec.ECDH(), alice_public)

    # 使用 HKDF 派生最终密钥
    alice_key = HKDF(
        algorithm=hashes.SHA256(), length=32,
        salt=None, info=b"ecdh derived key"
    ).derive(alice_shared)

    bob_key = HKDF(
        algorithm=hashes.SHA256(), length=32,
        salt=None, info=b"ecdh derived key"
    ).derive(bob_shared)

    print(f"Alice 派生密钥: {alice_key.hex()[:32]}...")
    print(f"Bob   派生密钥: {bob_key.hex()[:32]}...")
    print(f"密钥一致: {alice_key == bob_key}")

# ===== Ed25519 签名 =====
def ed25519_demo():
    print("\n=== Ed25519 签名/验签 ===")

    private_key = ed25519.Ed25519PrivateKey.generate()
    public_key = private_key.public_key()

    message = b"Hello, Ed25519 Signature!"
    signature = private_key.sign(message)

    print(f"消息: {message.decode()}")
    print(f"签名长度: {len(signature)} 字节")

    try:
        public_key.verify(signature, message)
        print("验签结果: 成功 ✓")
    except Exception:
        print("验签结果: 失败 ✗")

if __name__ == "__main__":
    ecdsa_demo()
    ecdh_demo()
    ed25519_demo()
```

### 5.4 Java 实现

```java
import java.security.*;
import java.security.spec.*;
import javax.crypto.KeyAgreement;
import java.util.HexFormat;

public class ECCExample {

    // ECDSA 签名
    public static byte[] sign(PrivateKey privateKey, byte[] message) throws Exception {
        Signature sig = Signature.getInstance("SHA256withECDSA");
        sig.initSign(privateKey);
        sig.update(message);
        return sig.sign();
    }

    // ECDSA 验签
    public static boolean verify(PublicKey publicKey, byte[] message, byte[] signature) 
            throws Exception {
        Signature sig = Signature.getInstance("SHA256withECDSA");
        sig.initVerify(publicKey);
        sig.update(message);
        return sig.verify(signature);
    }

    // ECDH 密钥协商
    public static byte[] performECDH(PrivateKey privateKey, PublicKey publicKey) throws Exception {
        KeyAgreement keyAgreement = KeyAgreement.getInstance("ECDH");
        keyAgreement.init(privateKey);
        keyAgreement.doPhase(publicKey, true);
        return keyAgreement.generateSecret();
    }

    public static void main(String[] args) throws Exception {
        // 生成 EC 密钥对（P-256）
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("EC");
        keyPairGen.initialize(new ECGenParameterSpec("secp256r1"));

        // === ECDSA ===
        System.out.println("=== ECDSA 签名/验签 ===");
        KeyPair keyPair = keyPairGen.generateKeyPair();

        byte[] message = "Hello, ECDSA Signature!".getBytes();
        byte[] signature = sign(keyPair.getPrivate(), message);

        System.out.println("消息: " + new String(message));
        System.out.println("签名长度: " + signature.length + " 字节");

        boolean valid = verify(keyPair.getPublic(), message, signature);
        System.out.println("验签结果: " + (valid ? "成功 ✓" : "失败 ✗"));

        // === ECDH ===
        System.out.println("\n=== ECDH 密钥协商 ===");
        KeyPair aliceKeyPair = keyPairGen.generateKeyPair();
        KeyPair bobKeyPair = keyPairGen.generateKeyPair();

        byte[] aliceShared = performECDH(aliceKeyPair.getPrivate(), bobKeyPair.getPublic());
        byte[] bobShared = performECDH(bobKeyPair.getPrivate(), aliceKeyPair.getPublic());

        System.out.println("Alice 共享密钥: " + 
            HexFormat.of().formatHex(aliceShared).substring(0, 32) + "...");
        System.out.println("Bob   共享密钥: " + 
            HexFormat.of().formatHex(bobShared).substring(0, 32) + "...");
        System.out.println("密钥一致: " + 
            HexFormat.of().formatHex(aliceShared).equals(HexFormat.of().formatHex(bobShared)));
    }
}
```

## 6. 曲线选择指南

| 需求 | 推荐曲线 | 原因 |
|------|----------|------|
| 通用签名（新项目） | Ed25519 | 快速、安全、确定性 |
| 政府/合规要求 | NIST P-256 | FIPS 认证 |
| 密钥协商 | X25519 | 高性能、抗侧信道 |
| 区块链 | secp256k1 | 行业标准 |
| 高安全需求 | P-384 或 Ed448 | 192+ 位安全 |

## 7. 实际应用

| 场景 | 使用的 ECC 方案 |
|------|----------------|
| TLS 1.3 | ECDHE (X25519) + ECDSA/EdDSA |
| SSH | Ed25519 密钥认证 |
| 比特币 | secp256k1 + ECDSA |
| Signal/WhatsApp | X25519 + Ed25519 |
| WireGuard | Curve25519 |
| iOS/macOS 签名 | P-256 ECDSA |
| FIDO2/WebAuthn | P-256 ECDSA |

## 8. 总结

| 维度 | 评价 |
|------|------|
| 安全性 | ⭐⭐⭐⭐⭐ 256 位即达 128 位安全强度 |
| 性能 | ⭐⭐⭐⭐⭐ 远优于同等安全级别的 RSA |
| 密钥大小 | ⭐⭐⭐⭐⭐ 极为紧凑 |
| 生态 | ⭐⭐⭐⭐ 主流语言和协议全面支持 |
| 未来性 | ⭐⭐ 受量子计算威胁（与 RSA 相同） |

ECC 是当前公钥密码学的**最优选择**——在相同安全级别下，它比 RSA 更快、密钥更短、带宽更省。对于新系统，**Ed25519（签名）+ X25519（密钥协商）**是最推荐的组合。
