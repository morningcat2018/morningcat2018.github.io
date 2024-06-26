---
layout:     post
title:      RSA算法
subtitle:   非对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## 概念
RSA是由罗纳德·李维斯特（Ron Rivest）、阿迪·萨莫尔（Adi Shamir）和伦纳德·阿德曼（Leonard Adleman）在1977年一起提出的。当时他们三人都在麻省理工学院工作。RSA 就是他们三人姓氏开头字母拼在一起组成的。

RSA算法（可用于`加密`和`数字签名`）的安全性基于这样的事实：`大整数的因式分解` 被认为是‘难以破解’的。

## 原理

1. 公钥与私钥的产生
![在这里插入图片描述](/png/cryptography/rsa1.png)

2. 加密消息
![在这里插入图片描述](/png/cryptography/rsa2.png)

3. 解密消息
![在这里插入图片描述](/png/cryptography/rsa3.png)

## java jdk实现

```java
package crypto.rsa;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.crypto.Cipher;
import java.security.*;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**
 * 非对称加密 RSA
 */
public class RsaUtils {

    private static final String ALGORITHM = "RSA";

    private static final String SIGNATURE_ALGORITHM = "MD5withRSA";

    public static InnerKey generateKey() throws NoSuchAlgorithmException {
        return generateKey(1024);
    }

    /**
     * 初始化密钥
     * <p>
     * 工作模式 ECB
     * 填充方式 NoPadding PKCS1Padding ...
     *
     * @param keysize 默认1024；范围在 [512~65536] ,且需要为 64 的倍数
     * @return
     * @throws NoSuchAlgorithmException
     */
    public static InnerKey generateKey(int keysize) throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        keyPairGenerator.initialize(keysize);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        //RSAPublicKey rsaPublicKey = (RSAPublicKey) keyPair.getPublic();
        //RSAPrivateKey rsaPrivateKey = (RSAPrivateKey) keyPair.getPrivate();
        return InnerKey.builder()
                .publicKey(keyPair.getPublic().getEncoded())
                .privateKey(keyPair.getPrivate().getEncoded())
                .build();
    }

    private static PublicKey getPublicKey(byte[] key) throws NoSuchAlgorithmException, InvalidKeySpecException {
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(key);
        PublicKey publicKey = KeyFactory.getInstance(ALGORITHM).generatePublic(x509EncodedKeySpec);
        return publicKey;
    }

    private static PrivateKey getPrivateKey(byte[] key) throws NoSuchAlgorithmException, InvalidKeySpecException {
        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(key);
        PrivateKey privateKey = KeyFactory.getInstance(ALGORITHM).generatePrivate(pkcs8EncodedKeySpec);
        return privateKey;
    }

    /**
     * 私钥加密 公钥解密---加密
     *
     * @param privateKey
     * @param data
     * @return
     * @throws Exception
     */
    public static byte[] encryptByPrivateKey(byte[] privateKey, byte[] data) throws Exception {
        return encrypt(true, privateKey, data);
    }

    /**
     * 私钥加密 公钥解密---解密
     *
     * @param publicKey
     * @param data
     * @return
     * @throws Exception
     */
    public static byte[] decryptByPublicKey(byte[] publicKey, byte[] data) throws Exception {
        return decrypt(false, publicKey, data);
    }

    /**
     * 公钥加密 私钥解密---加密
     *
     * @param publicKey
     * @param data
     * @return
     * @throws Exception
     */
    public static byte[] encryptByPublicKey(byte[] publicKey, byte[] data) throws Exception {
        return encrypt(false, publicKey, data);
    }

    /**
     * 公钥加密 私钥解密---解密
     *
     * @param privateKey
     * @param data
     * @return
     * @throws Exception
     */
    public static byte[] decryptByPrivateKey(byte[] privateKey, byte[] data) throws Exception {
        return decrypt(true, privateKey, data);
    }

    private static byte[] encrypt(boolean isPrivate, byte[] key, byte[] data) throws Exception {
        Key thisKey = isPrivate ? getPrivateKey(key) : getPublicKey(key);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, thisKey);
        return cipher.doFinal(data);
    }

    private static byte[] decrypt(boolean isPrivate, byte[] key, byte[] data) throws Exception {
        Key thisKey = isPrivate ? getPrivateKey(key) : getPublicKey(key);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, thisKey);
        return cipher.doFinal(data);
    }

    /**
     * 用私钥对信息生成数字签名
     *
     * @param privateKey
     * @param data
     * @return
     * @throws Exception
     */
    public static byte[] sign(byte[] privateKey, byte[] data) throws Exception {
        //
        Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);
        signature.initSign(getPrivateKey(privateKey));
        signature.update(data);
        return signature.sign();
    }

    /**
     * 验证签名
     *
     * @param publicKey
     * @param data
     * @param sign
     * @return
     * @throws Exception
     */
    public static boolean verifySign(byte[] publicKey, byte[] data, byte[] sign) throws Exception {
        Signature signature = Signature.getInstance(SIGNATURE_ALGORITHM);
        signature.initVerify(getPublicKey(publicKey));
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

测试代码

```java
package crypto.rsa;

import org.apache.commons.codec.binary.Base64;

public class RsaUtilsTest {

    public static void main(String[] args) throws Exception {
        RsaUtils.InnerKey innerKey = RsaUtils.generateKey();
        System.out.println("公钥：" + Base64.encodeBase64String(innerKey.getPublicKey()));
        System.out.println("私钥：" + Base64.encodeBase64String(innerKey.getPrivateKey()));

        byte[] data = "你好，世界".getBytes();
        testEncrypt1(innerKey.getPublicKey(), innerKey.getPrivateKey(), data);
        testEncrypt2(innerKey.getPublicKey(), innerKey.getPrivateKey(), data);
        testSign(innerKey.getPublicKey(), innerKey.getPrivateKey(), data);
    }

    private static void testEncrypt1(byte[] publicKey, byte[] privateKey, byte[] data) throws Exception {
        // 私钥加密 公钥解密---加密
        byte[] encryptBytes = RsaUtils.encryptByPrivateKey(privateKey, data);
        System.out.println("私钥加密公钥解密-密文：" + Base64.encodeBase64String(encryptBytes));
        // 私钥加密 公钥解密---解密
        byte[] result = RsaUtils.decryptByPublicKey(publicKey, encryptBytes);
        System.out.println("解密后：" + new String(result));
    }

    private static void testEncrypt2(byte[] publicKey, byte[] privateKey, byte[] data) throws Exception {
        // 公钥加密 私钥解密---加密
        byte[] encryptBytes = RsaUtils.encryptByPublicKey(publicKey, data);
        System.out.println("公钥加密私钥解密-密文：" + Base64.encodeBase64String(encryptBytes));
        // 公钥加密 私钥解密---解密
        byte[] result = RsaUtils.decryptByPrivateKey(privateKey, encryptBytes);
        System.out.println("解密后：" + new String(result));
    }

    private static void testSign(byte[] publicKey, byte[] privateKey, byte[] data) throws Exception {
        byte[] sign = RsaUtils.sign(privateKey, data);
        System.out.println("签名：" + Base64.encodeBase64String(sign));
        boolean result = RsaUtils.verifySign(publicKey, data, sign);
        System.out.println("验签结果：" + result);
    }
}

```

## java bc实现

[code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/rsa)