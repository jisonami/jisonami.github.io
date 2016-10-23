---
layout: post
title:  "Java的加密解密"
date:   2016-10-18 23:14:54
categories: Security
tags: Java Security
---

* content
{:toc}

加密解密本身是一个复杂的领域，而Java通过JCE规范建立的一个通用的加解密模型，使得我们无需了解加解密的细节即可实现比较健壮的安全方案。

当然了，对于加密解密领域的一些概念还是需要了解的，比如对称密钥算法、非对称密钥算法、公钥、私钥等。

还有常见的加解密算法有哪些？以及通过Java通用的加解密模型是怎么实现的？本文会对这些内容做一个总结，包括对《Java加密解密的艺术》第二版一书的加解密部分总结以及相应的实践。

基于这些内容，我封装了一个加解密的类库[https://github.com/jisonami/crypto](https://github.com/jisonami/crypto)




### 一些概念

**Base64算法**：Base64是电子邮件传输算法，Base64编码映射表是A-Za-z0-9加上+-=一共64个字符构成的一个映射表，Base64编码就是将一段数据的二进制流使用Base64的编码映射表转换的过程，Base64解码就是根据Base64的编码映射表将Base64编码后的数据重新还原成原来数据的二进制流的过程

**加密**：将明文转换为密文的过程

**解密**：将密文转换为明文的过程

**密钥**：加密解密过程中使用到的编码映射表

**消息摘要算法**：消息摘要算法对于相同的数据通过散列函数都会得到一段相同的结果值，这个结果值无法逆向还原成原来的数据。有人把消息摘要算法说成是单向加密算法，其实不是。消息摘要算法常常跟非对称密钥算法结合实现数据签名和验证签名的操作。常用的消息摘要算法有MD系列和SHA系列等

**对称密钥算法**：加密和解密使用的都是同一个密钥，常用的对称密钥算法有DES、DESede、AES、RC2、RC4、Blowfish和IDEA等

**非对称密钥算法**：加密和解密使用不是同一个密钥，公钥加密数据需要使用私钥解密，私钥加密数据需要使用公钥解密，常用的非对称密钥算法有RSA、ELGAMAL等

**密钥协商算法**：密钥协商算法是从对称密钥算法向非对称密钥算法过度的一种算法，加解密双方各自生成密钥对，分别将公钥发送对对方，加解密双方各自使用自己的私钥和对方公钥生成本地对称密钥，然后使用本地对称密钥进行加解密，生成的本地对称密钥是相同的，常用的密钥协商算法有DH、ECDH等

**基于口令的加解密算法**：基于口令的加解密算法不再显示地使用密钥，加解密双方只需要知道一个简单的口令和一个安全随机数（称之为盐）就可以实现加解密了。实际上使用过的还是对称加解密的方法，实现方式是通过口令生成了对称密钥对象。常用的基于口令的加解密算法包括PBEWithMD5AndDES、PBEWithMD5AndTripleDES、PBEWithSHA1AndDESede、PBEWithSHA1AndRC2_40等

**加解密其实是编码转换或者编码替换等的过程，而这个转换的规则则保存在密钥中**

### Java加解密相关API

Java加解密相关的API可以概括为如下类图

![Java加解密相关的API类图](/images/Java_Security/Crypto/JavaCryptoClassDiagram.png)

Cipher：实现加密与解密

Key：密钥接口

PublicKey：公钥接口

PrivateKey：私钥接口

SecretKey：对称密钥接口

KeyPairGenerator：密钥对生成器

KeyPair：密钥对，一个密钥对对象包含一个成对公钥和私钥

KeyGenerator：对称密钥生成器

KeySpec：密钥规范材料

SecretKeySpec：对称密钥规范材料

X509EncodedKeySpec：公钥编码规范材料

PKCS8EncodedKeySpec：私钥编码规范材料

#### 密钥生成

密钥对生成

```java
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(this.keyAlgorithm);
keyPairGenerator.initialize(this.keySize);
KeyPair keyPair = keyPairGenerator.generateKeyPair();
```

获得公钥

```java
PublicKey publicKey = keyPair.getPublic();
```

获得私钥

```java
PrivateKey privateKey = keyPair.getPrivate();
```

对称密钥生成

```java
KeyGenerator keyGenerator = KeyGenerator.getInstance(this.keyAlgorithm);
keyGenerator.init(this.keySize);
SecretKey secretKey = keyGenerator.generateKey();
```

#### 密钥还原

公钥还原

```java
X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(key);
KeyFactory keyFactory = KeyFactory.getInstance(this.keyAlgorithm);
PublicKey publicKey = keyFactory.generatePublic(x509EncodedKeySpec);
```

私钥还原

```java
PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(key);
KeyFactory keyFactory = KeyFactory.getInstance(this.keyAlgorithm);
PrivateKey privateKey = keyFactory.generatePrivate(pkcs8EncodedKeySpec);
```

对称密钥还原

```java
SecretKey secretKey = new SecretKeySpec(key, this.keyAlgorithm);
```

或者

```java
SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance(this.keyAlgorithm);
SecretKey secretKey = secretKeyFactory.generateSecret(keySpec);
```

#### 加密解密

加密

```java
Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
cipher.init(Cipher.ENCRYPT_MODE, key);
byte[] encryptData = cipher.doFinal(data);
```

解密

```java
Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
cipher.init(Cipher.DECRYPT_MODE, key);
byte[] encryptData = cipher.doFinal(data);
```

### Java加解密实现

#### 对称密钥算法加解密通用实现

```java

import org.apache.commons.codec.binary.Base64;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.Key;

public class SymmetricCryptography {

	/**
	 * 密钥生成算法
	 */
	private String keyAlgorithm;

	/**
	 * 加解密算法/分组策略/填充方式
	 */
	private String cipherAlgorithm;

	/**
	 * 密钥长度
	 */
	private int keySize;

	public SymmetricCryptography(String keyAlgorithm, String cipherAlgorithm, int keySize) {
		this.keyAlgorithm = keyAlgorithm;
		this.cipherAlgorithm = cipherAlgorithm;
		this.keySize = keySize;
	}

	/**
	 * 生成一个密钥
	 * @return 密钥的二进制形式
     */
	public byte[] initKey() throws Exception{
		KeyGenerator keyGenerator = KeyGenerator.getInstance(this.keyAlgorithm);
		keyGenerator.init(this.keySize);
		SecretKey secretKey = keyGenerator.generateKey();
		return secretKey.getEncoded();
	}

	/**
	 * 转换密钥
	 * @param key 密钥的二进制形式
	 * @return Key 密钥对象
     */
	private Key toSecretKey(byte[] key) {
		return new SecretKeySpec(key, this.keyAlgorithm);
	}

	/**
	 * 加密操作
	 * @param data 需要加密的数据
	 * @param key 密钥的二进制形式
	 * @return 加密后的数据
	 */
	public byte[] encrypt(byte[] data, byte[] key) throws Exception{
		Key secretKey = this.toSecretKey(key);
		Cipher cipher = cipher = Cipher.getInstance(this.cipherAlgorithm);
		cipher.init(Cipher.ENCRYPT_MODE, secretKey);
		return cipher.doFinal(data);
	}

	/**
	 * 解密操作
	 * @param data 需要解密的数据
	 * @param key 密钥的二进制形式
	 * @return 解密后的数据
	 */
	public byte[] decrypt(byte[] data, byte[] key) throws Exception{
		Key secretKey = this.toSecretKey(key);
		Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
		cipher.init(Cipher.DECRYPT_MODE, secretKey);
		return cipher.doFinal(data);
	}

	/**
	 * 测试
	 * @param args
     */
	public static void main(String[] args) throws Exception{
		String data = "测试对称密钥算法加解密";
		System.out.println("加密前数据：" + data);
		// 其它对称密钥算法更改构造参数即可
		// 例如SymmetricCryptography symmetricCryptography = new SymmetricCryptography("DES", "DES/ECB/PKCS5Padding", 56);
		SymmetricCryptography symmetricCryptography = new SymmetricCryptography("AES", "AES/ECB/PKCS5Padding", 128);
		String key =  Base64.encodeBase64String(symmetricCryptography.initKey());
		System.out.println("对称密钥：" + key);
		String encryptData = Base64.encodeBase64String(symmetricCryptography.encrypt(data.getBytes(), Base64.decodeBase64(key)));
		System.out.println("加密后数据：" + encryptData);
		String decryptData = new String(symmetricCryptography.decrypt(Base64.decodeBase64(encryptData), Base64.decodeBase64(key)));
		System.out.println("解密后数据：" + decryptData);
	}

}

```

#### 非对称密钥算法加解密通用实现

```java

import org.apache.commons.codec.binary.Base64;

import javax.crypto.Cipher;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

public class NonSymmetricCryptography {

    private static final String PUBLIC_KEY = "PublicKey";

    private static final String PRIVATE_KEY = "PrivateKey";

    /**
     * 密钥生成算法
     */
    private String keyAlgorithm;

    /**
     * 加解密算法/分组策略/填充方式
     */
    private String cipherAlgorithm;

    /**
     * 密钥长度
     */
    private int keySize;

    public NonSymmetricCryptography(String keyAlgorithm, String cipherAlgorithm, int keySize) {
        this.keyAlgorithm = keyAlgorithm;
        this.cipherAlgorithm = cipherAlgorithm;
        this.keySize = keySize;
    }

    /**
     * 使用公钥加密操作
     * @param data 需要加密的数据
     * @param key 公钥的二进制形式
     * @return 加密后的数据
     */
    public byte[] encryptByPublicKey(byte[] data, byte[] key) throws Exception {
        Key k = toPublicKey(key);
        return this.encrypt(data, k);
    }

    /**
     * 使用公钥解密操作
     * @param data 需要解密的数据
     * @param key 公钥的二进制形式
     * @return 解密后的数据
     */
    public byte[] decryptByPublicKey(byte[] data, byte[] key) throws Exception {
        Key k = toPublicKey(key);
        return this.decrypt(data, k);
    }

    /**
     * 使用私钥加密操作
     * @param data 需要加密的数据
     * @param key 私钥的二进制形式
     * @return 加密后的数据
     */
    public byte[] encryptByPrivateKey(byte[] data, byte[] key) throws Exception {
        Key k = toPrivateKey(key);
        return this.encrypt(data, k);
    }

    /**
     * 使用私钥解密操作
     * @param data 需要解密的数据
     * @param key 私钥的二进制形式
     * @return 解密后的数据
     */
    public byte[] decryptByPrivateKey(byte[] data, byte[] key) throws Exception {
        Key k = toPrivateKey(key);
        return this.decrypt(data, k);
    }

    /**
     * 将公钥二进制形式转换成公钥对象
     * @param key 公钥的二进制形式
     * @return 公钥对象
     */
    private PublicKey toPublicKey(byte[] key) throws Exception{
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(key);
        KeyFactory keyFactory = KeyFactory.getInstance(this.keyAlgorithm);
        return keyFactory.generatePublic(x509EncodedKeySpec);
    }

    /**
     * 将私钥二进制形式转换成私钥对象
     * @param key 私钥的二进制形式
     * @return 私钥对象
     */
    private PrivateKey toPrivateKey(byte[] key) throws Exception{
        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(key);
        KeyFactory keyFactory = KeyFactory.getInstance(this.keyAlgorithm);
        return keyFactory.generatePrivate(pkcs8EncodedKeySpec);
    }

    /**
     * 取得公钥
     * @param keyMap 密钥对map
     * @return 公钥的二进制形式
     */
    public byte[] getPublicKey(Map<String, Key> keyMap){
        return keyMap.get(PUBLIC_KEY).getEncoded();
    }

    /**
     * 取得私钥
     * @param keyMap 密钥对map
     * @return 私钥的二进制形式
     */
    public byte[] getPrivateKey(Map<String, Key> keyMap){
        return keyMap.get(PRIVATE_KEY).getEncoded();
    }

    /**
     * 生成秘钥对
     * @return 密钥对
     */
    public Map<String, Key> initKey() throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(this.keyAlgorithm);
        keyPairGenerator.initialize(this.keySize);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        PrivateKey privateKey = keyPair.getPrivate();
        PublicKey publicKey = keyPair.getPublic();
        Map<String, Key> keyMap = new HashMap<String, Key>();
        keyMap.put(PRIVATE_KEY, privateKey);
        keyMap.put(PUBLIC_KEY, publicKey);
        return keyMap;
    }

    /**
     * 加密操作
     * @param data 需要加密的数据
     * @param key 密钥对象
     * @return 加密后的数据
     */
    private byte[] encrypt(byte[] data, Key key) throws Exception{
        Cipher cipher = cipher = Cipher.getInstance(this.cipherAlgorithm);
        cipher.init(Cipher.ENCRYPT_MODE, key);
        return cipher.doFinal(data);
    }

    /**
     * 解密操作
     * @param data 需要解密的数据
     * @param key 密钥对象
     * @return 解密后的数据
     */
    private byte[] decrypt(byte[] data, Key key) throws Exception{
        Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
        cipher.init(Cipher.DECRYPT_MODE, key);
        return cipher.doFinal(data);
    }

    /**
     * 测试
     * @param args
     */
    public static void main(String[] args) throws Exception{
        String data = "测试非对称密钥算法加解密";
        System.out.println("加密前数据：" + data);
        // 其它非对称密钥算法更改构造参数即可
        // ELGAMAL算法只支持公钥加密私钥解密，并且需要BouncyCastle扩展支持
//        Security.addProvider(new BouncyCastleProvider());
//        NonSymmetricCryptography nonSymmetricCryptography = new NonSymmetricCryptography("ELGAMAL", "ELGAMAL/ECB/PKCS1Padding", 512);
        NonSymmetricCryptography nonSymmetricCryptography = new NonSymmetricCryptography("RSA", "RSA/ECB/PKCS1Padding", 1024);
        Map<String, Key> keyMap = nonSymmetricCryptography.initKey();
        System.out.println("公钥：" + Base64.encodeBase64String(nonSymmetricCryptography.getPublicKey(keyMap)));
        System.out.println("私钥：" + Base64.encodeBase64String(nonSymmetricCryptography.getPrivateKey(keyMap)));

        // 公钥加密私钥解密
        String encryptData = Base64.encodeBase64String(nonSymmetricCryptography.encryptByPublicKey(data.getBytes(), nonSymmetricCryptography.getPublicKey(keyMap)));
        System.out.println("使用公钥加密后数据：" + encryptData);
        String decryptData = new String(nonSymmetricCryptography.decryptByPrivateKey(Base64.decodeBase64(encryptData), nonSymmetricCryptography.getPrivateKey(keyMap)));
        System.out.println("使用私钥解密后数据：" + decryptData);

        // 私钥加密公钥解密
        String encryptData1 = Base64.encodeBase64String(nonSymmetricCryptography.encryptByPrivateKey(data.getBytes(), nonSymmetricCryptography.getPrivateKey(keyMap)));
        System.out.println("使用私钥加密后数据：" + encryptData1);
        String decryptData1 = new String(nonSymmetricCryptography.decryptByPublicKey(Base64.decodeBase64(encryptData1), nonSymmetricCryptography.getPublicKey(keyMap)));
        System.out.println("使用公钥解密后数据：" + decryptData1);
    }

}

```

#### 密钥协商算法加解密实现

```java

import org.apache.commons.codec.binary.Base64;

import javax.crypto.Cipher;
import javax.crypto.KeyAgreement;
import javax.crypto.SecretKey;
import javax.crypto.interfaces.DHKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.*;
import java.security.interfaces.ECKey;
import java.security.spec.AlgorithmParameterSpec;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

public class KeyAgreementCryptography {

    private static final String PUBLIC_KEY = "PublicKey";

    private static final String PRIVATE_KEY = "PrivateKey";

    /**
     * 密钥生成算法
     */
    private String keyAlgorithm;

    /**
     * 加解密算法/分组策略/填充方式
     */
    private String cipherAlgorithm;

    /**
     * 密钥长度
     */
    private int keySize;

    public KeyAgreementCryptography(String keyAlgorithm, String cipherAlgorithm, int keySize) {
        this.keyAlgorithm = keyAlgorithm;
        this.cipherAlgorithm = cipherAlgorithm;
        this.keySize = keySize;
    }

    /**
     * 转换密钥
     * @param key 密钥的二进制形式
     * @return Key 密钥对象
     */
    private Key toSecretKey(byte[] key) {
        return new SecretKeySpec(key, this.cipherAlgorithm);
    }

    /**
     * 加密操作
     * @param data 需要加密的数据
     * @param key 密钥的二进制形式
     * @return 加密后的数据
     */
    public byte[] encrypt(byte[] data, byte[] key) throws Exception{
        Key secretKey = this.toSecretKey(key);
        Cipher cipher = cipher = Cipher.getInstance(this.cipherAlgorithm);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        return cipher.doFinal(data);
    }

    /**
     * 解密操作
     * @param data 需要解密的数据
     * @param key 密钥的二进制形式
     * @return 解密后的数据
     */
    public byte[] decrypt(byte[] data, byte[] key) throws Exception{
        Key secretKey = this.toSecretKey(key);
        Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        return cipher.doFinal(data);
    }

    /**
     * 根据甲方私钥和乙方公钥生成甲方本地对称密钥，或者根据乙方私钥和甲方公钥生成乙方本地对称密钥
     * @param publicKey 公钥二进制形式
     * @param privateKey 私钥二进制形式
     * @return 对称密钥对象
     */
    public byte[] initSecretKey(byte[] publicKey, byte[] privateKey) throws Exception{
        PublicKey pubKey = this.toPublicKey(publicKey);
        PrivateKey priKey = this.toPrivateKey(privateKey);
        KeyAgreement keyAgreement = KeyAgreement.getInstance(this.keyAlgorithm);
        keyAgreement.init(priKey);
        keyAgreement.doPhase(pubKey, true);
        SecretKey secretKey = keyAgreement.generateSecret(this.cipherAlgorithm);
        return secretKey.getEncoded();
    }

    /**
     * 将公钥二进制形式转换成公钥对象
     * @param key 公钥的二进制形式
     * @return 公钥对象
     */
    private PublicKey toPublicKey(byte[] key) throws Exception{
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(key);
        KeyFactory keyFactory = KeyFactory.getInstance(this.keyAlgorithm);
        return keyFactory.generatePublic(x509EncodedKeySpec);
    }

    /**
     * 将私钥二进制形式转换成私钥对象
     * @param key 私钥的二进制形式
     * @return 私钥对象
     */
    private PrivateKey toPrivateKey(byte[] key) throws Exception{
        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(key);
        KeyFactory keyFactory = KeyFactory.getInstance(this.keyAlgorithm);
        return keyFactory.generatePrivate(pkcs8EncodedKeySpec);
    }

    /**
     * 取得公钥
     * @param keyMap 密钥对map
     * @return 公钥的二进制形式
     */
    public byte[] getPublicKey(Map<String, Key> keyMap){
        return keyMap.get(PUBLIC_KEY).getEncoded();
    }

    /**
     * 取得私钥
     * @param keyMap 密钥对map
     * @return 私钥的二进制形式
     */
    public byte[] getPrivateKey(Map<String, Key> keyMap){
        return keyMap.get(PRIVATE_KEY).getEncoded();
    }

    /**
     * 初始化密钥协商算法的甲方密钥对
     * @return 甲方密钥对
     */
    public Map<String, Key> initKey() throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(this.keyAlgorithm);
        keyPairGenerator.initialize(this.keySize);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        PrivateKey privateKey = keyPair.getPrivate();
        PublicKey publicKey = keyPair.getPublic();
        Map<String, Key> keyMap = new HashMap<String, Key>();
        keyMap.put(PRIVATE_KEY, privateKey);
        keyMap.put(PUBLIC_KEY, publicKey);
        return keyMap;
    }

    /**
     * 初始化密钥协商算法的乙方密钥对
     * @param publicKey 甲方公钥的二进制形式
     * @return 乙方密钥对
     */
    public Map<String, Key> initKey(byte[] publicKey) throws Exception{
        PublicKey pubKey = this.toPublicKey(publicKey);
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(this.keyAlgorithm);
        AlgorithmParameterSpec algorithmParameterSpec = null;
        if(pubKey instanceof DHKey){
            algorithmParameterSpec = ((DHKey) pubKey).getParams();
        } else if(pubKey instanceof ECKey) {
            algorithmParameterSpec = ((ECKey) pubKey).getParams();
        }
        keyPairGenerator.initialize(algorithmParameterSpec);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        Map<String, Key> keyMap = new HashMap<String, Key>();
        keyMap.put(PRIVATE_KEY, keyPair.getPrivate());
        keyMap.put(PUBLIC_KEY, keyPair.getPublic());
        return keyMap;
    }

    /**
     * 测试
     * @param args
     */
    public static void main(String[] args) throws Exception{
        String data = "测试密钥协商算法加解密";
        KeyAgreementCryptography keyAgreementCryptography = new KeyAgreementCryptography("DH", "DES", 1024);
        // 获取甲方密钥对
        Map<String,Key> keyMap = keyAgreementCryptography.initKey();
        String privateKey = Base64.encodeBase64String(keyAgreementCryptography.getPrivateKey(keyMap));
        String publicKey = Base64.encodeBase64String(keyAgreementCryptography.getPublicKey(keyMap));
        System.out.println("甲方DH私钥：" + privateKey);
        System.out.println("甲方DH公钥：" + publicKey);

        // 获取乙方密钥对
        Map<String,Key> keyMap1 = keyAgreementCryptography.initKey(Base64.decodeBase64(publicKey));
        System.out.println("加密前数据：" + data);
        String privateKey1 = Base64.encodeBase64String(keyAgreementCryptography.getPrivateKey(keyMap1));
        String publicKey1 = Base64.encodeBase64String(keyAgreementCryptography.getPublicKey(keyMap1));
        System.out.println("乙方DH私钥：" + privateKey1);
        System.out.println("乙方DH公钥：" + publicKey1);

        // 获取甲方本地对称密钥
        String secretKey = Base64.encodeBase64String(keyAgreementCryptography.initSecretKey(Base64.decodeBase64(publicKey1), Base64.decodeBase64(privateKey)));
        System.out.println("甲方本地对称密钥：" + secretKey);

        // 获取乙方本地对称密钥
        String secretKey1 = Base64.encodeBase64String(keyAgreementCryptography.initSecretKey(Base64.decodeBase64(publicKey), Base64.decodeBase64(privateKey1)));
        System.out.println("乙方本地对称密钥：" + secretKey1);

        // 甲方加密乙方解密
        String encryptData = Base64.encodeBase64String(keyAgreementCryptography.encrypt(data.getBytes(), Base64.decodeBase64(secretKey)));
        System.out.println("甲方加密后数据：" + encryptData);
        String decryptData = new String(keyAgreementCryptography.decrypt(Base64.decodeBase64(encryptData), Base64.decodeBase64(secretKey1)));
        System.out.println("乙方解密后数据：" + decryptData);
        // 乙方加密甲方解密
        String encryptData1 = Base64.encodeBase64String(keyAgreementCryptography.encrypt(data.getBytes(), Base64.decodeBase64(secretKey1)));
        System.out.println("乙方加密后数据：" + encryptData1);
        String decryptData1 = new String(keyAgreementCryptography.decrypt(Base64.decodeBase64(encryptData1), Base64.decodeBase64(secretKey)));
        System.out.println("甲方解密后数据：" + decryptData1);
    }
}

```

#### 基于口令的加解密实现

```java

import org.apache.commons.codec.binary.Base64;

import javax.crypto.Cipher;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.PBEParameterSpec;
import java.security.Key;
import java.security.SecureRandom;
import java.security.spec.AlgorithmParameterSpec;
import java.security.spec.KeySpec;

public class PBECryptography {

    /**
     * 密钥生成算法
     */
    private String keyAlgorithm;

    /**
     * 加解密算法/分组策略/填充方式
     */
    private String cipherAlgorithm;

    /**
     * 密钥长度
     */
    private int keySize;

    /**
     * PBE算法消息摘要迭代次数
     */
    private int pbeIterationCount;

    public PBECryptography(String keyAlgorithm, String cipherAlgorithm, int keySize, int pbeIterationCount) {
        this.keyAlgorithm = keyAlgorithm;
        this.cipherAlgorithm = cipherAlgorithm;
        this.keySize = keySize;
        this.pbeIterationCount = pbeIterationCount;
    }

    /**
     * 根据口令获取密钥对象
     * @param password 口令
     * @return 密钥对象
     */
    private Key toSecretKey(String password) throws Exception{
        KeySpec keySpec =  new PBEKeySpec(password.toCharArray());
        SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance(this.keyAlgorithm);
        return secretKeyFactory.generateSecret(keySpec);
    }

    /**
     * 初始化盐
     * @return 盐的二进制形式
     */
    public byte[] initSalt(){
        return new SecureRandom().generateSeed(8);
    }

    /**
     * 加密操作
     * @param data 需要加密的数据
     * @param password 口令
     * @param salt 盐
     * @return 加密后的数据
     */
    public byte[] encrypt(byte[] data, String password, byte[] salt) throws Exception{
        Key key = this.toSecretKey(password);
        AlgorithmParameterSpec algorithmParameterSpec = new PBEParameterSpec(salt, this.pbeIterationCount);
        Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
        cipher.init(Cipher.ENCRYPT_MODE, key, algorithmParameterSpec);
        return cipher.doFinal(data);
    }

    /**
     * 解密操作
     * @param data 需要解密的数据
     * @param password 口令
     * @param salt 盐
     * @return 解密后的数据
     */
    public byte[] decrypt(byte[] data, String password, byte[] salt) throws Exception{
        Key key = this.toSecretKey(password);
        AlgorithmParameterSpec algorithmParameterSpec = new PBEParameterSpec(salt, this.pbeIterationCount);
        Cipher cipher = Cipher.getInstance(this.cipherAlgorithm);
        cipher.init(Cipher.DECRYPT_MODE, key, algorithmParameterSpec);
        return cipher.doFinal(data);
    }

    /**
     * 测试
     * @param args
     */
    public static void main(String[] args) throws Exception {
        String data = "测试基于口令的加解密算法";
        // 其它PBE算法更改构造参数即可
        // PBECryptography pbeCryptography = new PBECryptography("PBEWithSHA1AndRC2_40", "PBEWithSHA1AndRC2_40", 128, 100);
        PBECryptography pbeCryptography = new PBECryptography("PBEWithMD5AndDES", "PBEWithMD5AndDES", 56, 100);
        String password = "password";
        System.out.println("口令：" + password);
        String salt = Base64.encodeBase64String(pbeCryptography.initSalt());
        System.out.println("盐：" + salt);
        String encryptData = Base64.encodeBase64String(pbeCryptography.encrypt(data.getBytes(), password, Base64.decodeBase64(salt)));
        System.out.println("加密前数据：" + data);
        System.out.println("加密后数据：" + encryptData);
        String decryptData = new String(pbeCryptography.decrypt(Base64.decodeBase64(encryptData), password, Base64.decodeBase64(salt)));
        System.out.println("解密后数据：" + decryptData);
    }
}

```

### commons-codec和BoucyCastle

commons-codec：Apache下的公共编码包，包含对Java平台消息摘要算法封装的工具类和Base64（含Base64URL）等算法的处理

BoucyCastle：开源的Java/C#加解密组件，实现了Java平台当前还没有实现的一些加解密算法，commons-codec包实现的工具类在这里基本都有实现。
