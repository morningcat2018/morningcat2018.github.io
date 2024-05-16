---
layout:     post
title:      AES算法
subtitle:   高级加密标准
date:       2022-03-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---

## 概念

高级加密标准（`Advanced Encryption Standard`：AES），又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。

这个标准用来替代原先的DES。
该算法为比利时密码学家Joan Daemen和Vincent Rijmen所设计，结合两位作者的名字，以Rijndael为名投稿高级加密标准的甄选流程。

严格地说，AES和Rijndael加密法并不完全一样（虽然在实际应用中两者可以互换），因为Rijndael加密法可以支持更大范围的区块和密钥长度：AES的`区块长度`固定为128比特，`密钥长度`则可以是128，192或256比特；而Rijndael使用的`密钥和区块长度`均可以是128，192或256比特。加密过程中使用的密钥是由Rijndael密钥生成方案产生。

大多数AES计算是在一个特别的`有限域`完成的。

## AES加密过程

AES加密过程是在一个4×4的字节矩阵上运作，这个矩阵又称为“体（state）”，其初值就是一个明文区块（矩阵中一个元素大小就是明文区块中的一个Byte）。
（Rijndael加密法因支持更大的区块，其矩阵的“列数（Row number）”可视情况增加）加密时，各轮AES加密循环（除最后一轮外）均包含4个步骤：

1. AddRoundKey 步骤

矩阵中的每一个字节都与该次`回合密钥（round key）`做`XOR运算`；每个子密钥由密钥生成方案产生。

`回合密钥`将会与原矩阵合并。在每次的加密循环中，都会由主密钥产生一把回合密钥（透过Rijndael密钥生成方案产生），这把密钥大小会跟原矩阵一样，以与原矩阵中每个对应的字节作异或（⊕）加法。

2. SubBytes 步骤

透过一个非线性的替换函数，用`查找表`的方式把每个字节替换成对应的字节。

在SubBytes步骤中，矩阵中的各字节透过一个8位的`S-box`进行转换。这个步骤提供了`加密法`非线性的变换能力。
S-box与GF(2^8)上的乘法`反元素`有关，已知具有良好的`非线性`特性。为了避免简单代数性质的攻击，S-box结合了乘法反元素及一个可逆的`仿射变换`矩阵建构而成。
此外在建构S-box时，刻意避开了`不动点`与`反不动点`，即以S-box替换字节的结果会相当于错排的结果。Rijndael S-box条目有针对S-box的详细描述。

3. ShiftRows步骤

将矩阵中的每个横列进行循环式移位。

ShiftRows描述矩阵的行操作。在此步骤中，每一行都向左循环位移某个`偏移量`。在AES中（区块大小128位），第一行维持不变，第二行里的每个字节都向左循环移动一格。
同理，第三行及第四行向左循环位移的偏移量就分别是2和3。128位和192比特的区块在此步骤的循环位移的模式相同。
经过ShiftRows之后，矩阵中每一竖列，都是由输入矩阵中的每个不同列中的元素组成。
Rijndael算法的版本中，偏移量和AES有少许不同；对于长度256比特的区块，第一行仍然维持不变，第二行、第三行、第四行的偏移量分别是1字节、2字节、3字节。
除此之外，ShiftRows操作步骤在Rijndael和AES中完全相同。

4. MixColumns步骤

为了充分混合矩阵中各个直行的操作。这个步骤使用`线性转换`来混合每内联的四个字节。最后一个加密循环中省略MixColumns步骤，而以另一个AddRoundKey取代。

每一列的四个字节透过线性变换互相结合。每一列的四个元素分别当作 1 , x , x^2 , x^3 的系数，合并即为 GF(2^8) 中的一个多项式，接着将此多项式和一个固定的多项式 c(x)=3x^3+x^2+x+2 在模 x^4+1 下相乘。此步骤亦可视为 `Rijndael 有限域`之下的矩阵乘法。MixColumns 函数接受4个字节的输入，输出4个字节，每一个输入的字节都会对输出的四个字节造成影响。因此 ShiftRows 和 MixColumns 两步骤为这个密码系统提供了扩散性。

## AES支持的模式

- ECB模式（The Electronic Codebook Mode）
- CBC模式（The Cipher Block Chaining Mode）
- CTR模式（The Counter Mode）
- GCM模式（The Galois/Counter Mode）
- CFB模式（The Cipher Feedback Mode）
- OFB模式（The Output Feedback Mode）

![图片](/png/cryptography/aes_mod.png)

1. 根据加密方式的不同，简单分为块加密模式与流加密模式: 块加密模式最为常见同时在工程化中使用最为普遍的是CBC模式, 流加密模式最具代表性的是GCM模式;
2. ECB 和 CBC 需要强制对齐, 不支持 NoPadding
3. 只有 ECB 模式不需要 iv ,其他模式均需要添加 iv
4. GCM 模式在java语言的官方库并没有提供, 需要使用bc库
5. ECB模式是不安全的，不建议在工程实践中使用

## java

1.  jdk 实现

```java
package crypto.aes;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.Key;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.util.Arrays;

/**
 * 对称加密 AES算法 jdk实现
 */
public class AesUtils {

    private static final String ALGORITHM = "AES";

    private static final byte[] DEFAULT_IV = new byte[16];

    public enum MODE {
        ECB, CBC, PCBC, CTR, CFB, OFB, GCM
    }

    public enum PADDING {
        NoPadding, PKCS5Padding, PKCS7Padding, ISO10126Padding
    }

    public static byte[] initKey() throws Exception {
        return initKey(128);
    }

    /**
     * @param keySIze 128 192 256
     * @return 密钥
     * @throws NoSuchAlgorithmException
     */
    public static byte[] initKey(int keySIze) throws Exception {
        if (keySIze != 128 && keySIze != 192 && keySIze != 256) {
            throw new RuntimeException("invalid key size, valid size is 128 or 192 or 256");
        }
        KeyGenerator keyGenerator = KeyGenerator.getInstance(ALGORITHM);
        SecureRandom secureRandom = new SecureRandom();
        keyGenerator.init(keySIze, secureRandom);// 密钥长度
        SecretKey secretKey = keyGenerator.generateKey();
        return secretKey.getEncoded();
    }

    protected static Key buildKey(byte[] key) {
        return new SecretKeySpec(key, ALGORITHM);
    }

    public static byte[] encrypt(byte[] key, byte[] data) throws Exception {
        return encrypt(key, data, MODE.CBC.name(), PADDING.PKCS5Padding.name(), DEFAULT_IV);
    }

    /**
     * @param key  密钥
     * @param data 需要加密的数据
     * @param mode 工作模式
     * @param pad  填充方式
     * @return 加密后的数据
     * @throws Exception
     */
    public static byte[] encrypt(byte[] key, byte[] data, String mode, String pad, byte[] iv_) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad));
        if (MODE.ECB.name().equals(mode)) {
            cipher.init(Cipher.ENCRYPT_MODE, buildKey(key));
        } else {
            IvParameterSpec iv = new IvParameterSpec(iv_);
            cipher.init(Cipher.ENCRYPT_MODE, buildKey(key), iv);
        }
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(byte[] key, byte[] data) throws Exception {
        return decrypt(key, data, MODE.CBC.name(), PADDING.PKCS5Padding.name(), DEFAULT_IV);
    }

    /**
     * @param key  密钥
     * @param data 需要解密的数据
     * @param mode 工作模式
     * @param pad  填充方式
     * @return 解密后的数据
     * @throws Exception
     */
    public static byte[] decrypt(byte[] key, byte[] data, String mode, String pad, byte[] iv_) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad));
        if (MODE.ECB.name().equals(mode)) {
            cipher.init(Cipher.DECRYPT_MODE, buildKey(key));
        } else {
            IvParameterSpec iv = new IvParameterSpec(iv_);
            cipher.init(Cipher.DECRYPT_MODE, buildKey(key), iv);
        }
        return cipher.doFinal(data);
    }

    public static byte[] buildIv(byte[] iv_) {
        if (iv_ == null) {
            return DEFAULT_IV;
        }
        if (iv_.length < 16) {
            return Arrays.copyOf(iv_, 16);
        }
        return Arrays.copyOfRange(iv_, 0, 16);
    }


}

```

```java
package crypto.aes;

import org.apache.commons.codec.binary.Base64;
import org.junit.Test;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2024/4/8 22:19
 */
public class TestAesUtils {

    @Test
    public void testKey() throws Exception {
        byte[] key1 = AesUtils.initKey(128);
        System.out.println("128密钥：" + Base64.encodeBase64String(key1));
        byte[] key2 = AesUtils.initKey(192);
        System.out.println("192密钥：" + Base64.encodeBase64String(key2));
        byte[] key3 = AesUtils.initKey(256);
        System.out.println("256密钥：" + Base64.encodeBase64String(key3));
    }

    public static final String key128 = "DppKP2MHu75V09kVX9P7qg==";
    public static final String key192 = "pMX5Zq8e6UiURseN2fBMvHWj4RduI2YI";
    public static final String key256 = "Wb/ZsZjhuCw1oH0xrRfQ4DlvnEU3iTAC4jB8jFDaSLU=";

    public static final String plainText = "世界人民大团结万岁";

    public static final byte[] IV = "hello world".getBytes();

    @Test
    public void testECB_encrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.ECB.name();
        String pad = AesUtils.PADDING.PKCS5Padding.name();
        System.out.println("密钥：" + key128);

        byte[] data = AesUtils.encrypt(key, plainText.getBytes("UTF-8"), mode, pad, null);
        System.out.println("密文：" + Base64.encodeBase64String(data));

    }

    @Test
    public void testECB_decrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.ECB.name();
        String pad = AesUtils.PADDING.PKCS5Padding.name();
        System.out.println("密钥：" + key128);

        String cipherText = "Cpd+1DNuqfIhYss+JJESVm3sbQli8KD+m5wBbT8Q/+4=";
        byte[] data = Base64.decodeBase64(cipherText);
        byte[] result = AesUtils.decrypt(key, data, mode, pad, null);
        System.out.println("明文：" + new String(result));
    }

    @Test
    public void testECB_encrypt2() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.ECB.name();
        String pad = AesUtils.PADDING.ISO10126Padding.name();
        System.out.println("密钥：" + key128);

        byte[] data = AesUtils.encrypt(key, plainText.getBytes("UTF-8"), mode, pad, null);
        System.out.println("密文：" + Base64.encodeBase64String(data));

    }

    @Test
    public void testECB_decrypt2() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.ECB.name();
        String pad = AesUtils.PADDING.ISO10126Padding.name();
        System.out.println("密钥：" + key128);

        String cipherText = "Cpd+1DNuqfIhYss+JJESVpyeiLEbtE1j7egrUM28YfM=";
        byte[] data = Base64.decodeBase64(cipherText);
        byte[] result = AesUtils.decrypt(key, data, mode, pad, null);
        System.out.println("明文：" + new String(result));
    }

    @Test
    public void testCBC_encrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.CBC.name();
        String pad = AesUtils.PADDING.PKCS5Padding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        byte[] data = AesUtils.encrypt(key, plainText.getBytes("UTF-8"), mode, pad, iv);
        System.out.println("密文：" + Base64.encodeBase64String(data));

    }

    @Test
    public void testCBC_decrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.CBC.name();
        String pad = AesUtils.PADDING.PKCS5Padding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        String cipherText = "qqKRgH2nogqE3kerhsgxFE/5DCEYXDYivlFb4SLhfLU=";
        byte[] data = Base64.decodeBase64(cipherText);
        byte[] result = AesUtils.decrypt(key, data, mode, pad, iv);
        System.out.println("明文：" + new String(result));
    }

    @Test
    public void testCTR_encrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.CTR.name();
        String pad = AesUtils.PADDING.NoPadding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        byte[] data = AesUtils.encrypt(key, plainText.getBytes("UTF-8"), mode, pad, iv);
        System.out.println("密文：" + Base64.encodeBase64String(data));

    }

    @Test
    public void testCTR_decrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.CTR.name();
        String pad = AesUtils.PADDING.NoPadding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        String cipherText = "h4cZP6C7G0Md1sUSD9u51WAs+p2S3fTtmYz/";
        byte[] data = Base64.decodeBase64(cipherText);
        byte[] result = AesUtils.decrypt(key, data, mode, pad, iv);
        System.out.println("明文：" + new String(result));
    }

    @Test
    public void testCFB_encrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.CFB.name();
        String pad = AesUtils.PADDING.PKCS5Padding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        byte[] cipherBytes = AesUtils.encrypt(key, plainText.getBytes("UTF-8"), mode, pad, iv);
        System.out.println("密文：" + Base64.encodeBase64String(cipherBytes));

    }

    @Test
    public void testCFB_decrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.CFB.name();
        String pad = AesUtils.PADDING.PKCS5Padding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        String cipherText = "h4cZP6C7G0Md1sUSD9u51TC/ATsFQAeB6Qs9Ad/rkqA=";
        byte[] data = Base64.decodeBase64(cipherText);
        byte[] result = AesUtils.decrypt(key, data, mode, pad, iv);
        System.out.println("明文：" + new String(result));
    }

}

```

2. bc 实现

```java
package crypto.aes;

import org.apache.commons.codec.binary.Base64;
import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.*;
import javax.crypto.spec.IvParameterSpec;
import java.security.*;

/**
 * 对称加密 AES算法 bouncycastle实现
 */
public class AesByBcUtils extends AesUtils {

    static {
        Security.addProvider(new BouncyCastleProvider());
    }

    private static final String ALGORITHM = "AES";

    private static final String PROVIDER = "BC";

    public static byte[] initKey() throws Exception {
        return initKey(128);
    }

    /**
     * 初始化密钥
     *
     * @param keySIze 128 192 256
     */
    public static byte[] initKey(int keySIze) throws Exception {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(ALGORITHM, PROVIDER);
        keyGenerator.init(keySIze);// 密钥长度
        SecretKey secretKey = keyGenerator.generateKey();
        return secretKey.getEncoded();
    }

    public static byte[] encrypt(byte[] key, byte[] data, String mode, String pad, byte[] iv_) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad), PROVIDER);
        if (MODE.ECB.name().equals(mode)) {
            cipher.init(Cipher.ENCRYPT_MODE, buildKey(key));
        } else {
            IvParameterSpec iv = new IvParameterSpec(iv_);
            cipher.init(Cipher.ENCRYPT_MODE, buildKey(key), iv);
        }
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(byte[] key, byte[] data, String mode, String pad, byte[] iv_) throws Exception {
        Cipher cipher = Cipher.getInstance(String.format("%s/%s/%s", ALGORITHM, mode, pad), PROVIDER);
        if (MODE.ECB.name().equals(mode)) {
            cipher.init(Cipher.DECRYPT_MODE, buildKey(key));
        } else {
            IvParameterSpec iv = new IvParameterSpec(iv_);
            cipher.init(Cipher.DECRYPT_MODE, buildKey(key), iv);
        }
        return cipher.doFinal(data);
    }

}

```

```java
package crypto.aes;

import org.apache.commons.codec.binary.Base64;
import org.junit.Test;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2024/4/8 22:32
 */
public class TestAesByBcUtils {

    @Test
    public void testKey() throws Exception {
        byte[] key1 = AesByBcUtils.initKey(128);
        System.out.println("128密钥：" + Base64.encodeBase64String(key1));
        byte[] key2 = AesByBcUtils.initKey(192);
        System.out.println("192密钥：" + Base64.encodeBase64String(key2));
        byte[] key3 = AesByBcUtils.initKey(256);
        System.out.println("256密钥：" + Base64.encodeBase64String(key3));
    }

    public static final String key128 = "w/9fq9evB8/UQvcTMzNNiw==";
    public static final String key192 = "6ojhHJL4ZJ4rMK7wIQWTn+JR44p3JTWP";
    public static final String key256 = "KPsj/EjIigZavN6KtptIjtLV27/dLsE3LOIKBcomKMY=";

    public static final String plainText = "世界人民大团结万岁";

    public static final byte[] IV = "hello world".getBytes();

    @Test
    public void testCBC_encrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.GCM.name();
        String pad = AesUtils.PADDING.NoPadding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        byte[] data = AesByBcUtils.encrypt(key, plainText.getBytes("UTF-8"), mode, pad, iv);
        System.out.println("密文：" + Base64.encodeBase64String(data));

    }

    @Test
    public void testCBC_decrypt() throws Exception {
        byte[] key = Base64.decodeBase64(key128);
        String mode = AesUtils.MODE.GCM.name();
        String pad = AesUtils.PADDING.NoPadding.name();
        byte[] iv = AesUtils.buildIv(IV);
        System.out.println("密钥：" + key128);
        System.out.println("iv:" + Base64.encodeBase64String(iv));

        String cipherText = "uWepaOrfD+G41mDxFdEJh0+EZ12AnwQoLkcyqUT7RrM9THJ08DjA+Pv8lQ==";
        byte[] data = Base64.decodeBase64(cipherText);
        byte[] result = AesByBcUtils.decrypt(key, data, mode, pad, iv);
        System.out.println("明文：" + new String(result));
    }


}

```

3. 依赖库

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.70</version>
</dependency>
```

[AES code](https://gitee.com/mengzhang6/java-read-sources-sample/tree/jdk8/src/main/java/crypto/aes)

## python

> pip3 install crypto

```py
import base64
import os

from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

BS = AES.block_size
key_ = "DppKP2MHu75V09kVX9P7qg=="
iv_ = "aGVsbG8gd29ybGQAAAAAAA=="
plaintext_ = "世界人民大团结万岁"


class AES_Util(object):
    @staticmethod
    def init_key(key_size: int):
        """

        :param key_size: 128 192 256
        :return:
        """
        key = os.urandom(key_size // 8)
        return base64.b64encode(key)

    @staticmethod
    def init_iv():
        key = os.urandom(128 // 8)
        return base64.b64encode(key)

    @staticmethod
    def cbc_encrypt(plaintext: str, key_str: str):
        """
        AES-CBC 加密
        key 必须是 16(AES-128)、24(AES-192) 或 32(AES-256) 字节的 AES 密钥；
        初始化向量 iv 为随机的 16 位字符串 (必须是16位)，
        解密需要用到这个相同的 iv，因此将它包含在密文的开头。
        """
        key = base64.b64decode(key_str)
        iv = base64.b64decode(iv_)
        cipher = AES.new(key, AES.MODE_CBC, iv)
        ciphertext = cipher.encrypt(pad(plaintext.encode("utf-8"), BS))
        return base64.b64encode(ciphertext)

    @staticmethod
    def cbc_decrypt(ciphertext: str, key_str: str):
        """
        AES-CBC 解密
        密文的前 16 个字节为 iv
        """
        key = base64.b64decode(key_str)
        iv = base64.b64decode(iv_)
        cipher_bytes = base64.b64decode(ciphertext)
        cipher = AES.new(key, AES.MODE_CBC, iv)
        plain_bytes = cipher.decrypt(cipher_bytes)
        return unpad(plain_bytes, BS).decode("utf-8")

    @staticmethod
    def gcm_encrypt(plaintext: str, key_str: str):
        key = base64.b64decode(key_str)
        iv = base64.b64decode(iv_)
        cipher = AES.new(key, AES.MODE_GCM, iv)
        ciphertext = cipher.encrypt(pad(plaintext.encode("utf-8"), BS))
        return base64.b64encode(ciphertext)

    @staticmethod
    def gcm_decrypt(ciphertext: str, key_str: str):
        key = base64.b64decode(key_str)
        iv = base64.b64decode(iv_)
        cipher_bytes = base64.b64decode(ciphertext)
        cipher = AES.new(key, AES.MODE_GCM, iv)
        plain_bytes = cipher.decrypt(cipher_bytes)
        return unpad(plain_bytes, BS).decode("utf-8")


def init_key_test():
    print("128位密钥", AES_Util.init_key(128))


def init_iv_test():
    print("IV", AES_Util.init_iv())


if __name__ == '__main__':
    ciphertext = AES_Util.cbc_encrypt(plaintext_, key_)
    print("CBC密文:", ciphertext)  # b'qqKRgH2nogqE3kerhsgxFE/5DCEYXDYivlFb4SLhfLU='
    plaintext = AES_Util.cbc_decrypt(ciphertext, key_)
    print("明文:", plaintext)

    ciphertext = AES_Util.gcm_encrypt(plaintext_, key_)
    print("GCM密文:", ciphertext)  # b'TCFfCjtg48lxXYJQ56MrHuh2gCrtM9rRWNYJmAoIjhQ='
    plaintext = AES_Util.gcm_decrypt(ciphertext, key_)
    print("明文:", plaintext)

```

## nodejs

> npm install crypto

```js
const crypto = require('crypto');
const algorithm = 'aes-128-cbc';
const key = crypto.randomBytes(16);
const iv = crypto.randomBytes(16);
const plaintext = "世界人民大团结万岁"
console.log("key=", Buffer.from(key).toString('base64'));
console.log("iv=", Buffer.from(iv).toString('base64'));

// 加密器
const cipher = crypto.createCipheriv(algorithm, key, iv);
ciphertext_base64 = cipher.update(plaintext, 'utf8', 'base64');
ciphertext_base64 += cipher.final('base64');
console.log("ciphertext=", ciphertext_base64);

// 解密器
const decipher = crypto.createDecipheriv(algorithm, key, iv);
let text = decipher.update(ciphertext_base64, 'base64', 'utf8');
text += decipher.final(utf8);
console.log("plaintext=", text);

```




