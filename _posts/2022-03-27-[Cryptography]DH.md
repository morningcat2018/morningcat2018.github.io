---
layout:     post
title:      DH算法
subtitle:   非对称加密
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## DH算法简介

迪菲-赫尔曼密钥交换（`Diffie–Hellman` key exchange，缩写为`D-H`） 是一种安全协议。
它可以让双方在完全没有对方任何预先信息的条件下通过不安全信道创建起一个密钥。
这个密钥可以在后续的通讯中作为`对称密钥`来加密通讯内容。

迪菲－赫尔曼通过公共信道交换一个信息，就可以创建一个可以用于在公共信道上安全通信的`对称密钥`

> 交换过程
![在这里插入图片描述](/png/cryptography/dh1.png)

## 原理

最简单，最早提出的个协议使用`一个质数p的整数模n乘法群以及其原根g`。下面展示这个算法，`绿色`表示非秘密信息，`红色`粗体表示秘密信息：
![在这里插入图片描述](/png/cryptography/dh2.png)

爱丽丝和鲍伯最终都得到了同样的值，因为在模p下 g^{ab} 和 g^{ba} 相等。 注意a, b 和 g^ab = g^ba mod p 是秘密的。 其他所有的值 p, g, `g^a mod p`, 以及 `g^b mod p` 都可以在公共信道上传递。 一旦爱丽丝和鲍伯得出了公共秘密，他们就可以把它用作对称密钥，以进行双方的加密通讯，因为这个密钥只有他们才能得到。

这个问题就是著名的离散对数问题。注意g则不需要很大，并且在一般的实践中通常是2或者5。IETF RFC3526 文档中有几个常用的大质数可供使用。

---

以下是一个更为一般的描述:

1. 爱丽丝和鲍伯协商一个有限循环群 G 和它的一个生成元 g。 （这通常在协议开始很久以前就已经规定好； g是公开的，并可以被所有的攻击者看到。）
2. 爱丽丝选择一个随机自然数 a 并且将 g^a mod p 发送给鲍伯。
3. 鲍伯选择一个随机自然数 b 并且将 g^b mod p 发送给爱丽丝。
4. 爱丽丝 计算 g^{ba} mod p
5. 鲍伯 计算 g^{ab}  mod p

## java jdk实现

DhUtils.java

```java
package crypto.dh2;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.crypto.*;
import javax.crypto.interfaces.DHPublicKey;
import javax.crypto.spec.DHParameterSpec;
import java.security.*;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**
 * 非对称加密：DH算法 jdk实现
 * <p>
 * DH算法的本质就是双方各自生成自己的私钥和公钥，私钥仅对自己可见，然后交换公钥，
 * 并根据自己的私钥和对方的公钥，生成最终的密钥secretKey，
 * DH算法通过数学定律保证了双方各自计算出的secretKey是相同的
 *
 * @link {https://www.cnblogs.com/HKUI/p/DH.html}
 */
public class DhUtils {

    /**
     * 交换密钥的算法
     */
    private static final String ALGORITHM = "DH";

    /**
     * 数据加密的算法
     */
    public static final String DEFAULT_ALGORITHM = "DES";

    /**
     * This must be a multiple of 64, ranging from 512 to 1024 (inclusive), or 2048. The default keysize is 1024.
     */
    private static final int DEFAULT_KEY_SIZE = 1024;

    public static InnerKey generateSenderKey() throws NoSuchAlgorithmException {
        return generateSenderKey(DEFAULT_KEY_SIZE);
    }

    /**
     * 生成发送者密钥对
     *
     * @return
     * @throws Exception
     */
    public static InnerKey generateSenderKey(int keySize)
            throws NoSuchAlgorithmException {
        KeyPairGenerator senderGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        senderGenerator.initialize(keySize);
        KeyPair senderKeyPair = senderGenerator.generateKeyPair();
        return InnerKey.builder()
                .publicKey(senderKeyPair.getPublic().getEncoded())
                .privateKey(senderKeyPair.getPrivate().getEncoded())
                .build();
    }

    /**
     * 根据发送者公钥；生成接收者密钥对
     *
     * @param senderPublicKeyBytes 发送者公钥
     * @return
     * @throws Exception
     */
    public static InnerKey generateReceiverKey(byte[] senderPublicKeyBytes)
            throws NoSuchAlgorithmException, InvalidKeySpecException, InvalidAlgorithmParameterException {
        // 根据发送方公钥 初始化接收方密钥
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(senderPublicKeyBytes);
        PublicKey senderPublicKey = keyFactory.generatePublic(x509EncodedKeySpec);

        DHParameterSpec dhParameterSpec = ((DHPublicKey) senderPublicKey).getParams();
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        keyPairGenerator.initialize(dhParameterSpec);
        KeyPair receiverKeyPair = keyPairGenerator.generateKeyPair();
        return InnerKey.builder()
                .publicKey(receiverKeyPair.getPublic().getEncoded())
                .privateKey(receiverKeyPair.getPrivate().getEncoded())
                .build();
    }

    /**
     * 计算密钥
     *
     * @param publicKeyBytes  公钥
     * @param privateKeyBytes 私钥
     * @param algorithm       加密算法
     * @return
     * @throws Exception
     */
    public static SecretKey computeSecretKey(byte[] publicKeyBytes, byte[] privateKeyBytes, String algorithm)
            throws NoSuchAlgorithmException, InvalidKeySpecException, InvalidKeyException {
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);

        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(publicKeyBytes);
        PublicKey publicKey = keyFactory.generatePublic(x509EncodedKeySpec);

        PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(privateKeyBytes);
        PrivateKey privateKey = keyFactory.generatePrivate(pkcs8KeySpec);

        KeyAgreement keyAgreement = KeyAgreement.getInstance(keyFactory.getAlgorithm());
        keyAgreement.init(privateKey);
        keyAgreement.doPhase(publicKey, true);
        SecretKey secretKey = keyAgreement.generateSecret(algorithm);
        return secretKey;
    }

    public static byte[] encrypt(byte[] receiverPublicKey, byte[] senderPrivateKey, byte[] data)
            throws NoSuchAlgorithmException, InvalidKeyException, InvalidKeySpecException, NoSuchPaddingException,
            BadPaddingException, IllegalBlockSizeException {
        return encrypt(receiverPublicKey, senderPrivateKey, data, DEFAULT_ALGORITHM);
    }

    /**
     * 发送者加密
     *
     * @param receiverPublicKey 接收者公钥
     * @param senderPrivateKey  发送者私钥
     * @param data              加密数据
     * @return
     * @throws Exception
     */
    public static byte[] encrypt(byte[] receiverPublicKey, byte[] senderPrivateKey, byte[] data, String algorithm)
            throws NoSuchAlgorithmException, InvalidKeyException, InvalidKeySpecException, NoSuchPaddingException,
            BadPaddingException, IllegalBlockSizeException {
        SecretKey secretKey = computeSecretKey(receiverPublicKey, senderPrivateKey, algorithm);
        Cipher cipher = Cipher.getInstance(algorithm);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        byte[] bytes = cipher.doFinal(data);
        return bytes;
    }

    public static byte[] decrypt(byte[] senderPublicKey, byte[] receiverPrivateKey, byte[] data)
            throws NoSuchAlgorithmException, InvalidKeyException, InvalidKeySpecException, NoSuchPaddingException,
            BadPaddingException, IllegalBlockSizeException {
        return decrypt(senderPublicKey, receiverPrivateKey, data, DEFAULT_ALGORITHM);
    }

    /**
     * 接收者解密
     *
     * @param senderPublicKey    发送者公钥
     * @param receiverPrivateKey 接受者私钥
     * @param data               解密数据
     * @return
     * @throws Exception
     */
    public static byte[] decrypt(byte[] senderPublicKey, byte[] receiverPrivateKey, byte[] data, String algorithm)
            throws NoSuchAlgorithmException, InvalidKeyException, InvalidKeySpecException, NoSuchPaddingException,
            BadPaddingException, IllegalBlockSizeException {
        SecretKey secretKey = computeSecretKey(senderPublicKey, receiverPrivateKey, algorithm);
        Cipher cipher = Cipher.getInstance(algorithm);
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] bytes = cipher.doFinal(data);
        return bytes;
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
package crypto.dh2;

import org.apache.commons.codec.binary.Base64;

import javax.crypto.SecretKey;
import java.util.Objects;

/**
 * 添加 VM 运行参数： -Djdk.crypto.KeyAgreement.legacyKDF=true
 */
public class DhUtilsTest {

    public static void main(String[] args) throws Exception {
        String text = "DH算法是一个密钥协商算法，双方最终协商出一个共同的密钥，而这个密钥不会通过网络传输";
        DhUtils.InnerKey senderKey = DhUtils.generateSenderKey();
        System.out.println("发送者公钥：" + Base64.encodeBase64String(senderKey.getPublicKey()));
        System.out.println("发送者私钥：" + Base64.encodeBase64String(senderKey.getPrivateKey()));

        DhUtils.InnerKey receiverKey = DhUtils.generateReceiverKey(senderKey.getPublicKey());
        System.out.println("接收者公钥：" + Base64.encodeBase64String(receiverKey.getPublicKey()));
        System.out.println("接收者私钥：" + Base64.encodeBase64String(receiverKey.getPrivateKey()));

        String algorithm = "DES";
        SecretKey senderSecretKey = DhUtils.computeSecretKey(receiverKey.getPublicKey(), senderKey.getPrivateKey(), algorithm);
        SecretKey receiverSecretKey = DhUtils.computeSecretKey(senderKey.getPublicKey(), receiverKey.getPrivateKey(), algorithm);
        if (Objects.equals(senderSecretKey, receiverSecretKey)) {
            System.out.println("密钥生成成功");
        } else {
            throw new RuntimeException("密钥生成失败");
        }

        // 加密
        byte[] bytes = DhUtils.encrypt(receiverKey.getPublicKey(), senderKey.getPrivateKey(), text.getBytes(), algorithm);
        System.out.println("密文：" + Base64.encodeBase64String(bytes));

        // 解密
        byte[] result = DhUtils.decrypt(senderKey.getPublicKey(), receiverKey.getPrivateKey(), bytes, algorithm);
        System.out.println("明文：" + new String(result));
    }

}

```

[code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/dh2)

## 补充知识
[理解 Deffie-Hellman 密钥交换算法](http://wsfdl.com/algorithm/2016/02/04/%E7%90%86%E8%A7%A3Diffie-Hellman%E5%AF%86%E9%92%A5%E4%BA%A4%E6%8D%A2%E7%AE%97%E6%B3%95.html) 中关于`求模公式`的说明

假设 q 为素数，对于正整数 a,x,y，有：
`(a^x mod p)^y mod p = a^(xy) mod p`