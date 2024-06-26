---
layout:     post
title:      DSA算法
subtitle:   非对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## 原理

`数字签名算法`（`DSA` - Digital Signature Algorithm）是用于`数字签名`的算法，基于`模算数和离散对数的复杂度`。DSA是Schnorr和ElGamal签名方案的变体。

DSA 算法包含了四种操作：密钥生成、密钥分发、签名、验证

1. 密钥生成

密钥生成包含两个阶段。第一阶段是`算法参数的选择`，可以在系统的不同用户之间共享，而第二阶段则`为每个用户计算独立的密钥组合`。

![在这里插入图片描述](/png/cryptography/dsa1.png)

2. 密钥分发

签名者需要透过可信任的管道发布`公钥` y，并且安全地保护 x 不被其他人知道。

3. 签名流程

![在这里插入图片描述](/png/cryptography/dsa2.png)

4. 验证签名

![在这里插入图片描述](/png/cryptography/dsa3.png)

---

下列密码学库有提供 DSA 的支持：

- OpenSSL
- GnuTLS
- wolfCrypt
- Crypto++
- cryptlib
- Botan
- Bouncy Castle
- libgcrypt
- Nettle

数据来源 -- 维基百科

## Java jdk实现

DsaUtils.java

```java
package crypto.dsa;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.security.*;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**
 * 非对称加密 DSA 不能用于加密数据，只能用于数字签名
 */
public class DsaUtils {

    private static final String ALGORITHM = "DSA";

    /**
     * @link {https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#Signature}
     */
    private static final String DEFAULT_SIGNATURE_ALGORITHM = "SHA1withDSA";

    /**
     * This must be a multiple of 64, ranging from 512 to 1024 (inclusive), or 2048. The default keysize is 1024.
     */
    private static final int DEFAULT_KEY_SIZE = 1024;

    /**
     * 生成密钥对
     *
     * @link {https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#KeyPairGenerator}
     */
    public static InnerKey generateKey() throws NoSuchAlgorithmException {
        return generateKey(DEFAULT_KEY_SIZE);
    }

    /**
     * 生成密钥对
     *
     * @param keysize
     * @return
     * @throws NoSuchAlgorithmException
     */
    public static InnerKey generateKey(int keysize) throws NoSuchAlgorithmException {
        // 初始化密钥
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        keyPairGenerator.initialize(keysize);

        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        //DSAPublicKey dsaPublicKey = (DSAPublicKey) keyPair.getPublic();
        //DSAPrivateKey dsaPrivateKey = (DSAPrivateKey) keyPair.getPrivate();
        return InnerKey.builder()
                .publicKey(keyPair.getPublic().getEncoded())
                .privateKey(keyPair.getPrivate().getEncoded())
                .build();
    }

    public static byte[] sign(byte[] privateKey, byte[] data)
            throws InvalidKeySpecException, NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        return sign(privateKey, data, DEFAULT_SIGNATURE_ALGORITHM);
    }

    /**
     * 使用私钥进行签名
     *
     * @param privateKey         私钥
     * @param data               数据
     * @param signatureAlgorithm 签名算法
     * @return
     * @throws Exception
     */
    public static byte[] sign(byte[] privateKey, byte[] data, String signatureAlgorithm)
            throws NoSuchAlgorithmException, InvalidKeySpecException, InvalidKeyException, SignatureException {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);

        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(privateKey);
        PrivateKey privateKey2 = keyFactory.generatePrivate(pkcs8EncodedKeySpec);

        Signature signature = Signature.getInstance(signatureAlgorithm);
        signature.initSign(privateKey2);
        signature.update(data);
        byte[] bytes = signature.sign();
        return bytes;
    }

    public static boolean verifySign(byte[] publicKey, byte[] data, byte[] sign)
            throws InvalidKeySpecException, NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        return verifySign(publicKey, data, sign, DEFAULT_SIGNATURE_ALGORITHM);
    }

    /**
     * 使用公钥验证签名
     *
     * @param publicKey          公钥
     * @param data               数据
     * @param sign               数据签名
     * @param signatureAlgorithm 签名算法
     * @return
     * @throws Exception
     */
    public static boolean verifySign(byte[] publicKey, byte[] data, byte[] sign, String signatureAlgorithm)
            throws NoSuchAlgorithmException, InvalidKeySpecException, InvalidKeyException, SignatureException {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);

        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(publicKey);
        PublicKey publicKey2 = keyFactory.generatePublic(x509EncodedKeySpec);

        Signature signature = Signature.getInstance(signatureAlgorithm);
        signature.initVerify(publicKey2);
        signature.update(data);
        boolean bool = signature.verify(sign);
        return bool;
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class InnerKey {
        private byte[] publicKey;
        private byte[] privateKey;
    }
}

```

测试代码

```java
package crypto.dsa;

import org.apache.commons.codec.binary.Base64;

public class DsaUtilsTest {

    public static void main(String[] args) throws Exception {
        String text = "你好世界 DSA签名";
        DsaUtils.InnerKey innerKey = DsaUtils.generateKey();
        System.out.println("公钥：" + Base64.encodeBase64String(innerKey.getPublicKey()));
        System.out.println("私钥：" + Base64.encodeBase64String(innerKey.getPrivateKey()));

        byte[] sign = DsaUtils.sign(innerKey.getPrivateKey(), text.getBytes());
        System.out.println("原文:" + text);
        System.out.println("数字签名：" + Base64.encodeBase64String(sign));
        boolean bool = DsaUtils.verifySign(innerKey.getPublicKey(), text.getBytes(), sign);
        System.out.println("验签结果：" + bool);
    }
}
```

## Java jdk实现 ECDSA

```java
package crypto.dsa;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

public class EcDsaUtils {

    private static final String ALGORITHM = "EC";

    /**
     * NONEwithECDSA
     * SHA1withECDSA
     * SHA224withECDSA
     * SHA256withECDSA
     * SHA384withECDSA
     * SHA512withECDSA
     *
     * @link {https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#Signature}
     */
    public static final String DEFAULT_SIGNATURE_ALGORITHM = "SHA1withECDSA";

    public enum SignatureAlgorithm {
        NONEwithECDSA,
        SHA1withECDSA,
        SHA224withECDSA,
        SHA256withECDSA,
        SHA384withECDSA,
        SHA512withECDSA
    }

    public static InnerKey generateKey() throws Exception {
        return generateKey(256);
    }

    /**
     * 初始化密钥
     *
     * @param keySize Keysize must range from 112 to 571 (inclusive).
     * @return
     * @throws Exception
     * @link {https://docs.oracle.com/javase/8/docs/technotes/guides/security/SunProviders.html#SunEC}
     */
    public static InnerKey generateKey(int keySize) throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        keyPairGenerator.initialize(keySize);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        //ECPublicKey ecPublicKey = (ECPublicKey) keyPair.getPublic();
        //ECPrivateKey ecPrivateKey = (ECPrivateKey) keyPair.getPrivate();
        return InnerKey.builder()
                .publicKey(keyPair.getPublic().getEncoded())
                .privateKey(keyPair.getPrivate().getEncoded())
                .build();
    }

    public static byte[] sign(byte[] privateKey, byte[] data) throws Exception {
        return sign(privateKey, data, DEFAULT_SIGNATURE_ALGORITHM);
    }

    /**
     * 执行签名
     *
     * @param privateKey 私钥
     * @param data       数据
     * @param algorithm  签名算法 {@link SignatureAlgorithm}
     * @return
     * @throws Exception
     */
    public static byte[] sign(byte[] privateKey, byte[] data, String algorithm) throws Exception {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);

        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(privateKey);
        PrivateKey privateKey2 = keyFactory.generatePrivate(pkcs8EncodedKeySpec);
        Signature signature = Signature.getInstance(algorithm);
        signature.initSign(privateKey2);
        signature.update(data);
        return signature.sign();
    }

    public static boolean verifySign(byte[] publicKey, byte[] data, byte[] sign) throws Exception {
        return verifySign(publicKey, data, sign, DEFAULT_SIGNATURE_ALGORITHM);
    }

    /**
     * 验证签名
     *
     * @param publicKey 公钥
     * @param data      数据
     * @param sign      数据签名
     * @param algorithm 签名算法 {@link SignatureAlgorithm}
     * @return
     * @throws Exception
     */
    public static boolean verifySign(byte[] publicKey, byte[] data, byte[] sign, String algorithm) throws Exception {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);

        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(publicKey);
        PublicKey publicKey2 = keyFactory.generatePublic(x509EncodedKeySpec);

        Signature signature = Signature.getInstance(algorithm);
        signature.initVerify(publicKey2);
        signature.update(data);
        return signature.verify(sign);
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class InnerKey {
        private byte[] publicKey;
        private byte[] privateKey;
    }
}

```

```java
package crypto.dsa;

import org.apache.commons.codec.binary.Base64;

public class EcDsaUtilsTest {

    public static void main(String[] args) throws Exception {
        String text = "你好世界 ECDSA签名";
        EcDsaUtils.InnerKey innerKey = EcDsaUtils.generateKey(112);
        System.out.println("公钥：" + Base64.encodeBase64String(innerKey.getPublicKey()));
        System.out.println("私钥：" + Base64.encodeBase64String(innerKey.getPrivateKey()));

        byte[] sign = EcDsaUtils.sign(innerKey.getPrivateKey(), text.getBytes(),
                EcDsaUtils.SignatureAlgorithm.SHA224withECDSA.name());
        System.out.println("原文:" + text);
        System.out.println("数字签名：" + Base64.encodeBase64String(sign));
        boolean bool = EcDsaUtils.verifySign(innerKey.getPublicKey(), text.getBytes(), sign,
                EcDsaUtils.SignatureAlgorithm.SHA224withECDSA.name());
        System.out.println(bool);
    }
}

```

[code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/dsa)