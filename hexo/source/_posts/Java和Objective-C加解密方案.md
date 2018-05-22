---
title: Java和Objective-C加解密方案
date: 2018-05-08 09:38:59
tags: [每日小技巧, Java, Android, iOS, 服务器]
---

相信在做移动开发的时候都会遇到请求加密的问题，那么如何保证服务器端、前端(Android、iOS)加解密算法一致就是坑。服务器通常也是采用Java开发的，那么其实就是要保证Java和Objective-C算法一致。

经过我的研究以及采坑，下面有一个简单的方案。另外还有一个[aes的方案](https://github.com/DLTech21/libsecurity)
<!--more-->
### Java端

```java
import java.security.Key;

import javax.crypto.Cipher;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESedeKeySpec;
import javax.crypto.spec.IvParameterSpec;

public class Des3 {
    // 密钥
    private final static String secretKey = "dltech21@github@io";
    // 向量
    private final static String iv = "0123456789";
    // 加解密统一使用的编码方式
    private final static String encoding = "utf-8";

    /**
     * 3DES加密
     *
     * @param plainText 普通文本
     * @return
     * @throws Exception
     */
    public static String encode(String plainText) throws Exception {
        Key deskey = null;
        DESedeKeySpec spec = new DESedeKeySpec(secretKey.getBytes());
        SecretKeyFactory keyfactory = SecretKeyFactory.getInstance("desede");
        deskey = keyfactory.generateSecret(spec);

        Cipher cipher = Cipher.getInstance("desede/CBC/PKCS5Padding");
        IvParameterSpec ips = new IvParameterSpec(iv.getBytes());
        cipher.init(Cipher.ENCRYPT_MODE, deskey, ips);
        byte[] encryptData = cipher.doFinal(plainText.getBytes(encoding));
        return Base64.encode(encryptData);
    }

    /**
     * 3DES解密
     *
     * @param encryptText 加密文本
     * @return
     * @throws Exception
     */
    public static String decode(String encryptText) throws Exception {
        Key deskey = null;
        DESedeKeySpec spec = new DESedeKeySpec(secretKey.getBytes());
        SecretKeyFactory keyfactory = SecretKeyFactory.getInstance("desede");
        deskey = keyfactory.generateSecret(spec);
        Cipher cipher = Cipher.getInstance("desede/CBC/PKCS5Padding");
        IvParameterSpec ips = new IvParameterSpec(iv.getBytes());
        cipher.init(Cipher.DECRYPT_MODE, deskey, ips);

        byte[] decryptData = cipher.doFinal(Base64.decode(encryptText));

        return new String(decryptData, encoding);
    }
}
```

上面的加密工具类会使用到Base64这个类，该类的源代码如下

```java
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.OutputStream;

public class Base64 {
    private static final char[] legalChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/".toCharArray();

    public static String encode(byte[] data) {
        int start = 0;
        int len = data.length;
        StringBuffer buf = new StringBuffer(data.length * 3 / 2);

        int end = len - 3;
        int i = start;
        int n = 0;

        while (i <= end) {
            int d = ((((int) data[i]) & 0x0ff) << 16) | ((((int) data[i + 1]) & 0x0ff) << 8) | (((int) data[i + 2]) & 0x0ff);

            buf.append(legalChars[(d >> 18) & 63]);
            buf.append(legalChars[(d >> 12) & 63]);
            buf.append(legalChars[(d >> 6) & 63]);
            buf.append(legalChars[d & 63]);

            i += 3;

            if (n++ >= 14) {
                n = 0;
                buf.append(" ");
            }
        }

        if (i == start + len - 2) {
            int d = ((((int) data[i]) & 0x0ff) << 16) | ((((int) data[i + 1]) & 255) << 8);

            buf.append(legalChars[(d >> 18) & 63]);
            buf.append(legalChars[(d >> 12) & 63]);
            buf.append(legalChars[(d >> 6) & 63]);
            buf.append("=");
        } else if (i == start + len - 1) {
            int d = (((int) data[i]) & 0x0ff) << 16;

            buf.append(legalChars[(d >> 18) & 63]);
            buf.append(legalChars[(d >> 12) & 63]);
            buf.append("==");
        }

        return buf.toString();
    }

    private static int decode(char c) {
        if (c >= 'A' && c <= 'Z')
            return ((int) c) - 65;
        else if (c >= 'a' && c <= 'z')
            return ((int) c) - 97 + 26;
        else if (c >= '0' && c <= '9')
            return ((int) c) - 48 + 26 + 26;
        else
            switch (c) {
                case '+':
                    return 62;
                case '/':
                    return 63;
                case '=':
                    return 0;
                default:
                    throw new RuntimeException("unexpected code: " + c);
            }
    }

    /**
     * Decodes the given Base64 encoded String to a new byte array. The byte array holding the decoded data is returned.
     */

    public static byte[] decode(String s) {

        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        try {
            decode(s, bos);
        } catch (IOException e) {
            throw new RuntimeException();
        }
        byte[] decodedBytes = bos.toByteArray();
        try {
            bos.close();
            bos = null;
        } catch (IOException ex) {
            System.err.println("Error while decoding BASE64: " + ex.toString());
        }
        return decodedBytes;
    }

    private static void decode(String s, OutputStream os) throws IOException {
        int i = 0;

        int len = s.length();

        while (true) {
            while (i < len && s.charAt(i) <= ' ')
                i++;

            if (i == len)
                break;

            int tri = (decode(s.charAt(i)) << 18) + (decode(s.charAt(i + 1)) << 12) + (decode(s.charAt(i + 2)) << 6) + (decode(s.charAt(i + 3)));

            os.write((tri >> 16) & 255);
            if (s.charAt(i + 2) == '=')
                break;
            os.write((tri >> 8) & 255);
            if (s.charAt(i + 3) == '=')
                break;
            os.write(tri & 255);

            i += 4;
        }
    }
}
```

### Objective-C
这里会采用通用的GTMBase64，可以通过下载源码或者pod 'GTMBase64'

```
//  
//  DES3Util.h  
//  
  
#import <Foundation/Foundation.h>  
  
  
@interface DES3Util : NSObject {  
  
}  
  
// 加密方法  
+ (NSString*)encrypt:(NSString*)plainText;  
  
// 解密方法  
+ (NSString*)decrypt:(NSString*)encryptText;  
  
@end  
```

```
//  
//  DES3Util.m  
//  
  
#import "DES3Util.h"  
#import <CommonCrypto/CommonCryptor.h>  
#import "GTMBase64.h"  
  
#define gkey            @"dltech21@github@io"  
#define gIv             @"0123456789"  
  
@implementation DES3Util  
  
// 加密方法  
+ (NSString*)encrypt:(NSString*)plainText {  
    NSData* data = [plainText dataUsingEncoding:NSUTF8StringEncoding];  
    size_t plainTextBufferSize = [data length];  
    const void *vplainText = (const void *)[data bytes];  
      
    CCCryptorStatus ccStatus;  
    uint8_t *bufferPtr = NULL;  
    size_t bufferPtrSize = 0;  
    size_t movedBytes = 0;  
      
    bufferPtrSize = (plainTextBufferSize + kCCBlockSize3DES) & ~(kCCBlockSize3DES - 1);  
    bufferPtr = malloc( bufferPtrSize * sizeof(uint8_t));  
    memset((void *)bufferPtr, 0x0, bufferPtrSize);  
      
    const void *vkey = (const void *) [gkey UTF8String];  
    const void *vinitVec = (const void *) [gIv UTF8String];  
      
    ccStatus = CCCrypt(kCCEncrypt,  
                       kCCAlgorithm3DES,  
                       kCCOptionPKCS7Padding,  
                       vkey,  
                       kCCKeySize3DES,  
                       vinitVec,  
                       vplainText,  
                       plainTextBufferSize,  
                       (void *)bufferPtr,  
                       bufferPtrSize,  
                       &movedBytes);  
      
    NSData *myData = [NSData dataWithBytes:(const void *)bufferPtr length:(NSUInteger)movedBytes];  
    NSString *result = [GTMBase64 stringByEncodingData:myData];  
    return result;  
}  
  
// 解密方法  
+ (NSString*)decrypt:(NSString*)encryptText {  
    NSData *encryptData = [GTMBase64 decodeData:[encryptText dataUsingEncoding:NSUTF8StringEncoding]];  
    size_t plainTextBufferSize = [encryptData length];  
    const void *vplainText = [encryptData bytes];  
      
    CCCryptorStatus ccStatus;  
    uint8_t *bufferPtr = NULL;  
    size_t bufferPtrSize = 0;  
    size_t movedBytes = 0;  
      
    bufferPtrSize = (plainTextBufferSize + kCCBlockSize3DES) & ~(kCCBlockSize3DES - 1);  
    bufferPtr = malloc( bufferPtrSize * sizeof(uint8_t));  
    memset((void *)bufferPtr, 0x0, bufferPtrSize);  
      
    const void *vkey = (const void *) [gkey UTF8String];  
    const void *vinitVec = (const void *) [gIv UTF8String];  
      
    ccStatus = CCCrypt(kCCDecrypt,  
                       kCCAlgorithm3DES,  
                       kCCOptionPKCS7Padding,  
                       vkey,  
                       kCCKeySize3DES,  
                       vinitVec,  
                       vplainText,  
                       plainTextBufferSize,  
                       (void *)bufferPtr,  
                       bufferPtrSize,  
                       &movedBytes);  
      
    NSString *result = [[[NSString alloc] initWithData:[NSData dataWithBytes:(const void *)bufferPtr   
                                length:(NSUInteger)movedBytes] encoding:NSUTF8StringEncoding] autorelease];  
    return result;  
}  
  
@end 
```

其实请求内容加密只是安全的一种，还有就是API加校验，保证API的访问门槛。


