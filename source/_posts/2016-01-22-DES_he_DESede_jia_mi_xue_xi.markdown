---
layout: post
title: "DES和DESede加密学习"
date: 2016-01-22 14:40:58 +0800
comments: true
categories: android
---
### DES和DESede算法了解以及在android中的使用 ###
##### 1. 算法介绍 #####
首先DES和DESede是对称加密，DESede是对DES算法改进的三重加密算法，不过处理速度也变得较慢，密钥计算时间较长
<!--more-->
##### 2. 在android中的应用
###### 2.1 DES使用
首先看一下使用代码
{% codeblock java DES.java %}
	public static final String KEY_ALGORITHM = "DES";

	//格式：加密(解密)算法/工作模式/填充模式
	public static final String CIPHER_ALGORITHM = "DES/ECB/PKCS5Padding";
    //编码
    public static byte[] desEncodeECB(String _key, String _data) {
	    try {
		    byte[] key = _key.getBytes(encoding);
		    byte[] data = _data.getBytes(encoding);
		    Key desKey;
		    DESKeySpec dks = new DESKeySpec(key);
		    SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(KEY_ALGORITHM);
		    desKey = keyFactory.generateSecret(dks);
		    Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
		    cipher.init(Cipher.ENCRYPT_MODE, desKey);
		    return cipher.doFinal(data);
	    } catch (Exception ex) {
	    	return null;
	    }
    }

	//解码
	public static byte[] desDecodeECB(String _key, byte[] data) {
        try {
            byte[] key = _key.getBytes(encoding);
            Key desKey;
            DESKeySpec spec = new DESKeySpec(key);
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance(KEY_ALGORITHM);
            desKey = keyFactory.generateSecret(spec);
            Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, desKey);
            return cipher.doFinal(data);
        } catch (Exception ex) {
            return null;
        }
    }
{% endcodeblock %}

**基本流程如下:**

① 实例化密钥规范

从源码中可以看到密钥的字节长度为 8
{% codeblock java DESKeySpec.java %}
public static final int DES_KEY_LEN = 8;	
{% endcodeblock %}
构造方法
{% codeblock java DESKeySpec.java%}
	/**
	 * key的长度不能 < 8
	 */
	public DESKeySpec(byte[] key) throws InvalidKeyException {
        this(key, 0);
    }

	/**
	 * @param offset 偏移位置
	 */
	public DESKeySpec(byte[] key, int offset) throws InvalidKeyException {
        if (key == null) {
            throw new NullPointerException("key == null");
        }
        if (key.length - offset < DES_KEY_LEN) {
            throw new InvalidKeyException("key too short");
        }
        this.key = new byte[DES_KEY_LEN];
        System.arraycopy(key, offset, this.key, 0, DES_KEY_LEN);
    }
{% endcodeblock %}
② 接着实例化密钥工厂

{% codeblock java SecretKeyFactory.java %}
	/**
	 * @param algorithm 算法名称(eg. DES)
	 */ 
	public static final SecretKeyFactory getInstance(String algorithm)
            throws NoSuchAlgorithmException {
        if (algorithm == null) {
            throw new NullPointerException("algorithm == null");
        }
        Engine.SpiAndProvider sap = ENGINE.getInstance(algorithm, null);
        return new SecretKeyFactory((SecretKeyFactorySpi) sap.spi, sap.provider, algorithm);
    }

	/**
	 * @param keyFacSpi SPI委托
	 * @param provider 密钥工厂的提供者
	 * @param algorithm 算法名称
	 */
	protected SecretKeyFactory(SecretKeyFactorySpi keyFacSpi,
            Provider provider, String algorithm) {
        this.provider = provider;
        this.algorithm = algorithm;
        this.spiImpl = keyFacSpi;
    }
{% endcodeblock %}
可以明显的看到是`Engine.SpiAndProvider`是主角。
一些概念：
`Engine`顾名思义，是引擎的意思。其实就是一个一个功能的封装。
`SPI`全名：(`Service Provider Interface`)服务提供者接口。
这块涉及到`Java Security`的知识，这块我不做阐述(主要是自己也没看懂)，具体链接[http://blog.csdn.net/innost/article/details/44081147](http://blog.csdn.net/innost/article/details/44081147 "深入理解Android之Java Security")

总之，这块会拿到一个密钥工厂对象

③ 根据密钥规范生成密钥

生成方法
{% codeblock java SecretKeyFactory.java %}
	public final SecretKey generateSecret(KeySpec keySpec)
            throws InvalidKeySpecException {
        return spiImpl.engineGenerateSecret(keySpec);
    }
{% endcodeblock %}
调用的是加密模块的方法

④ 获取加/解密类对象，并使用

不能实例化，获取对象
{% codeblock java Cipher.java%}
	public final byte[] doFinal(byte[] input) throws IllegalBlockSizeException,
            BadPaddingException {
        if (mode != ENCRYPT_MODE && mode != DECRYPT_MODE) {
            throw new IllegalStateException();
        }
        return spiImpl.engineDoFinal(input, 0, input.length);
    }
{% endcodeblock %}

到此为止，android中DES加/解密的使用就学习到这。

**DESede**的使用方法和DES的一样，唯一不同的是密码规范`DESedeKeySpec`及算法名称`DESede`和DES的不同