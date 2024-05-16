---
layout:     post
title:      RSA 跨语言使用
subtitle:   非对称加密
date:       2022-10-27
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - Cryptography
---


## java 版本

1. 获取密钥

```java

import java.math.BigInteger;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

public class GetKey {

    public static final String ALGORITHM = "RSA";

    public static void main(String[] args) throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(ALGORITHM);
        keyPairGenerator.initialize(512);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        RSAPublicKey rsaPublicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey rsaPrivateKey = (RSAPrivateKey) keyPair.getPrivate();

        // pubKey = (n,e) ; priKey = (n,d)
        BigInteger n = rsaPublicKey.getModulus();
        BigInteger e = rsaPublicKey.getPublicExponent();
        BigInteger d = rsaPrivateKey.getPrivateExponent();
        System.out.println("n: " + n);
        System.out.println("e: " + e);
        System.out.println("d: " + d);
    }
}

```

```
n: 10024046402334537665438324593256381027774491568777420349437153155855551494930992968579095115335998035992292512366201628140916921363490577352034455478837127
e: 65537
d: 3547885627180858685742517620049361647927996497083495467072710142411544203677535574073374414767899187836766831505613082822722655998801933606944964650191553
```

2. 加解密

```java

import org.apache.commons.codec.binary.Hex;

import javax.crypto.Cipher;
import java.math.BigInteger;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.RSAPrivateKeySpec;
import java.security.spec.RSAPublicKeySpec;

public class RsaUtilByNed {
    public static final String ALGORITHM = "RSA";

    public static void main(String[] args) throws Exception {
        // pubKey = (n,e)
        // priKey = (n,d)
        BigInteger n = new BigInteger("10024046402334537665438324593256381027774491568777420349437153155855551494930992968579095115335998035992292512366201628140916921363490577352034455478837127");
        BigInteger e = new BigInteger("65537");
        BigInteger d = new BigInteger("3547885627180858685742517620049361647927996497083495467072710142411544203677535574073374414767899187836766831505613082822722655998801933606944964650191553");

        String text = "Hello 世界";
        // 加密
        byte[] encryptedBytes = encrypt(n, e, text.getBytes());
        String encrypted = Hex.encodeHexString(encryptedBytes);
        System.out.println(encrypted);
        // 解密
        encrypted = "202983f28f436601b9bf8f36250c2ba17abf88cdc8a81614d6f7c6cc9aff7e1136cfbf6744330b2b67464f4e1f866c364198d6e89eb2660c76e1546c3c9ae34a";
        byte[] data = decrypt(n, d, Hex.decodeHex(encrypted));
        System.out.println(new String(data));
    }

    public static byte[] encrypt(BigInteger modulus, BigInteger publicExponent, byte[] data) throws Exception {
        PublicKey publicKey = generateRSAPublicKey(modulus.toByteArray(), publicExponent.toByteArray());
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(BigInteger modulus, BigInteger privateExponent, byte[] data) throws Exception {
        PrivateKey privateKey = generateRSAPrivateKey(modulus.toByteArray(), privateExponent.toByteArray());
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        return cipher.doFinal(data);
    }

    /**
     * 根据给定的系数和专用指数构造一个RSA专用的公钥对象。
     *
     * @param modulus        系数。
     * @param publicExponent 专用指数。
     * @return RSA专用公钥对象。
     */
    public static RSAPublicKey generateRSAPublicKey(byte[] modulus, byte[] publicExponent) throws Exception {
        RSAPublicKeySpec publicKeySpec = new RSAPublicKeySpec(new BigInteger(modulus), new BigInteger(publicExponent));
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);
        return (RSAPublicKey) keyFactory.generatePublic(publicKeySpec);
    }

    /**
     * 根据给定的系数和专用指数构造一个RSA专用的私钥对象。
     *
     * @param modulus         系数。
     * @param privateExponent 专用指数。
     * @return RSA专用私钥对象。
     */
    public static RSAPrivateKey generateRSAPrivateKey(byte[] modulus, byte[] privateExponent) throws Exception {
        RSAPrivateKeySpec privateKeySpec = new RSAPrivateKeySpec(new BigInteger(modulus), new BigInteger(privateExponent));
        KeyFactory keyFactory = KeyFactory.getInstance(ALGORITHM);
        return (RSAPrivateKey) keyFactory.generatePrivate(privateKeySpec);
    }
}
```

## Python 版本

1. 获取密钥

```python
import rsa


def main():
    (pubkey, priKey) = rsa.newkeys(512)

    # pubKey = (n,e)
    n = pubkey.n
    e = pubkey.e
    # priKey = (n,d)
    d = priKey.d

    print(n)
    print(e)
    print(d)


if __name__ == '__main__':
    main()

```

2. 加解密

```python
import random

import rsa


def getpq(n: int, e: int, d: int):
    p = 1
    q = 1
    while p == 1 and q == 1:
        k = d * e - 1
        g = random.randint(0, n)
        while p == 1 and q == 1 and k % 2 == 0:
            k //= 2
            y = pow(g, k, n)
            if y != 1 and gcd(y - 1, n) > 1:
                p = gcd(y - 1, n)
                q = n // p
    return p, q


def gcd(a: int, b: int):
    if a < b:
        a, b = b, a
    while b != 0:
        temp = a % b
        a = b
        b = temp
    return a


# pubKey = (n,e) ; priKey = (n,d)
n = 10024046402334537665438324593256381027774491568777420349437153155855551494930992968579095115335998035992292512366201628140916921363490577352034455478837127
e = 65537
d = 3547885627180858685742517620049361647927996497083495467072710142411544203677535574073374414767899187836766831505613082822722655998801933606944964650191553


def main():
    p, q = getpq(n, e, d)
    pub_key = rsa.key.PublicKey(n=n, e=e)
    pri_key = rsa.key.PrivateKey(n=n, e=e, d=d, p=p, q=q)

    data = '你好 python '.encode('utf-8')
    encrypt_res = rsa.encrypt(data, pub_key)
    encrypt_res_hex = encrypt_res.hex()
    print(encrypt_res_hex)

    encrypt_res_hex = '81ec8aec8eed3752c6f34100bcba1ed0478d4d9348ca474a4cfd25870a974d94620f2c2096e01409b7d86298040bb945e8e1614607ebfd37c38b0cdf6bbc1b1b'
    bs = rsa.decrypt(bytes.fromhex(encrypt_res_hex), pri_key)
    print(bs.decode(encoding='utf-8'))


if __name__ == '__main__':
    main()

```

## Golang 版本

1. 获取密钥

```go
func getKey() {
	// 生成密钥
	privateKey, err := rsa.GenerateKey(rand.Reader, 512+64*0)
	if err != nil {
		panic(err)
	}
	publicKey := privateKey.PublicKey

	// n e d
	n := publicKey.N
	e := publicKey.E
	d := privateKey.D
	fmt.Println(n)
	fmt.Println(e)
	fmt.Println(d)
}
```

2. 加解密

```go
package main

import (
	"bytes"
	"crypto/rand"
	"crypto/rsa"
	"fmt"
	"math/big"
	"strconv"
)

// bytes to hex string
func bytesToHexString(b []byte) string {
	var buf bytes.Buffer
	for _, v := range b {
		t := strconv.FormatInt(int64(v), 16)
		if len(t) > 1 {
			buf.WriteString(t)
		} else {
			buf.WriteString("0" + t)
		}
	}
	return buf.String()
}

// hex string to bytes
func hexStringToBytes(s string) []byte {
	bs := make([]byte, 0)
	for i := 0; i < len(s); i = i + 2 {
		b, _ := strconv.ParseInt(s[i:i+2], 16, 16)
		bs = append(bs, byte(b))
	}
	return bs
}

func fromBase10(base10 string) *big.Int {
	i, ok := new(big.Int).SetString(base10, 10)
	if !ok {
		panic("bad number: " + base10)
	}
	return i
}

func encryptTest(publicKey rsa.PublicKey) {

	text := "你好，GOLANG" //需要加密的字符串
	// encryptedBytes, err := rsa.EncryptOAEP(sha256.New(), rand.Reader, &publicKey, []byte(text), nil)
	encryptedBytes, err := rsa.EncryptPKCS1v15(rand.Reader, &publicKey, []byte(text))
	if err != nil {
		panic(err)
	}
	fmt.Println(bytesToHexString(encryptedBytes))
}

func decryptTest(privateKey *rsa.PrivateKey) {
	//decryptedBytes, err := privateKey.Decrypt(nil, encryptedBytes, &rsa.OAEPOptions{Hash: crypto.SHA256})
	minwen := "8cb43410ddb49fec8c1b87c63715409c77f3d76e67edf3b66b57515632aa0a3a6f6264dbe54a1d5633143780f0c8ddc2d58eff2e260c17a204247277a5dacb18"
	encryptedBytes2 := hexStringToBytes(minwen)
	decryptedBytes, err := rsa.DecryptPKCS1v15(rand.Reader, privateKey, encryptedBytes2)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(decryptedBytes))
}

func main() {

	privateKey := &rsa.PrivateKey{
		PublicKey: rsa.PublicKey{
			N: fromBase10("10024046402334537665438324593256381027774491568777420349437153155855551494930992968579095115335998035992292512366201628140916921363490577352034455478837127"),
			E: 65537,
		},
		D: fromBase10("3547885627180858685742517620049361647927996497083495467072710142411544203677535574073374414767899187836766831505613082822722655998801933606944964650191553"),
		Primes: []*big.Int{
			fromBase10("16775196964030542637"),
			fromBase10("17328218193455850539"),
		},
	}

	// 加密
	publicKey := privateKey.PublicKey
	encryptTest(publicKey)

	// 解密
	decryptTest(privateKey)
}

```

## 总结

1. 三种语言是使用了一套密钥的
2. 每次加密之后得到的密文是不一样的，但解密的结果都是一样的
3. 后面考虑使用其他语言也试试