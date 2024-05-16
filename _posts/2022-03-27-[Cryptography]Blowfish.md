---
layout:     post
title:      Blowfish算法
subtitle:   对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## 概念

`Blowfish` 是一个对称密钥加密分组密码算法，由布鲁斯·施奈尔于1993年设计，现已应用在多种加密产品。
Blowfish 算法由于分组长度太小已被认为不安全，施奈尔更建议在现代应用中使用 `Twofish` 密码。

施奈尔设计的Blowfish算法用途广泛，意在替代老旧的DES及避免其他算法的问题与限制。
Blowfish刚刚研发出的时候，大部分其他加密算法是专利所有的或属于商业(政府)机密，所以发展起来非常受限制。
施奈尔则声明Blowfish的使用没有任何限制，任何国家任何人任何时候都可以随意使用Blowfish算法。

---

Twofish的标志性特点是它采用了和密钥相关的替换盒（S盒）。
密钥输入位的一半被用于“真正的”加密流程进行编排并作为Feistel的轮密钥使用，而另一半用于修改算法所使用的S盒。
Twofish的密钥编排非常复杂。

软件实现的128位Twofish在大多数平台上的运行速度不及最终胜出AES评选的128位Rijndael算法，
不过，256位的Twofish运行速度却较AES-256稍快。

## java jdk 实现

```java
package crypto;


import org.apache.commons.codec.binary.Hex;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.Provider;
import java.security.Security;

/**
 * @describe: Blowfish是一个对称密钥加密分组密码算法，由布鲁斯·施奈尔于1993年设计，现已应用在多种加密产品。
 * Blowfish算法由于分组长度太小已被认为不安全，施奈尔更建议在现代应用中使用Twofish密码。
 * @author: morningcat.zhang
 * @date: 2022/4/9 下午7:35
 */
public class BlowFishUtils {

    private static final String ALGORITHM = "Twofish";
    // "Blowfish"
    // "Twofish"

    static {
        Provider provider = new BouncyCastleProvider();
        Security.addProvider(provider);
    }

    public static byte[] getKey() throws Exception {
        KeyGenerator keygenerator = KeyGenerator.getInstance(ALGORITHM);
        SecretKey secretkey = keygenerator.generateKey();
        return secretkey.getEncoded();
    }

    public static byte[] encrypt(byte[] key, byte[] data) throws Exception {
        SecretKeySpec secretKeySpec = new SecretKeySpec(key, ALGORITHM);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
        byte[] encrypted = cipher.doFinal(data);
        return encrypted;
    }

    public static byte[] decrypt(byte[] key, byte[] data) throws Exception {
        SecretKeySpec secretKeySpec = new SecretKeySpec(key, ALGORITHM);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);
        byte[] decrypted = cipher.doFinal(data);
        return decrypted;
    }

    public static void main(String[] args) throws Exception {
        byte[] key = getKey();
        System.out.println(Hex.encodeHexString(key));
        byte[] encrypted = encrypt(key, "Blowfish是一个对称密钥加密分组密码算法".getBytes());
        System.out.println(Hex.encodeHexString(encrypted));
        byte[] decrypted = decrypt(key, encrypted);
        System.out.println(new String(decrypted));
    }
}
```