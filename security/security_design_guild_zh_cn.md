# MindSpore开源安全设计指南
本规范主要参考业界广泛标准和开源最佳实践，结合MindSpore业务发展趋势，制定出MindSpore开源安全设计指南，用于指导开发者进行模块设计，避免出现高安全设计风险。

## 1. 身份认证&鉴权
1.1 所有跨网络传输的机机接口需要有接入认证机制， 且认证处理过程应放到服务端进行。  
**说明**：跨网络接口需要支持身份认证避免攻击者仿冒接入。  
  
1.2 应在服务器端对于每一个需要授权访问的请求核实用户身份是否被授权执行这个操作。  
**说明**：URL越权是典型的Web安全漏洞，利用该漏洞，攻击者可以很轻易地绕过系统的权限控制，未经授权地访问系统资源、使用系统的功能。为防止用户通过直接输入URL，越权请求并执行一些页面，在请求某一URL时，后台需对请求该URL的用户进行权限鉴定。  

1.3 在服务器端对所有来自不可信数据源进行大小、类型、长度、以及特殊字符等合法性校验，拒绝任何没有通过校验的数据。  
**说明**：避免攻击者通过代理拦截并篡改请求绕过客户端的合法性校验，所以需要在服务端进行数据校验。  

1.4 不应存在可绕过系统安全机制（认证、权限控制、日志记录）对系统或数据进行访问的功能。  
**说明**：产品若存在可绕过系统安全机制对系统或数据进行访问的功能，有可能被恶意人员知晓进行私自操作，从而给系统造成很大影响。  

1.5 根据权限最小化原则，运行软件程序的帐号要尽可能的使用操作系统低权限的帐号。  
**说明**：对于运行软件程序的操作系统帐号，不应使用“root”、“administrator”、“supervisor”等特权帐号或高级别权限帐号，应该使用普通权限的账号运行。  

## 2. 安全传输  

2.1 在跨网络之间进行敏感数据（包括口令，批量图片、语音等个人数据）或关键业务数据(网络结构、模型参数、梯度数据)传输应采用安全传输协议或者加密后传输。  
**说明**：在跨网络进行传输时，容易遭受攻击者窃取或篡改，需要对重要数据进行保护。  

2.2 不应使用SSL2.0、SSL3.0、TLS1.0、TLS1.1协议进行安全传输，推荐使用更安全协议TLS1.2和TLS1.3。  
**说明**：SSL2.0和SSL3.0协议因存在很多安全问题已分别于2011年3月和2015年6月被IETF禁用。TLS1.0协议中对称加密算法仅支持RC4算法和分组加密算法的CBC模式，RC4已经公认不安全并被IETF在TLS所有版本协议中禁用，而对称算法的CBC模式存在IV可预测的问题，从而容易受到BEAST攻击。因此，推荐使用TLS1.2和TLS1.3协议。  

## 3. 敏感/隐私数据保护  

3.1 认证凭据(如口令、密钥等)不允许明文存储在系统中，应该加密保护，在不需要还原明文的场景，应使用不可逆PBKDF2算法加密，如果需要还原，则可以使用AES-256 GCM算法加密。  

3.2 默认不应直接读取数据主体的个人数据，如果需要，则要提供相应接口获得数据主体授权。  
**说明**：个人数据都属于数据主体，如果需要访问和收集，需要获取数据主体同意并授权。  

3.3 默认不应对训练和推理过程中的数据转移给第三方，如果需要，则要提供相应接口获得数据主体授权。  
**说明**：在将数据主体的个人数据提供给第三方前，需提供合理方式告知数据主体转移的个人数据类型、转移目的以及数据接收方的相关信息，并获得数据主体的同意。  

3.4 如果要将个人数据用于营销、用户画像、市场调查，则要提供相应接口获得数据主体授权，并提供数据主体随时撤销授权的接口。  
**说明**：如果使用个人数据进行用户画像和营销时，需要获取获取用户明示授权，让用户自由选择是否同意将自己的个人数据用于提供基本服务。在欧盟，涉及用户画像和营销时，除了要告知用户有否决权，而且需要告知用户个人数据是否转出EEA、用户画像的逻辑、拒绝提供个人数据的后果。  

## 4. 加密算法&密钥管理  

4.1 不应使用私有的、非标准的密码算法。  
**说明**：私有的、非标准的密码算法不应该在产品中使用。一方面，非密码学专业人员设计的密码算法，其安全强度难以保证。另一方面，这类算法在技术上也未经业界分析验证，有可能存在未知的缺陷，并且这样的设计违背公开透明的安全设计原则，推荐使用业界常用的开源密码组件，如OpenSSL、OpenSSH等。  

4.2 不应使用不安全的密码算法，推荐使用强密码算法。  
**说明**：随着密码技术的发展以及计算能力的提升，一些密码算法已不再适合现今的安全领域，如不安全对称加密算法DES、RC4等，不安全非对称加密算法RSA 1024等，不安全哈希算法SHA-0、SHA-1、MD2、MD4、MD5等，不安全密钥协商DH-1024等，推荐使用AES-256、RSA-3072、SHA-256、PBKDF2或更强密码算法。  

4.3 用于对数据进行加密的密钥不能在代码中硬编码。  
**说明**：密钥要支持替换，避免长期使用增加被泄露风险。

4.4 密码算法中使用到的随机数应是密码学意义上的安全随机数。  
**说明**：随机数用于生成IV、盐值、密钥等，都属于密码算法用途。不安全的随机数使得密钥、IV等可被全部或部分预测。密码学意义上的安全随机数，要求必须保证其不可预测性。  

4.5 用于产生密钥的随机数发生器应使用安全随机数发生器。  
**说明**：如果密钥是通过随机数发生器来产生的，则该随机数发生器必须是安全的随机数发生器。使用不安全的随机数发生器来产生密钥，可能会使得到的密钥存在被预测的风险。例如可以使用： OpenSSL库的RAND_bytes、Linux操作系统的/dev/random设备接口、TPM中的随机数产生器（RNG）模块。  

## 5. 安全资料  

5.1 所有新增跨网络通信的公开函数接口、RESTful接口、本地函数接口、命令行接口、以及身份认证使用到的默认用户/口令都必须在详细设计文档中说明。  
**说明**：所有上面列举的新增接口都要在详细设计文档中说明，以便更好的理解相应的模块。  

5.2 对外通信连接必须是系统运行和维护必需，对使用到的所有通信端口需要在详细设计文档中说明，动态侦听端口必须限定合理范围。  
**说明**：若对外通信端口不在文档中说明，则可能会影响用户安全配置。  