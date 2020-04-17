  

本文主要讲解在APP上如何实现双向RSA + AES加密。

先上一张主要流程图：

![](https://img-blog.csdn.net/20180124093540459)  

  

场景预设：

由于客户端是APP而不是网页，APP在第一次加载的时候会生成一对RSA秘钥对(我们称它为APP公钥私钥，不同APP的秘钥对不一样)，生成以后就写在配置文件里，而且每次都不变，这样可以保证Server公钥和APP公钥不会在网络上明文传输，从而避免了被掉包的可能。服务器也生成一对RSA秘钥对(我们称它为Server公钥私钥)，也是不可修改的。APP端也会事先将服务端的Server公钥写死在配置文件里。主要的交互流程如上图所示。

  

接下来上代码

  

AES的工具类：AESUtil

```java
package com.zhuyun.encrypt;
 
import java.io.IOException;
 
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
 
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;
 
public class AESUtil {
	//生成AES秘钥，然后Base64编码
	public static String genKeyAES() throws Exception{
		KeyGenerator keyGen = KeyGenerator.getInstance("AES");
		keyGen.init(128);
		SecretKey key = keyGen.generateKey();
		String base64Str = byte2Base64(key.getEncoded());
		return base64Str;
	}
	
	//将Base64编码后的AES秘钥转换成SecretKey对象
	public static SecretKey loadKeyAES(String base64Key) throws Exception{
		byte[] bytes = base642Byte(base64Key);
		SecretKeySpec key = new SecretKeySpec(bytes, "AES");
		return key;
	}
	
	//加密
	public static byte[] encryptAES(byte[] source, SecretKey key) throws Exception{
		Cipher cipher = Cipher.getInstance("AES");
		cipher.init(Cipher.ENCRYPT_MODE, key);
		return cipher.doFinal(source);
	}
	
	//解密
	public static byte[] decryptAES(byte[] source, SecretKey key) throws Exception{
		Cipher cipher = Cipher.getInstance("AES");
		cipher.init(Cipher.DECRYPT_MODE, key);
		return cipher.doFinal(source);
	}
	
	//字节数组转Base64编码
	public static String byte2Base64(byte[] bytes){
		BASE64Encoder encoder = new BASE64Encoder();
		return encoder.encode(bytes);
	}
	
	//Base64编码转字节数组
	public static byte[] base642Byte(String base64Key) throws IOException{
		BASE64Decoder decoder = new BASE64Decoder();
		return decoder.decodeBuffer(base64Key);
	}
}
```
  
  

  

RSA工具类：RSAUtil

```java
package com.zhuyun.encrypt;
 
import java.io.IOException;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
 
import javax.crypto.Cipher;
 
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;
 
public class RSAUtil {
	//生成秘钥对
	public static KeyPair getKeyPair() throws Exception {
		KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
		keyPairGenerator.initialize(2048);
		KeyPair keyPair = keyPairGenerator.generateKeyPair();
		return keyPair;
	}
	
	//获取公钥(Base64编码)
	public static String getPublicKey(KeyPair keyPair){
		PublicKey publicKey = keyPair.getPublic();
		byte[] bytes = publicKey.getEncoded();
		return byte2Base64(bytes);
	}
	
	//获取私钥(Base64编码)
	public static String getPrivateKey(KeyPair keyPair){
		PrivateKey privateKey = keyPair.getPrivate();
		byte[] bytes = privateKey.getEncoded();
		return byte2Base64(bytes);
	}
	
	//将Base64编码后的公钥转换成PublicKey对象
	public static PublicKey string2PublicKey(String pubStr) throws Exception{
		byte[] keyBytes = base642Byte(pubStr);
		X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
		PublicKey publicKey = keyFactory.generatePublic(keySpec);
		return publicKey;
	}
	
	//将Base64编码后的私钥转换成PrivateKey对象
	public static PrivateKey string2PrivateKey(String priStr) throws Exception{
		byte[] keyBytes = base642Byte(priStr);
		PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
		PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
		return privateKey;
	}
	
	//公钥加密
	public static byte[] publicEncrypt(byte[] content, PublicKey publicKey) throws Exception{
		Cipher cipher = Cipher.getInstance("RSA");
		cipher.init(Cipher.ENCRYPT_MODE, publicKey);
		byte[] bytes = cipher.doFinal(content);
		return bytes;
	}
	
	//私钥解密
	public static byte[] privateDecrypt(byte[] content, PrivateKey privateKey) throws Exception{
		Cipher cipher = Cipher.getInstance("RSA");
		cipher.init(Cipher.DECRYPT_MODE, privateKey);
		byte[] bytes = cipher.doFinal(content);
		return bytes;
	}
	
	//字节数组转Base64编码
	public static String byte2Base64(byte[] bytes){
		BASE64Encoder encoder = new BASE64Encoder();
		return encoder.encode(bytes);
	}
	
	//Base64编码转字节数组
	public static byte[] base642Byte(String base64Key) throws IOException{
		BASE64Decoder decoder = new BASE64Decoder();
		return decoder.decodeBuffer(base64Key);
	}
}
```
  
  

接下来是服务端的秘钥对(Server公钥和私钥)，我们选择写死在了代码里：KeyUtil

```java
package com.zhuyun.encrypt;
 
public class KeyUtil {
	//服务端的RSA公钥(Base64编码)
	public final static String SERVER_PUBLIC_KEY = "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApkixN3Dc6BLzb/V74VpxRXsSIu9AabGm"
												+ "K4xfcPiIqub0JS99a+P6XAOGuiMT2W4p1C8U9MZDRgHjUOrKGcc5ve9uT+U90LiAgwG58YdrklOT"
												+ "wlGvo6Xh4HQLRXMNoGsn6jLGdOV1RIVfWQ5EWfEB1+5v86QarLyfLIJ4ujVQfafEJ4dCwmoNSJk8"
												+ "xqVBAW9tDZlNOOgaZPJuEXVIFEEjIZCkFkFxkomwVNdp79Xewrj0mCybCDVy6Mcx3jOxY0gGwbGg"
												+ "S3YQxDbOpqYna8rcmf6CVJ2GA75sCU61Y8Of244CR5Rwkspbr1Pbf4UNSbVbpxzI08z1jrJvCVYW"
												+ "NQLMwwIDAQAB";
	
	//服务端的RSA私钥(Base64编码)
	public final static String SERVER_PRIVATE_KEY = "MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCmSLE3cNzoEvNv9XvhWnFFexIi"
												+ "70BpsaYrjF9w+Iiq5vQlL31r4/pcA4a6IxPZbinULxT0xkNGAeNQ6soZxzm9725P5T3QuICDAbnx"
												+ "h2uSU5PCUa+jpeHgdAtFcw2gayfqMsZ05XVEhV9ZDkRZ8QHX7m/zpBqsvJ8sgni6NVB9p8Qnh0LC"
												+ "ag1ImTzGpUEBb20NmU046Bpk8m4RdUgUQSMhkKQWQXGSibBU12nv1d7CuPSYLJsINXLoxzHeM7Fj"
												+ "SAbBsaBLdhDENs6mpidrytyZ/oJUnYYDvmwJTrVjw5/bjgJHlHCSyluvU9t/hQ1JtVunHMjTzPWO"
												+ "sm8JVhY1AszDAgMBAAECggEAFTOHhO4a/GwOJeRC20TQ1G8QrOucZt2DtmG7eYf2xPOVhXg8oZj7"
												+ "vuekMe9vBHYLV0Z5gYwV38M13IdTJV5FenYgtocgDpC3sfxyXN1LVejaGhiYMGFiH2AsX7p/rkh7"
												+ "Wl0G+LiY7xeiRJSRGnakKYf5NjNiQ0v5b49jHTrW/G5G579xHOC0EXJezgKKflD+XXhpBjVCMfzk"
												+ "Y8tK+ZT5XnUf2U+I7o9u6G3Y3wky2ajR0GyBigLGwI0mVqyA4ie3ggSiMxotfcDwP1HpV/fK//DP"
												+ "pUgBiUgspFt1NtVcNDxeQlqw3UYakfUn5j3inA4C+fZwAe7VNwZkW2uaCl7HIQKBgQDXE49dGomU"
												+ "rvIXeXt/muUw5sy81sufYxUdGm9slfr8DhyEr0KKdrnKFdLtRPdr0mHnp0mlihE+rQQyQg67qys3"
												+ "c4C4TcLq6PHmtkd6K0+rUDYM2BNltjgd42UA86tchZ5Qu1u7dqxmnExYUb/kXbNk/euOcv1iNkdV"
												+ "Gq61AZVhfwKBgQDF7G+4q92AQTjYk1dT0Us5oAkGj4GRs5XMeJB+0jhWRhyNvgXRuKkhMgEjbSog"
												+ "AnywQfD93xDw2nZw2TVFqSS5cGCR+KnwTkQcA2vE2IbduDQRTT9JUQas3EJhZ1Cp+xyXfAQorCQW"
												+ "5TSNfb0LskTnV+aJaM/Yxb32NzPlXksuvQKBgDn9FhxeOVYTTUazBG9FTiI/OFh5+XDCAEFWjVBT"
												+ "p9Yp39qOfnxiwnkQJUy/2Y4CrU8ONbciYL/rWkRKtzo2TnKm+7+1h6ZapE42O1NfNh3UhJ417BTy"
												+ "anL0ipkVGdDaXfMacQM8XgNUhOkTMY/bC7FhHQ/NRTAjvlvd09kN0j71AoGAHKaoWZRPgTRv1TIn"
												+ "DxQaDqJzDAcUG5JimfHOAP3Pd/W4RnB+iShxG0QQ1B8GXRHfGOjCyQ1Ud3k4cgePZaEhltKEuDzF"
												+ "5Op/g4qfPCSYCVqT9vk2sxdOnxFXbqA1FhYqwmcKdxTMOKA/ZkgQaLQKs26PCc8pX1josc617Xsj"
												+ "6QUCgYA3sHC9I8fan7FAneJvE1Fsc1ZATMuo/yNA8WlASg+OPgeCgIv5AHAvkKqj3ZYapnafmHJj"
												+ "6L6jUL9dFwKqMncAdVOYI/V0oVoa8wgCHQ6b4S55dWMYhd3hSg8+8mUdxw6oOFL7rpRYduX1KxKc"
												+ "8Y4OuB7RsQgdKlT/sEBDEFdS/w==";
}
```
  
  

  

接下来是主要的交互代码了，从流程图可以看出大体分为四个步骤：APP加密，Server解密，Server加密，APP解密。 我们把它封装成了四个方法，对外可直接调用：

  

HttpEncryptUtil  
```java
package com.zhuyun.encrypt;
 
import java.security.PrivateKey;
import java.security.PublicKey;
 
import javax.crypto.SecretKey;
 
import net.sf.json.JSONObject;
 
 
public class HttpEncryptUtil {
 
	//APP加密请求内容
	public static String appEncrypt(String appPublicKeyStr, String content) throws Exception{
		//将Base64编码后的Server公钥转换成PublicKey对象
		PublicKey serverPublicKey = RSAUtil.string2PublicKey(KeyUtil.SERVER_PUBLIC_KEY);
		//每次都随机生成AES秘钥
		String aesKeyStr = AESUtil.genKeyAES();
		SecretKey aesKey = AESUtil.loadKeyAES(aesKeyStr);
		//用Server公钥加密AES秘钥
		byte[] encryptAesKey = RSAUtil.publicEncrypt(aesKeyStr.getBytes(), serverPublicKey);
		//用AES秘钥加密APP公钥
		byte[] encryptAppPublicKey = AESUtil.encryptAES(appPublicKeyStr.getBytes(), aesKey);
		//用AES秘钥加密请求内容
		byte[] encryptRequest = AESUtil.encryptAES(content.getBytes(), aesKey);
		
		JSONObject result = new JSONObject();
		result.put("ak", RSAUtil.byte2Base64(encryptAesKey).replaceAll("\r\n", ""));
		result.put("apk", RSAUtil.byte2Base64(encryptAppPublicKey).replaceAll("\r\n", ""));
		result.put("ct", RSAUtil.byte2Base64(encryptRequest).replaceAll("\r\n", ""));
		return result.toString();
	}
	
	//APP解密服务器的响应内容
	public static String appDecrypt(String appPrivateKeyStr, String content) throws Exception{
		JSONObject result = JSONObject.fromObject(content);
		String encryptAesKeyStr = (String) result.get("ak");
		String encryptContent = (String) result.get("ct");
		
		//将Base64编码后的APP私钥转换成PrivateKey对象
		PrivateKey appPrivateKey = RSAUtil.string2PrivateKey(appPrivateKeyStr);
		//用APP私钥解密AES秘钥
		byte[] aesKeyBytes = RSAUtil.privateDecrypt(RSAUtil.base642Byte(encryptAesKeyStr), appPrivateKey);
		//用AES秘钥解密请求内容
		SecretKey aesKey = AESUtil.loadKeyAES(new String(aesKeyBytes));
		byte[] response = AESUtil.decryptAES(RSAUtil.base642Byte(encryptContent), aesKey);
		
		return new String(response);
	}
	
	//服务器加密响应给APP的内容
	public static String serverEncrypt(String appPublicKeyStr, String aesKeyStr, String content) throws Exception{
		//将Base64编码后的APP公钥转换成PublicKey对象
		PublicKey appPublicKey = RSAUtil.string2PublicKey(appPublicKeyStr);
		//将Base64编码后的AES秘钥转换成SecretKey对象
		SecretKey aesKey = AESUtil.loadKeyAES(aesKeyStr);
		//用APP公钥加密AES秘钥
		byte[] encryptAesKey = RSAUtil.publicEncrypt(aesKeyStr.getBytes(), appPublicKey);
		//用AES秘钥加密响应内容
		byte[] encryptContent = AESUtil.encryptAES(content.getBytes(), aesKey);
		
		JSONObject result = new JSONObject();
		result.put("ak", RSAUtil.byte2Base64(encryptAesKey).replaceAll("\r\n", ""));
		result.put("ct", RSAUtil.byte2Base64(encryptContent).replaceAll("\r\n", ""));
		return result.toString();
	}
	
	//服务器解密APP的请求内容
	public static String serverDecrypt(String content) throws Exception{
		JSONObject result = JSONObject.fromObject(content);
		String encryptAesKeyStr = (String) result.get("ak");
		String encryptAppPublicKeyStr = (String) result.get("apk");
		String encryptContent = (String) result.get("ct");
		
		//将Base64编码后的Server私钥转换成PrivateKey对象
		PrivateKey serverPrivateKey = RSAUtil.string2PrivateKey(KeyUtil.SERVER_PRIVATE_KEY);
		//用Server私钥解密AES秘钥
		byte[] aesKeyBytes = RSAUtil.privateDecrypt(RSAUtil.base642Byte(encryptAesKeyStr), serverPrivateKey);
		//用AES秘钥解密APP公钥
		SecretKey aesKey = AESUtil.loadKeyAES(new String(aesKeyBytes));
		byte[] appPublicKeyBytes = AESUtil.decryptAES(RSAUtil.base642Byte(encryptAppPublicKeyStr), aesKey);
		//用AES秘钥解密请求内容
		byte[] request = AESUtil.decryptAES(RSAUtil.base642Byte(encryptContent), aesKey);
		
		JSONObject result2 = new JSONObject();
		result2.put("ak", new String(aesKeyBytes));
		result2.put("apk", new String(appPublicKeyBytes));
		result2.put("ct", new String(request));
		return result2.toString();
	}
}
```

  
  

接下来是测试的代码

TestHttpEncrypt  

```java
package com.zhuyun.encrypt;
 
import java.io.InputStream;
import java.security.KeyPair;
import java.util.Properties;
 
import org.junit.Test;
 
 
public class TestHttpEncrypt {
 
	@Test
	public void testGenerateKeyPair() throws Exception{
		//生成RSA公钥和私钥，并Base64编码
		KeyPair keyPair = RSAUtil.getKeyPair();
		String publicKeyStr = RSAUtil.getPublicKey(keyPair);
		String privateKeyStr = RSAUtil.getPrivateKey(keyPair);
		System.out.println("RSA公钥Base64编码:" + publicKeyStr);
		System.out.println("RSA私钥Base64编码:" + privateKeyStr);
	}
	
	
	@Test
	public void testGenerateAesKey() throws Exception{
		//生成AES秘钥，并Base64编码
		String base64Str = AESUtil.genKeyAES();
		System.out.println("AES秘钥Base64编码:" + base64Str);
	}
	
	//测试  APP加密请求内容
	@Test
	public void testAppEncrypt() throws Exception{
		//APP端公钥和私钥从配置文件读取，不能写死在代码里
		Properties prop = new Properties();
		InputStream in = TestHttpEncrypt.class.getClassLoader().getResourceAsStream("client.properties");
		prop.load(in);
		String appPublicKey = prop.getProperty("app.public.key");
		//请求的实际内容
//		String content = "{\"name\":\"infi\", \"weight\":\"60\"}";
		String content = "{\"tenantid\":\"1\", \"account\":\"13015929018\", \"pwd\":\"123456\"}";
		String result = HttpEncryptUtil.appEncrypt(appPublicKey, content);
		System.out.println(result);
	}
	
	//测试  服务器解密APP的请求内容
	@Test
	public void testServerDecrypt() throws Exception{
		String result = "{\"ak\":\"iLHfi1XRz4gnirU2OKggNCkz5x0i6aSonm1u3bE+ncI4AuiUG9LX2nbrQV/lWUIqwRp/q/P+SrIPnh5JbgEzSi+K46N4enyDFYbWpC6gONqQpF3tNt6Q1Y+UdX3L5l9hFPAS9tIhI2kT10AbhMox2kKOhr6ZQmmC/A3qeFEbTuUUf8bOCr4nqz4qSNyCZgcJdoAQonJeN8IilWuTD+LpbllNimFNR/sGY5jlyjvVydrdpNs15oFaXtfTLUjSXe2e5Ha1r3K7lP93C2E+KL55001xFJhQZcZXa9ZlYCMQgI+2cJlED4uA3bl2ul1dtnvXK+41Yky9e9QrRDc5luqB6w==\",\"apk\":\"P0SJaTzKWuBMi/fj2G8wwZ9+FWFIrE3BAwdoXwIfiTxptYXumLxnMpZZkCBNqQBvhvSzAEPyA3c9kCjhYCxdTnV61N+T/DZM+B62u4vqCy1MsFZT06BJjrNFW29AfSRNmQdKhJEyDPARcf5FerULbIDWGvrHzHys7jVbicjlYWtQpnyQf5Wl0Bd7taEqSwUSKejoEsN74frwlk8Hu4KP4bLvVy9S7DjOP2juXbVkHYaKgVmhM2V3yElVOEb1TDCLSFMNtug74+7itlzlChDR8wEWdh11vQcp69iGmDXMo2vcJ9tO1YZP+hCYZvujHMRwAzHtkqafEoSJsvSN8PWS+qmQdUX2frf6A0cl6SGnTbGUUEV/w0rBIU/oGhP8cl8+ghqPbp7HzvwXFOJsUciy+7tsLRrdDpLeOcz+fh/c0RSpCKNEZtRmcuUqBQ+3tZKYGPhl+StsFh3s1RCkhI4EsSD95bCbES4r1r1E4dytdELi0ebJug7Quk3rwFVXGX9o4wrnnvcbTaSyyAAg2YTNfA==\",\"ct\":\"mkC7hE/crHbmW+h2OCMBANCA64xtMFLTRmLahOU+UysZrXzK30qRj6RUcvpQz4mJ6EOYYAK34+BQBkN9gapdIw==\"}";
		System.out.println(HttpEncryptUtil.serverDecrypt(result));
	}
	
	//测试 服务器加密响应给APP的内容
	@Test
	public void testserverEncrypt() throws Exception{
		String aesKeyStr = "dSRWXM6IkWkKk7I/ZGouqA==";
		String appPublicKeyStr = "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqqKPH/L0AZyn1fJ9xK2ol2nHY5jPu8qw7COwFukkRdr2j0oNJmD8vCTmxgzKWV0CkihiJ7Y0OekrGc78JL5tpL2SqeZTLa2bCJZJaTM3KFOXYb82nc8Xbr2caDnf7mgjyt0AALHG/YfYwd7hifZRB6Ct89uBTn6W5x/7oxGT6D1C8siXKV+99AZPMv2HobglWyquyjIL5TZOhYmCMzFUPMOiXzzGYXMZj2gmfUFXMf/2jitMPGg3zQPJxPSYunjoE1fMInk1obEhEfU8n2YxT5ZbGMWZGjt4hZwF+FJJLV+WOantfUJ4rMBB8qxgQtkT+VzddfLCEoyy4Rl50fvjzwIDAQAB";
		String content = "{\"retcode\":\"200\"}";
		System.out.println(HttpEncryptUtil.serverEncrypt(appPublicKeyStr, aesKeyStr, content));
	}
	
	//测试 	APP解密服务器的响应内容
	@Test
	public void testAppDecrypt() throws Exception{
		//APP端公钥和私钥从配置文件读取，不能写死在代码里
		Properties prop = new Properties();
		InputStream in = TestHttpEncrypt.class.getClassLoader().getResourceAsStream("client.properties");
		prop.load(in);
		String appPrivateKey = prop.getProperty("app.private.key");
		String content = "{\"ak\":\"mRBa005mea+6QIaFhTHrfCTBBFL+sy1uHI1iSN6LUK5/VQK/Bt9JZ+5/e2TQYMiD8U6KXBzZgHOl4RL8AErno9K4bbC+4Ke5Bl/IIGZ6kPJB4OjzbqBwxmmA+zJrcS3TlzIsVGpuIzGMQzIT0rlJl+BsQj6N9F3jfCeXBXH+JoTPEaTZqzQ9odgfPooP8jvuBOneqAiTmIgNzcVJwr7EB1tB65FjYPWFJqC0xrmLlrvev0KrD/XnKkzL1wGHc/eXeYzRXHuz4tbTHQV0mrZNz+tITXPVorRb0Tl0mglUafiqTkUBsXUv4abUvz2JImlF1nSAmQfKWfMNd7Fwag480g==\",\"ct\":\"DPMIYZaJL5e7Jvs2Vsy6jgnEPWBYFgjb1K1yf7gcWUCVyAfBPkLGK93onQkvLl8urp2yTwEsxzP6o1om0mqjkEU4oPpYf4NJC+QPQRQ2YTo=\"}";
		System.out.println(HttpEncryptUtil.appDecrypt(appPrivateKey, content));
	}
}
```
  
  

上面的过程需要APP从客户端的配置文件读取APP公钥和私钥，配置文件内容如下：

client.properties  

```properties
app.public.key=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAqqKPH/L0AZyn1fJ9xK2ol2nHY5jPu8qw7COwFukkRdr2j0oNJmD8vCTmxgzKWV0CkihiJ7Y0OekrGc78JL5tpL2SqeZTLa2bCJZJaTM3KFOXYb82nc8Xbr2caDnf7mgjyt0AALHG/YfYwd7hifZRB6Ct89uBTn6W5x/7oxGT6D1C8siXKV+99AZPMv2HobglWyquyjIL5TZOhYmCMzFUPMOiXzzGYXMZj2gmfUFXMf/2jitMPGg3zQPJxPSYunjoE1fMInk1obEhEfU8n2YxT5ZbGMWZGjt4hZwF+FJJLV+WOantfUJ4rMBB8qxgQtkT+VzddfLCEoyy4Rl50fvjzwIDAQAB
app.private.key=MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCqoo8f8vQBnKfV8n3EraiXacdjmM+7yrDsI7AW6SRF2vaPSg0mYPy8JObGDMpZXQKSKGIntjQ56SsZzvwkvm2kvZKp5lMtrZsIlklpMzcoU5dhvzadzxduvZxoOd/uaCPK3QAAscb9h9jB3uGJ9lEHoK3z24FOfpbnH/ujEZPoPULyyJcpX730Bk8y/YehuCVbKq7KMgvlNk6FiYIzMVQ8w6JfPMZhcxmPaCZ9QVcx//aOK0w8aDfNA8nE9Ji6eOgTV8wieTWhsSER9TyfZjFPllsYxZkaO3iFnAX4UkktX5Y5qe19QniswEHyrGBC2RP5XN118sISjLLhGXnR++PPAgMBAAECggEAAP6dkvQZlADTwZ1+Oi1A9FD7hosXeuK9kULL/fYx7e5OzZsC5JxgHMCiT7k3XLn8D9oIaG7ZcxT22VmpgpVRkkpAlpjvFy8R3kTx/Jj901BZa4pvyQ+x9UVJqhncQkl9G+uZ2mcu379w9gBUlDdJVaAMI4W+BTUbsBExqEur7wiZ8S3XK3rDltKINIguPRjwVTTuepzrTNd3k7WKPD8za/OV87bGEbaGLLo8KSKzm7bJnyMSbUwkjKA+WezNidUatB1fqaXDnlVYokCdfBD7xypbbVO+HoBNNBgAwT5p3dm3hbgYsb9WNZLJXREMaNtp+AhWHGFj4qeYRl8dfGO7oQKBgQDYXTChQhKdo80nFWMtFvh1yw75uma0oOpnHEiIfYh1ekcTBLlgsGTbg60t5Klb2lF1SdfhNXb+4WQNmSMlQYoKwakiRn2JyKbk+zs/j6q1r1PV1QAP5LJ28J/FjG5bEyCXQdE9nVhTps1EX2Fuy6I9pKCQKuv86NPOsX7IMat4VwKBgQDJ5NFcORrRgxLwcx/eYGjyeyf7EDKacVpd6SzobakXnvaX3WCxw2wjvbex+zL1s+XnLmp+4l87FfuP0+yd78pRHASXwDh6MNqs8bN2ksT4LOb7bImXygcxkX8Iog9qkydKZ1kUgLtDPZMzSUyic0+/6qVro9JR3aw0oJly9p4lSQKBgQCli6gJumRD+XCe1t5rQYgZmKR8rwKmcfjnq9xTkrk2Kbj39EVilZSV4MpAsxRiE0kAVN+4kQ/bNNk5DlK1zs+wKz0d3JFxOvV3fkJ2/5W+LcgXdEH35yQlnTaiEDDfvmLRWKqgWiOa3aVxCwmhnG0mfS/dHvoxKHPnUiePRXHNQQKBgQC7lZrgsT41xC9osc6+c52PDtbK8vXRgdiQwQI0ww8FH3HHEK2y/PwRCUkQWXGz0P6fmgTg97u7zmT58dI7vHyieAHcbYEMJzBG2BwC48OXQ0EqAmKlYdTlPWZmwwzH3Qn4m6Ws4x8bDq8iS8ykc7d5fa9NH91eqzRBgaaRpoqx4QKBgEzWhwVHhTxdSBzqh6JaTKVUOrt1CwsbQOSlOy/Y8k/TJFJaQh+/yGKSBpkGLfWkY5HVra+nhAgWuCB2X301DpSMCQtTiABYvjGNNysrkm40xQOuTOmO6OTqDfyVQZmi/xUXeztiT2vKjz0em+tButyg7OP7zKzYwqW3KhAxZ94t
```
  
  

  

最后将它放入了Web项目中，部署在tomcat上面，在相同的环境下大概测试了一下API的吞吐量，如下：

  

 

--- | 普通http | https | http加密(本文的方法)
--- | --- | --- | ---
同步 | 10000 TPS | 460 TPS | 760 TPS
异步 | 9000 TPS |760 TPS|1250 TPS

  

  

实验表明，在相同环境下它是https方法的吞吐量的1.6倍左右，而且比https更加安全，不过仅限于APP端，在浏览器上不适用。
