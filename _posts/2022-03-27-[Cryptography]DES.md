---
layout:     post
title:      DES算法
subtitle:   对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## 概念

`数据加密标准`（Data Encryption Standard，缩写为 `DES`）是一种`对称`密钥加密块密码算法

DES现在已经不是一种安全的加密方法，主要因为它使用的56位密钥过短。
1999年1月，distributed.net与电子前哨基金会合作，在22小时15分钟内即公开破解了一个DES密钥。
也有一些分析报告提出了该算法的理论上的弱点，虽然在实际中难以应用。
为了提供实用所需的安全性，可以使用DES的派生算法3DES来进行加密，
虽然3DES也存在理论上的攻击方法。DES标准和3DES标准已逐渐被高级加密标准（AES）所取代。

## java jdk 实现

```java
package crypto.des;

import crypto.aes.AesUtils;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;

/**
 * 对称加密 DES算法 jdk实现
 */
public class DesUtils {

    private static final String ALGORITHM = "DES";

    public enum MODE {
        ECB,
        CBC,
        PCBC,
        CTR,
        CFB,
        OFB
    }

    public enum PADDING {
        NoPadding,
        PKCS5Padding,
        ISO10126Padding
    }

    public static byte[] initKey() throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(ALGORITHM);
        keyGenerator.init(56);// 密钥长度
        SecretKey secretKey = keyGenerator.generateKey();
        return secretKey.getEncoded();
    }

    private static SecretKey buildKey(byte[] key)
            throws InvalidKeyException, NoSuchAlgorithmException, InvalidKeySpecException {
        // KEY转换
        DESKeySpec desKeySpec = new DESKeySpec(key);
        SecretKeyFactory factory = SecretKeyFactory.getInstance(ALGORITHM);
        SecretKey secretKey = factory.generateSecret(desKeySpec);
        return secretKey;
    }

    public static byte[] encrypt(byte[] key, byte[] data) throws Exception {
        return encrypt(key, data, AesUtils.MODE.ECB.name(), AesUtils.PADDING.PKCS5Padding.name());
    }

    public static byte[] encrypt(byte[] key, byte[] data, String mode, String pad) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad));
        cipher.init(Cipher.ENCRYPT_MODE, buildKey(key));
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(byte[] key, byte[] data) throws Exception {
        return decrypt(key, data, AesUtils.MODE.ECB.name(), AesUtils.PADDING.PKCS5Padding.name());
    }

    public static byte[] decrypt(byte[] key, byte[] data, String mode, String pad) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad));
        cipher.init(Cipher.DECRYPT_MODE, buildKey(key));
        return cipher.doFinal(data);
    }
}

```

```java
package crypto.des;

import org.apache.commons.codec.binary.Base64;

public class DesUtilsTest {

    public static void main(String[] args) throws Exception {
        byte[] key = DesUtils.initKey();
        System.out.println("密钥：" + Base64.encodeBase64String(key));

        byte[] data = DesUtils.encrypt(key, "你好 world".getBytes());
        System.out.println("密文：" + Base64.encodeBase64String(data));
        byte[] result = DesUtils.decrypt(key, data);
        System.out.println("明文：" + new String(result));
    }
}
```

## 3DES jdk 实现

```java
package crypto.des;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESedeKeySpec;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;

/**
 * 对称加密 三重DES算法 jdk实现
 */
public class DesTripleUtils {

    private static final String ALGORITHM = "DESede";

    public enum MODE {
        ECB,
        CBC,
        PCBC,
        CTR,
        CFB,
        OFB
    }

    public enum PADDING {
        NoPadding,
        PKCS5Padding,
        ISO10126Padding
    }

    public static byte[] initKey() throws NoSuchAlgorithmException {
        return initKey(168);
    }

    /**
     * @param keySize must be equal to 112 or 168
     * @return
     * @throws NoSuchAlgorithmException
     */
    public static byte[] initKey(int keySize) throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(ALGORITHM);
        keyGenerator.init(keySize);
        SecretKey secretKey = keyGenerator.generateKey();
        return secretKey.getEncoded();
    }

    private static SecretKey buildKey(byte[] key)
            throws InvalidKeyException, NoSuchAlgorithmException, InvalidKeySpecException {
        // KEY转换
        DESedeKeySpec desKeySpec = new DESedeKeySpec(key);
        SecretKeyFactory factory = SecretKeyFactory.getInstance(ALGORITHM);
        SecretKey secretKey = factory.generateSecret(desKeySpec);
        return secretKey;
    }

    public static byte[] encrypt(byte[] key, byte[] data) throws Exception {
        return encrypt(key, data, MODE.ECB.name(), PADDING.PKCS5Padding.name());
    }

    public static byte[] encrypt(byte[] key, byte[] data, String mode, String pad) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad));
        cipher.init(Cipher.ENCRYPT_MODE, buildKey(key));
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(byte[] key, byte[] data) throws Exception {
        return decrypt(key, data, MODE.ECB.name(), PADDING.PKCS5Padding.name());
    }

    public static byte[] decrypt(byte[] key, byte[] data, String mode, String pad) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad));
        cipher.init(Cipher.DECRYPT_MODE, buildKey(key));
        return cipher.doFinal(data);
    }
}
```

```java
package crypto.des;

import org.apache.commons.codec.binary.Base64;

public class DesTripleUtilsTest {

    public static void main(String[] args) throws Exception {
        byte[] key = DesTripleUtils.initKey();
        System.out.println("密钥：" + Base64.encodeBase64String(key));

        byte[] data = DesTripleUtils.encrypt(key, "你好 world".getBytes());
        System.out.println("密文：" + Base64.encodeBase64String(data));
        byte[] result = DesTripleUtils.decrypt(key, data);
        System.out.println("明文：" + new String(result));
    }
}
```

[code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/des)