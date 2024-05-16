---
layout:     post
title:      PBE算法
subtitle:   对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## 概念

PBE算法（Password Based Encryption，基于口令加密）是一种基于口令的加密算法，其特点是使用口令代替了密钥，而口令由用户自己掌管，采用随机数杂凑多重加密等方法保证数据的安全性。

## 加密过程

PBE算法在加密过程中并不是直接使用口令来加密，而是加密的密钥由口令生成，这个功能由PBE算法中的KDF函数完成。KDF函数的实现过程为：将用户输入的口令首先通过“盐”（salt）的扰乱产生准密钥，再将准密钥经过散列函数多次迭代后生成最终加密密钥，密钥生成后，PBE算法再选用对称加密算法对数据进行加密，可以选择DES、3DES、RC5等对称加密算法 。

## java jdk 实现

PbeUtils.java

```java
package crypto.pbe;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.PBEParameterSpec;
import java.security.SecureRandom;

/**
 * 对称加密 PBE算法 java实现
 */
public class PbeUtils {

    /**
     * @link {https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#Algorithms}
     */
    private static final String ALGORITHM = "PBEWITHMD5andDES";

    /**
     * 获取随机的8字节长的盐值
     */
    public static byte[] getSalt() {
        return getSalt(8);
    }

    /**
     * 获取随机的 numBytes 字节长的盐值
     *
     * @param numBytes 盐值的字节长度
     */
    public static byte[] getSalt(int numBytes) {
        SecureRandom random = new SecureRandom();
        byte[] salt = random.generateSeed(numBytes);
        return salt;
    }


    /**
     * @param password 口令
     */
    private static SecretKey getKey(String password) throws Exception {
        PBEKeySpec pbeKeySpec = new PBEKeySpec(password.toCharArray());
        SecretKeyFactory factory = SecretKeyFactory.getInstance(ALGORITHM);
        SecretKey secretKey = factory.generateSecret(pbeKeySpec);
        return secretKey;
    }

    public static byte[] encrypt(String password, byte[] data) throws Exception {
        return encrypt(password, new byte[]{0, 0, 0, 0, 0, 0, 0, 0}, 100, data);
    }

    /**
     * 加密
     *
     * @param password       口令
     * @param salt           盐值
     * @param data           需要加密的数据
     * @param iterationCount 迭代次数
     */
    public static byte[] encrypt(String password, byte[] salt, int iterationCount, byte[] data) throws Exception {
        PBEParameterSpec pbeParameterSpec = new PBEParameterSpec(salt, iterationCount);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, getKey(password), pbeParameterSpec);
        byte[] bytes = cipher.doFinal(data);
        return bytes;
    }

    public static byte[] decrypt(String password, byte[] data) throws Exception {
        return decrypt(password, new byte[]{0, 0, 0, 0, 0, 0, 0, 0}, 100, data);
    }

    /**
     * 解密
     *
     * @param password       口令
     * @param salt           盐值
     * @param data           需要解密的数据
     * @param iterationCount 迭代次数
     */
    public static byte[] decrypt(String password, byte[] salt, int iterationCount, byte[] data) throws Exception {
        PBEParameterSpec pbeParameterSpec = new PBEParameterSpec(salt, iterationCount);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, getKey(password), pbeParameterSpec);
        return cipher.doFinal(data);
    }

}

```

```java
package crypto.pbe;

import org.apache.commons.codec.binary.Base64;
import org.apache.commons.codec.binary.Hex;

public class PbeUtilsTest {

    public static void main(String[] args) throws Exception {
        String text = "世界人民大团结万岁";
        byte[] salt = PbeUtils.getSalt();
        System.out.println("盐值：" + Hex.encodeHexString(salt));
        String password = "123456";// 口令
        int iterationCount = 10;

        byte[] encryptBytes = PbeUtils.encrypt(password, salt, iterationCount, text.getBytes());
        System.out.println("密文：" + Base64.encodeBase64String(encryptBytes));
        byte[] result = PbeUtils.decrypt(password, salt, iterationCount, encryptBytes);
        System.out.println("解密：" + new String(result));
    }
}

```


[code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/pbe)