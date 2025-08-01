- # 衡量随机性的尺度：熵
	- **[[$red]]==Entropy==**，熵，熵可以用需要用多少位(bit)编码所有可能性来衡量
	- 信息位数$= \log_2(\#Possibility)$
	- 例如：
		- 扔一枚硬币，有两种可能性，则需要$\log_22=1$位来编码所有可能性
		- 扔骰子，有六种可能性，需要$\log_26\approx 3$位来编码所有可能性
	- 熵常常用来衡量一个密码的强度，在密码学中使用很广泛
- # 哈希函数
	- **[[$red]]==Hash==**
	- 将数据映射为固定长度的一串bit
	- 著名的算法有sha1等
		- 在linux下使用``sha1sum``命令可以计算输入的sha1哈希码
	- 哈希函数拥有一些**重要的性质**
		- 给出一个hash结果，[[$red]]==难以==逆向推得原数据，即**不可逆**
		- **防碰撞(collision resistant)**，很难发现两个不同的输出拥有完全一致的hash码
	- 在密码学中，hash函数往往被用于**身份认证**，证明某个文件就是我们期望的文件
		- 最基本的应用就是对某个文件求一个hash码，然后下载了该文件的人可以再求一次hash码确认下载的是正确的文件
		- 另一种应用是**提交策略(commit schemes)**，想提交某些数据，但是想过一段时间再揭晓实际数据。先不发送实际数据，但是告诉对方我已经准备好了你需要的数据，并发送给对方一个hash码，在确认可以向对方发送数据之后，将实际数据发送过去，对方可以根据hash码自行检查是否确实是之前生成的数据
- # 密钥派生函数
	- **[[$red]]==Key Derivation Functions==**，简称KDFs
	- **运行速度较低**(故意的)，为了防止线下暴力破解
	- 一个典型的KDF是PBKDF2
	- 类似于hash函数，也是在不同长度的输入下给出固定长度的输出，但是其输出一般用作其他加密算法的输入
	- 可以用于密码生成
- # 对称加密
	- [[$red]]==**Symmetric Cryptography**==
	- 用于完成密文的加解密
	- 使用如下的流程完成加解密
		- ``keygen()`` -> ``key``，密钥生成函数一般是随机化的
		- ``encrypt(plaintext: array<byte>, key)`` -> ``array<byte>``，生成密文（ciphertext）
		- ``decrypt(ciphertext: array<byte>, key)`` -> ``array<byte>`` 解密获得明文(plaintext)
	- 对于给定的使用加密函数加密的密文，很难判定其原来的明文，而给定密文和密钥，解密函数一定能解密获得正确的明文
	- 一个典型的对称加密算法是**AES**算法
- # 非对称加密
	- [[$red]]==**Asymmetric Cryptography**==
	- 使用如下流程完成加解密
		- ``keygen()`` -> ``(public key, private key)``，生成一对密钥，公钥和私钥。生成过程随机化
		- ``encrypt(plaintext: array<byte>, public key)`` -> ``array<byte>`` ，生成密文(ciphertext)
		- ``decrypt(ciphertext: array<byte>, private key)`` -> ``array<byte>``，解密获得明文(plaintext)
		- ``sign(message: array<byte>, private key)`` -> array<byte> ，生成签名(signature)
		- ``verify(message: array<byte>, signature: array<byte>, public key)`` -> ``bool`` 判断一个签名是否有效
	- 公钥可以直接明文发送到互联网上，所有人都可以知道公钥
	- 想和我建立联系的人可以使用公钥加密密文，然后发送给我，我用私钥解密
	- 因此非对称加密可以在不安全的信道上建立加密通信
	- 给了别人加密的能力，但使其不具备解密的能力
	- 对于sign 和 verify函数，在不持有私钥的情况下是很难伪造一个可以通过检验函数的
	- **ssh**认证也是用非对称加密