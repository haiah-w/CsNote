https://www.jianshu.com/p/2b2d1f511959

# TLS/SSL
HTTPS = HTTP + TLS/SSL(应用层协议)

SSL：Secure Socket Layer，安全套接字层,已经更名为TLS;

TLS：传输层安全协议，是 SSL3.0 的后续版本，更安全，更完善；

TLS 1.1,1.2(废弃)

TLS 1.2(主流)

TLS 1.3(性能更好,更安全)

TLS建立在传输层、应用层之间,为应用层提供数据封装、压缩、加解密等功能；

# HTTPS握手过程


1、客户端告知服务端，自己支持的TLS版本、加密套件、和第1个随机数
2、服务端确认支持的TLS版本、选择的加密套件、并生成第2个随机数，返回给客户端
3、发送CA证书（含有服务端公钥、CA签名）给客户端
4、都发送完成，发送Server Hello结束报文
5、客户端使用CA公钥验签证书，验证服务端身份，获取服务端公钥
6、客户端生成第3个随机数，并使用公钥加密，发送给服务端，服务端私钥解密拿到预主密钥

只有客户端、服务端知道 预主密钥
双方通过3个随机数 计算出 会话密钥

7、服务端确认握手完成，表示可以使用会话密钥加密通信
后续数据通信，都使用预主密钥进行通信（对称加密：效率高）

两种加密方式用途

非对称加密：用于交换信息、验证身份、保障获取对称密钥（会话密钥）
对称加密：用于最终业务数据通信

# 单向认证



# 双向认证



# JKS/PKCS12



# 私钥格式
Pem格式文件是存储、传输密码学的密钥、公开密钥证书和其他数据的文件格式的业界标准
格式标准：
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQ.....
-----END RSA PRIVATE KEY-----
```


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

