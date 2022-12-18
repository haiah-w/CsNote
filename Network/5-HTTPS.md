https://www.jianshu.com/p/2b2d1f511959

# 443端口和80端口

当发现使用的是Https协议，会自动使用443端口建立连接；

在建立连接过程中，规定数据传输使用的加密

1、客户端提交可用的TLS版本，双方确定TLS版本；

2、客户端提交可以使用的加密算法列表，双方确定通信使用的加密算法

3、客户端接受服务端的CA证书，并进行验证服务端身份；

4、客户端接受CA证书中的公钥，用于加密随机数；

5、通过3个随机数，生成双方通信使用的对称密钥；用于数据传输加密；

传输数据仍然使用80端口

# HTTPS握手抓包

ip.src_host == 192.168.1.68 or ip.dst_host == 220.194.117.141

# 单向认证

# 双向认证

# JKS/PKCS12

# openssl

openssl：开源的安全套接字层密码库，包含常用的密码算法，提供密钥生成、证书封装、验证等功能

1、生成私钥，pem格式文件(存储、传输密码学的密钥、公开密钥证书和其他数据的文件格式的业界标准)

```shell
openssl genrsa -out root.key 2048
```

可以从私钥中创建一个公钥：

```shell
openssl rsa -pubout -in root.key
```

2、用私钥生成请求文件(.csr)，用于

```shell
openssl req -new -out root-req.csr -key root.key -keyform PEM
```

3、自签名

```shell
openssl x509 -req -in root-req.csr -out root-cert.cer -signkey root.key -CAcreateserial -days 365
```

- x509：公钥证书的格式标准
- signkey：指定签名使用的私钥
- in：
- out：
- days：
- CAcreateserial：创建证书序列号
