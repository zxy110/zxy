---
title: ECC
date: 2019-03-10 09:55:09
categories:
  - 密码学
tags: [区块链]
---

签名算法有RSA、DSA、ECC（ECDSA）等。前两者因为秘钥长度和性能的关系，现在使用越来越少，例如常见的RSA2048，秘钥长度就达到了2048bit，也就是2KB大小，在一些嵌入式场合消耗比较大，而ECC只需要224bit，因此比特币在保证数据安全性基础的算法选择上选择了ECC。
ECC也就是椭圆曲线密码学，在使用ECC进行数字签名的时候，需要构造一条曲线，也可以选择标准曲线，诸如：prime256v1、secp256r1、nistp256、secp256k1等等。我们使用的是secp256k1，也就是比特币选择的加密曲线,秘钥长度256bit，也就是32字节。

## secp256k1
### 秘钥生成
KeyUtils.java:
```
    /*
     * This function is used to generate ECC keys
     */
    public static KeyPair generateKeypair(String curveName)throws Exception{
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("EC");
        ECGenParameterSpec ecGenParameterSpec = new ECGenParameterSpec(curveName);
        keyPairGenerator.initialize(ecGenParameterSpec, new SecureRandom());
        KeyPair keyPair = keyPairGenerator.generateKeyPair();

        //System.out.println("public key: "+keyPair.getPublic());
        //System.out.println("private key: "+keyPair.getPrivate());
        return keyPair;
    }
```
secp256k1.java:
```
    public static void generateKeypair(){
        try{
            keyPair=KeyUtils.generateKeypair("secp256k1");
        }catch(Exception e){
            e.printStackTrace();
        }
    }
```

### 秘钥存储
KeyUtils.java:
```
    /*
     * The following two functions are used to save the keys.
     * You can run the follow command in terminal to print the pub.der and pri.der:
     * pub.der: openssl pkey -inform DER -pubin -in publickey.der -text
     * pri.der: openssl pkey -inform DER -in privatekey.der -text
     */
    public static void savePublicKey(KeyPair keyPair, String fileName)throws Exception{
        PublicKey publicKey = keyPair.getPublic();
        byte[] pub = publicKey.getEncoded();

        OutputStream pubfs = new FileOutputStream(fileName);
        for (int i = 0; i < pub.length; i++) {
            pubfs.write(pub[i]);
        }
    }
    public static void savePrivateKey(KeyPair keyPair, String fileName)throws Exception{
        PrivateKey privateKey=keyPair.getPrivate();
        byte[] pri = privateKey.getEncoded();

        OutputStream prifs=new FileOutputStream(fileName);
        for(int i=0;i<pri.length;i++){
            prifs.write(pri[i]);
        }
    }

```
secp256k1.java:
```
    public static void saveKeypair(){
        try{
            KeyUtils.savePublicKey(keyPair,"keys/pub.der");
            KeyUtils.savePrivateKey(keyPair,"keys/pri.der");
        }catch(Exception e){
            e.printStackTrace();
        }
    }
```

### 秘钥加载
KeyUtils.java:
```
    /*
     * The following two functions are used to read the keys.
     */
    public static PublicKey readPublicKey(byte[] encodedKey, String algorithm)
            throws NoSuchAlgorithmException, InvalidKeySpecException {
        X509EncodedKeySpec keySpec=new X509EncodedKeySpec(encodedKey);
        KeyFactory keyFactory=KeyFactory.getInstance(algorithm);
        return keyFactory.generatePublic(keySpec);
    }
    public static PrivateKey readPrivateKey(byte[] encodedKey, String algorithm)
            throws NoSuchAlgorithmException, InvalidKeySpecException{
        PKCS8EncodedKeySpec keySpec=new PKCS8EncodedKeySpec(encodedKey);
        KeyFactory keyFactory=KeyFactory.getInstance(algorithm);
        return keyFactory.generatePrivate(keySpec);
    }


    /*
     * The following two functions are used to get the keys.
     */
    public static PublicKey getPublicKey(String filename, String algorithm) throws Exception{
        return readPublicKey(IOUtils.readBytes(
                new FileInputStream(filename)),algorithm);
    }
    public static PrivateKey getPrivateKey(String filename, String algorithm) throws Exception{
        return readPrivateKey(IOUtils.readBytes(
                new FileInputStream(filename)),algorithm);
    }
```
secp256k1.java:
```
    /*
     * This function is used to get the public key
     */
    public static PublicKey getPublicKey(){
        try{
            return KeyUtils.getPublicKey("keys/pub.der","EC");
        }catch(Exception e){
            e.printStackTrace();
            return null;
        }
    }

    /*
     * This function is used to get the private key
     */
    public static PrivateKey getPrivateKey(){
        try{
            return KeyUtils.getPrivateKey("keys/pri.der","EC");
        }catch(Exception e){
            e.printStackTrace();
            return null;
        }
    }
```

## 签名
在Java中，签名和校验，都是通过： Signature 类来实现的。
该类的主要方法：
- getInstance(String algorithm)

工厂方法，获取Signature实例，而参数：algorithm就是签名算法的名称，这里我们使用的是：SHA256withECDSA
更具体的参数，请浏览：https://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#Signature

- initSign(PrivateKey privateKey)
- initVerify(PublicKey publicKey)

初始化成签名或者校验，Signature类的实例，在同一个阶段只能完成签名或者校验之一的任务，因此调用initSign或者initVeify让Signature进入相应的状态

- update(byte[] data)

无论是计算签名，还是校验数据，都需要传入被签名或者被校验的数据，调用该方法进行计算。注意，该方法可以调用多次，通常数据都可能非常大，不可能一次性读入内存（从磁盘上或者网络），因此我们可以对数据进行分块，一次性一KB或者合适的块进行多次调用

- sign()

获取签名，当所有的数据都调用update计算后，就可以获取签名了，返回的签名是一个byte数组

- verify(byte[] signature)

校验签名，当所有的数据都调用update计算之后，调用该方法，传递签名数据，如果签名正确，那么返回true，否则返回false，说明数据可能被篡改、伪造，或者对应的私钥不正确。

```
    public class Signatures {
    /*
     * This function is used to sign data with the algorithm and key
     */
    public static byte[] signData(String algorithm, byte[] data, PrivateKey key)
            throws Exception{
        Signature signer = Signature.getInstance(algorithm);
        signer.initSign(key);
        signer.update(data);
        return (signer.sign());
    }


    /*
     * This function is used to verify signature
     */
    public static boolean verifySign(String algorithm, byte[] data, PublicKey key, byte[] sig)
        throws Exception{
        Signature signer = Signature.getInstance(algorithm);
        signer.initVerify(key);
        signer.update(data);
        return (signer.verify(sig));
    }
}

```

## 参考
https://www.jianshu.com/p/676a0eb33d31

https://www.jianshu.com/p/e6ac2c75e692

https://www.pediy.com/kssd/pediy06/pediy6014.htm