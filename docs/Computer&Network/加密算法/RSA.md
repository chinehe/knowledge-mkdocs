# RSA非对称加解密&签名算法

## 1. 算法概述

**RSA** 是世界上第一个也是最广泛使用的非对称（公钥）密码算法，由 Ron **R**ivest、Adi **S**hamir 和 Leonard **A**dleman 于 1977 年在 MIT 发明，以三人姓氏首字母命名。

RSA 既可用于**加密**也可用于**数字签名**，其安全性基于**大整数因数分解问题**的计算困难性。

### 1.1 基本参数

| 参数 | 值 |
|------|-----|
| 算法类型 | 非对称加密/签名 |
| 密钥长度 | 2048 / 3072 / 4096 位（推荐 ≥ 2048） |
| 数学基础 | 大整数因数分解问题 |
| 标准 | PKCS#1 (RFC 8017) |
| 安全状态 | ✅ 2048+ 位当前安全，但受量子计算威胁 |

### 1.2 核心特性

- **非对称**：公钥加密，私钥解密；或私钥签名，公钥验签
- **双重用途**：同时支持加密和数字签名
- **计算密集**：比对称加密慢 1000 倍以上，不适合直接加密大数据
- **广泛部署**：TLS、SSH、数字证书、代码签名等核心基础设施

## 2. 算法原理（简述）

### 2.1 密钥生成

1. 选择两个大质数 p 和 q（各 1024 位以上）
2. 计算 n = p × q（RSA 模数）
3. 计算欧拉函数 φ(n) = (p-1)(q-1)
4. 选择公钥指数 e，满足 gcd(e, φ(n)) = 1（常用 e = 65537）
5. 计算私钥指数 d = e⁻¹ mod φ(n)

**密钥对**：公钥 (n, e)，私钥 (n, d)

### 2.2 加密与解密

```
加密：C = M^e mod n（使用公钥）
解密：M = C^d mod n（使用私钥）
```

### 2.3 签名与验签

```
签名：S = Hash(M)^d mod n（使用私钥对消息哈希签名）
验签：Hash(M) =? S^e mod n（使用公钥验证）
```

> **为什么安全？** 知道公钥 (n, e) 想推出私钥 d，必须分解 n = p × q。对于 2048 位的 n，当前没有已知算法能在可接受时间内完成因式分解。

## 3. 填充方案（开发者必知）

**裸 RSA**（直接对消息做模幂运算）是**不安全的**，必须配合填充方案使用。这是开发者最容易忽略的点。

### 3.1 推荐方案

| 用途 | 推荐方案 | 标准 | 说明 |
|------|----------|------|------|
| **加密** | OAEP | PKCS#1 v2.x | 引入随机性，CCA2 安全，相同明文每次加密结果不同 |
| **签名** | PSS | PKCS#1 v2.x | 引入随机盐值，可证明安全性 |

### 3.2 旧方案（应避免）

| 方案 | 问题 | 建议 |
|------|------|------|
| PKCS#1 v1.5 加密 | 易受 Bleichenbacher 攻击（1998年） | 迁移到 OAEP |
| PKCS#1 v1.5 签名 | 存在实现层面漏洞 | 迁移到 PSS |

> **⚠️ 如果你在代码中看到 `PKCS1v15` 相关的加密调用，应考虑迁移到 OAEP。** 签名场景中 PKCS#1 v1.5 仍有广泛使用（如某些证书标准），但新系统推荐 PSS。

## 4. 安全性分析

### 4.1 密钥长度与安全性

| RSA 密钥长度 | 等效对称密钥 | 安全性评估 | 开发建议 |
|-------------|-------------|-----------|----------|
| 1024 位 | ~80 位 | ❌ 不安全 | 禁止使用 |
| 2048 位 | ~112 位 | ✅ 安全至 ~2030 年 | 最低推荐 |
| 3072 位 | ~128 位 | ✅ 安全至 ~2040 年 | 推荐 |
| 4096 位 | ~152 位 | ✅ 长期安全 | 高安全需求 |

### 4.2 已知攻击与防护

| 攻击方法 | 防护措施 |
|----------|----------|
| Bleichenbacher 攻击 | 使用 OAEP 替代 PKCS#1 v1.5 |
| 低指数攻击 | 使用 e = 65537 |
| 共模攻击 | 每个用户独立生成密钥对 |
| 时序攻击 | 使用标准库（已实现常数时间和 RSA 盲化） |
| Coppersmith 攻击 | 安全的密钥生成（使用标准库） |

### 4.3 量子计算威胁

> **🚨 Shor 算法可在量子计算机上以多项式时间分解大整数，这将彻底破解 RSA。**

当前量子计算机规模不足以威胁 2048+ 位 RSA，但应为后量子时代做准备：

- 长期保密数据（如医疗档案、政府机密）现在就应考虑后量子方案
- NIST 已发布后量子密码标准（ML-KEM、ML-DSA）
- 过渡方案：RSA + 后量子算法混合使用

## 5. 开发者实践指南

### 5.1 密钥长度选择

| 场景 | 推荐长度 | 理由 |
|------|----------|------|
| 通用场景 | 2048 位 | 平衡安全性和性能 |
| 需长期有效（>5年） | 3072 位 | 更大安全余量 |
| 高安全要求 | 4096 位 | 最大安全余量，但性能下降明显 |

### 5.2 使用要点

| 要点 | 说明 |
|------|------|
| 加密用 OAEP | `rsa.EncryptOAEP(sha256.New(), ...)` |
| 签名用 PSS | `rsa.SignPSS(...)` |
| 公钥指数 | 使用 65537（标准库默认） |
| 不要加密大数据 | RSA 一次只能加密 (密钥长度 - 填充开销) 的数据 |
| 使用混合加密 | 大数据场景：RSA 加密对称密钥 + AES 加密数据 |
| 密钥生成 | 使用标准库生成，不要自己实现素数选择 |

### 5.3 混合加密模式（实际使用方式）

RSA 在实际中几乎不会直接加密业务数据，而是用于加密对称密钥：

```
发送方：
1. 生成随机 AES-256 密钥 K
2. 用 AES-GCM + K 加密数据 → 密文 C
3. 用 RSA-OAEP 加密 K → 加密密钥 EK
4. 发送 (EK, C)

接收方：
1. 用 RSA 私钥解密 EK → K
2. 用 AES-GCM + K 解密 C → 明文
```

### 5.4 RSA vs ECC 选型

| 维度 | RSA-2048 | ECDSA P-256 | Ed25519 |
|------|----------|-------------|---------|
| 公钥大小 | 256 字节 | 64 字节 | 32 字节 |
| 签名大小 | 256 字节 | 64 字节 | 64 字节 |
| 签名速度 | 慢 | 中 | 快 |
| 验签速度 | 快 | 中 | 快 |
| 兼容性 | 最广泛 | 广泛 | 较新系统 |
| 推荐度 | 兼容性优先时 | 合规场景 | **新项目首选** |

### 5.5 密钥管理

| 实践 | 说明 |
|------|------|
| 私钥加密存储 | 使用密码保护 PEM 文件，或使用 HSM |
| 证书定期轮换 | 遵循组织的密钥轮换策略 |
| 密钥泄露应急 | 立即吊销证书，重新签发 |
| PEM 格式 | 公钥用 `PUBLIC KEY`，私钥用 `RSA PRIVATE KEY` 或 PKCS#8 |

## 6. Go 语言实现

```go
package main

import (
	"crypto"
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"crypto/x509"
	"encoding/pem"
	"fmt"
)

// 生成 RSA 密钥对
func GenerateKeyPair(bits int) (*rsa.PrivateKey, *rsa.PublicKey, error) {
	privateKey, err := rsa.GenerateKey(rand.Reader, bits)
	if err != nil {
		return nil, nil, err
	}
	return privateKey, &privateKey.PublicKey, nil
}

// RSA-OAEP 加密
func Encrypt(publicKey *rsa.PublicKey, plaintext []byte) ([]byte, error) {
	return rsa.EncryptOAEP(sha256.New(), rand.Reader, publicKey, plaintext, nil)
}

// RSA-OAEP 解密
func Decrypt(privateKey *rsa.PrivateKey, ciphertext []byte) ([]byte, error) {
	return rsa.DecryptOAEP(sha256.New(), rand.Reader, privateKey, ciphertext, nil)
}

// RSA-PSS 签名
func Sign(privateKey *rsa.PrivateKey, message []byte) ([]byte, error) {
	hash := sha256.Sum256(message)
	return rsa.SignPSS(rand.Reader, privateKey, crypto.SHA256, hash[:], nil)
}

// RSA-PSS 验签
func Verify(publicKey *rsa.PublicKey, message, signature []byte) error {
	hash := sha256.Sum256(message)
	return rsa.VerifyPSS(publicKey, crypto.SHA256, hash[:], signature, nil)
}

// 私钥导出为 PEM
func PrivateKeyToPEM(privateKey *rsa.PrivateKey) string {
	privDER := x509.MarshalPKCS1PrivateKey(privateKey)
	privBlock := &pem.Block{
		Type:  "RSA PRIVATE KEY",
		Bytes: privDER,
	}
	return string(pem.EncodeToMemory(privBlock))
}

// 公钥导出为 PEM
func PublicKeyToPEM(publicKey *rsa.PublicKey) string {
	pubDER, _ := x509.MarshalPKIXPublicKey(publicKey)
	pubBlock := &pem.Block{
		Type:  "PUBLIC KEY",
		Bytes: pubDER,
	}
	return string(pem.EncodeToMemory(pubBlock))
}

func main() {
	// 生成 2048 位密钥对
	privateKey, publicKey, err := GenerateKeyPair(2048)
	if err != nil {
		panic(err)
	}

	fmt.Println("=== RSA 密钥对生成 ===")
	fmt.Printf("密钥长度: %d 位\n", privateKey.N.BitLen())

	// 加密/解密示例
	plaintext := []byte("Hello, RSA-OAEP Encryption!")
	fmt.Printf("\n=== 加密/解密 ===\n")
	fmt.Printf("明文: %s\n", plaintext)

	ciphertext, err := Encrypt(publicKey, plaintext)
	if err != nil {
		panic(err)
	}
	fmt.Printf("密文长度: %d 字节\n", len(ciphertext))

	decrypted, err := Decrypt(privateKey, ciphertext)
	if err != nil {
		panic(err)
	}
	fmt.Printf("解密: %s\n", decrypted)

	// 签名/验签示例
	message := []byte("This message needs to be signed.")
	fmt.Printf("\n=== 签名/验签 ===\n")
	fmt.Printf("消息: %s\n", message)

	signature, err := Sign(privateKey, message)
	if err != nil {
		panic(err)
	}
	fmt.Printf("签名长度: %d 字节\n", len(signature))

	err = Verify(publicKey, message, signature)
	if err != nil {
		fmt.Println("验签失败:", err)
	} else {
		fmt.Println("验签成功 ✓")
	}

	// 篡改消息后验签
	tamperedMessage := []byte("This message has been tampered.")
	err = Verify(publicKey, tamperedMessage, signature)
	if err != nil {
		fmt.Println("篡改后验签失败 ✓ (预期行为):", err)
	}
}
```

## 7. 实际应用场景

| 场景 | 用法 | 说明 |
|------|------|------|
| TLS/HTTPS | 密钥交换 + 服务器认证 | RSA 签名验证证书 |
| 数字证书 (X.509) | CA 签名 | 签发和验证证书链 |
| 代码签名 | 签名 | 验证软件来源和完整性 |
| SSH | 身份认证 | RSA 密钥对登录 |
| JWT | 签名 | RS256/RS384/RS512 |
| PGP/GPG | 加密 + 签名 | 邮件加密和签名 |

## 8. 总结

| 维度 | 评价 |
|------|------|
| 安全性 | ⭐⭐⭐⭐ 2048+ 位当前安全，但受量子计算威胁 |
| 性能 | ⭐⭐ 显著慢于对称加密和 ECC |
| 生态 | ⭐⭐⭐⭐⭐ 全球最广泛部署的公钥算法 |
| 未来性 | ⭐⭐ 量子计算将构成根本性威胁 |
| 开发者建议 | 签名新项目优先用 Ed25519；加密用 ECDH+AES；兼容性需求用 RSA |

**开发者决策指南**：

- **需要数字签名？** → 新项目用 Ed25519，兼容性要求用 RSA-PSS
- **需要加密小数据/密钥？** → RSA-OAEP 或 ECIES
- **需要加密大数据？** → 混合加密（RSA/ECDH 加密对称密钥 + AES-GCM 加密数据）
- **需要考虑后量子安全？** → 关注 NIST ML-KEM / ML-DSA 标准
