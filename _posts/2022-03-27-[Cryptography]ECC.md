---
layout:     post
title:      ECC算法
subtitle:   非对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## ECC 算法

`椭圆曲线密码学`（`Elliptic Curve Cryptography`，缩写：ECC）是一种`基于椭圆曲线数学`的公开密钥加密算法。

ECC的主要优势是它相比RSA加密算法使用较小的密钥长度并提供相当等级的安全性。

![在这里插入图片描述](/png/cryptography/ecc1.png)


## java jdk未提供ecc实现

## java bouncycastle实现

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.70</version>
</dependency>
```

EccUtils.java

```java
package crypto.ec;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.Cipher;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**
 * bouncycastle @link{https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on/1.70} 实现的ECC非对称加密实现
 */
public class EccUtils {

    private static final String ALGORITHM = "EC";

    private static final String DEFAULT_TRANSFORMATION = "ECIES";

    private static final String PROVIDER = "BC";

    static {
        Provider provider = new BouncyCastleProvider();
        Security.addProvider(provider);
        //Provider[] providers = Security.getProviders();
        //Arrays.asList(providers).stream().forEach(p -> print(p));
    }

    private static void print(Provider provider) {
        System.out.println("\n服务提供方：" + provider.getName());
        provider.getServices().forEach(service -> System.out.println(service.getType() + " : " + service.getAlgorithm()));
    }

    public static InnerKey generateKey() throws Exception {
        return generateKey(224);
    }

    /**
     * 初始化密钥
     *
     * @param keySize 192 bit, 224 bit, 256 bit
     * @return
     * @throws Exception
     */
    public static InnerKey generateKey(int keySize) throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM, PROVIDER);
        keyPairGenerator.initialize(keySize);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        return InnerKey.builder().publicKey(keyPair.getPublic().getEncoded()).privateKey(keyPair.getPrivate().getEncoded()).build();
    }

    public static byte[] encrypt(byte[] publicKey, byte[] data) throws Exception {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM, PROVIDER); // https://gitee.com/dromara/hutool/issues/I1UYNJ
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(publicKey);
        PublicKey publicKey2 = keyFactory.generatePublic(x509EncodedKeySpec);

        Cipher cipher = Cipher.getInstance(DEFAULT_TRANSFORMATION, PROVIDER);
        cipher.init(Cipher.ENCRYPT_MODE, publicKey2);
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(byte[] privateKey, byte[] data) throws Exception {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM, PROVIDER);
        PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(privateKey);
        PrivateKey privateKey2 = keyFactory.generatePrivate(pkcs8KeySpec);

        Cipher cipher = Cipher.getInstance(DEFAULT_TRANSFORMATION, PROVIDER);
        cipher.init(Cipher.DECRYPT_MODE, privateKey2);
        return cipher.doFinal(data);
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
package crypto.ec;

import org.apache.commons.codec.binary.Base64;

public class EccUtilsTest {

    public static void main(String[] args) throws Exception {
        EccUtils.InnerKey key = EccUtils.generateKey(256);
        System.out.println("公钥：" + Base64.encodeBase64String(key.getPublicKey()));
        System.out.println("私钥：" + Base64.encodeBase64String(key.getPrivateKey()));

        byte[] bytes = EccUtils.encrypt(key.getPublicKey(), "你好中国".getBytes());
        System.out.println("密文：" + Base64.encodeBase64String(bytes));
        byte[] result = EccUtils.decrypt(key.getPrivateKey(), bytes);
        System.out.println(new String(result));
    }
}

```

[code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/ec)