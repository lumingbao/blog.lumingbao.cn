+++
title = "使用rsa对密码加密传输(js前端加密，java后台解密)"
date = 2021-11-22 09:12:00
url = "archives/16805"
tags = ["加解密"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner2.jpg'
+++

## 使用rsa对密码加密传输(js前端加密，java后台解密)

生成密钥
````java
package cn.lumingbao;

import org.bouncycastle.jce.provider.BouncyCastleProvider;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.Provider;
import java.security.SecureRandom;

/**
 * ============================
 *
 * @ClassName Test
 * @Description
 * @Author lumingbao
 * @Date 2021/11/17 15:56
 * ============================
 */
public class Test {
    public static void main(String[] args) throws Exception{
        //生产秘钥对
        //bouncy castle（轻量级密码术包）是一种用于 Java 平台的开放源码的轻量级密码术包
        Provider provider = new BouncyCastleProvider();
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA", provider);
        keyPairGen.initialize(1024, new SecureRandom());
        KeyPair keyPair = keyPairGen.generateKeyPair();
        //打印公钥
        System.out.println(keyPair.getPublic());
        //打印私钥
        System.out.println(keyPair.getPrivate());
    }
}
````

以上代码执行得到如下结果：
![](https://oss.lumingbao.cn/images/20211122/7bf38b2dc0c14d1cb0511abceeb79769.png)

其中private exponent是私钥用户后台解密不能泄露出去  
RSA Public Key 中的modulus和public exponent 公钥用于交给前段js加密

前端：引入js文件     security.js  
附上security.js地址：https://oss.lumingbao.cn/security.js

html页面中导入此js,以下为js加密代码
````javascript
//rsa 私钥
var password = 'ceshi';
var key = RSAUtils.getKeyPair('10001', '', '82110fdd17475879d8c853e0c081a3b705985266a066eac23270bb6b7fb3596da5e6aed75a6b3af15a0d2773b6de15d356a2ed427e7d374e9c1d4f639dc4174f2d838492a46a613151b4219955490eb64705474721c22a453d52cece42b31f03ab4811d173ca83cd76566cf287b2dfbdda0706137fbf53915285294cd5b2922f');
//加密后的密码
password = RSAUtils.encryptedString(key, password);
console.log("password:" + password);
````

java后台解密
````java
public static void main(String[] args) throws Exception {
    //js前端加密后的字符串
    String Password = "";
    
    //RSA Private CRT Key的modulus
    String hexModulus = "82110fdd17475879d8c853e0c081a3b705985266a066eac23270bb6b7fb3596da5e6aed75a6b3af15a0d2773b6de15d356a2ed427e7d374e9c1d4f639dc4174f2d838492a46a613151b4219955490eb64705474721c22a453d52cece42b31f03ab4811d173ca83cd76566cf287b2dfbdda0706137fbf53915285294cd5b2922f";
    //RSA Private CRT Key的private exponent
    String hexPrivateExponent = "3eb7df806b233a24b746123c4457bf0c182495476b7d752263943cabdf8e2a4757425f78f4ded433618b0a45201f03433f799d12fd4f8005e5fdb43482f4f58fc4ef0a9a72329658463042aa3febf42e5b8aa02c0ac3e4f6bc481e912c2afad8dfda9d75ffacfded25083b1bc8b2d86fcc3bcd3845824943c7f9d35c3e4cc211";
    Provider provider = new BouncyCastleProvider();
    KeyFactory keyFac = KeyFactory.getInstance("RSA", provider);
    RSAPrivateKeySpec priKeySpec = new RSAPrivateKeySpec(new BigInteger(hexModulus, 16), new BigInteger(hexPrivateExponent, 16));
    // 生成用于解密的私钥
    RSAPrivateKey pk = (RSAPrivateKey) keyFac.generatePrivate(priKeySpec);
    // 解密
    Cipher cipher = Cipher.getInstance("RSA", provider);
    cipher.init(2, pk);
    byte[] pwd = cipher.doFinal(Hex.decodeHex(Password.toCharArray()));
    Password = StringUtils.reverse(new String(pwd));
    
    System.out.println("解密后：" + Password);
}
````

未完待续......
