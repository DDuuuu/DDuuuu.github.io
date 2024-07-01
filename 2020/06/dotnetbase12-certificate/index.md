# DotNet基础 证书加密签名


## 网络威胁

1.窃听：网络传输是在公共信道上进行的，特别是HTTP传输大多以明文传输，黑客进行窃听，获取敏感信息。

![img](/dotnet/zhengshuqieting.png)

2.伪装：这个基本每个人都遇到过，早期诈骗，说你账户被盗要求你登录改密码，会进入一个与官网相似的改密码页面来获取你的真实密码。

![img](/dotnet/zhangshuweizhuang.png)

3.篡改：信息在提交到服务器之前被非法修改，黑客可以盗取用户的Session、Cache信息，来修改提交到服务器的请求信息。

![img](/dotnet/zhuangshuchuangai.png)

4.抵赖：用户可以否定自己曾经做过的事情。在日常生活中也会遇到同样的问题，比如领导明明安排的一个任务，但你可能忘记了就说不知道，没人通知你，这就叫抵赖

![img](/dotnet/zhengshudilai.png)

## 基本概念

证书是针对上述网络威胁建立起来的解决方案，基本概念：

1. CA(Certificate Authority)：数字证书认证中心。发放、管理、废除数字证书的机构，CA的作用是检查证书持有者身份的合法性，并签发证书（在证书上签字），以防证书被伪造或篡改，以及对证书和密钥进行管理。
2. 证书有以下信息：证书颁布机构，证书的有效期，公钥，私钥，签名及所用算法，指纹和指纹算法。
   1. 证书颁布机构（Issuer）：指出是什么机构发布的这个证书，也就是指明这个证书是哪个公司创建的(只是创建证书，不是指证书的使用者)。
   2. 证书有效期（Valid from , Valid to）：证书的使用期限。 过了有效期限，证书就会作废，不能使用了
   3. 主体（Subject）：证书是发布给谁的，或者说证书的所有者
   4. 签名所使用的算法（Signature algorithm）：指的这个数字证书的数字签名所使用的加密算法。
   5. 指纹及指纹算法（Thumbprint, Thumbprint algorithm）：这个是用来保证证书的完整性的，也就是说确保证书没有被修改过。 其原理就是在发布证书时，发布者根据指纹算法(一个hash算法)计算整个证书的hash值(指纹)并和证书放在一起，使用者在打开证书时，自己也根据指纹算法计算一下证书的hash值(指纹)，如果和刚开始的值对得上，就说明证书没有被修改过，因为证书的内容被修改后，根据证书的内容计算的出的hash值(指纹)是会变化的。

### 证书的特性

1. 保密性（**Privacy**）：确认信息的保密，不被窃取。
2. 鉴别与授权（**Authentication &amp; Authorization**）：确认对方的身份并确保其不越权
3. 完整性（**Integrity**）：确保你收到信息没有被篡改
4. 抗抵赖性（**Non-Repudiation**）：有证据保证交易不被否认 

### 证书的常见形式

.CER，文件是二进制格式，只保存证书，不保存私钥。公钥，指纹，加密算法

#### cer证书内容

![1610930135488](/dotnet/1610930135488.png)

.PEM，一般是文本格式，可保存证书，可保存私钥。

.PFX，二进制格式，同时包含证书和私钥，一般有密码保护。

.JKS，二进制格式，同时包含证书和私钥，一般有密码保护。

## 应用与实现

### 证书创建

微软官方给了一个证书创建帮助类&lt;https://docs.microsoft.com/en-us/archive/blogs/dcook/creating-a-self-signed-certificate-in-c&gt;

可以使用其很方便的创建一个证书

```c#
        /// &lt;summary&gt;
        /// 创建证书
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public bool CreateCertificate()
        {
            var commonName = m_CertificateCommonName;
            var password = m_Password;
            var path = m_CertificatePath;
            var certificateName = commonName;
            if (!commonName.StartsWith(&#34;CN=&#34;, StringComparison.OrdinalIgnoreCase))
                    certificateName = &#34;CN=&#34; &#43; commonName;
            byte[] certificateData = Certificate.CreateSelfSignCertificatePfx(certificateName, //host name
                DateTime.Now, //not valid before
                DateTime.Now.AddYears(5), //not valid after
                password);
            var filename = Path.Combine(path, commonName);
            if (!filename.EndsWith(&#34;.pfx&#34;))
            {
                filename = filename &#43; &#34;.pfx&#34;;
            }
            using (BinaryWriter binWriter = new BinaryWriter(File.Open(filename, FileMode.Create)))
            {
                binWriter.Write(certificateData);
                binWriter.Flush();
            }

            if (TryGetCertificate(out X509Certificate2 cert))
            {
                selfCertificate = cert;
            }
            RunInfo = &#34;创建成功&#34;;
            return true;
        }
```

导出cer证书

```c#
        /// &lt;summary&gt;
        /// 尝试导出cer证书文件，也就是公钥证书
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public bool TryExportCerFile()
        {
            if (string.IsNullOrEmpty(m_BinaryExportPath) || string.IsNullOrEmpty(m_BinaryExportName))
            {
                RunInfo = &#34;请输入参数信息&#34;;
                return false;
            }
            //find the certificate
            byte[] cerByte = selfCertificate.Export(X509ContentType.Cert);
            var filename = Path.Combine(m_BinaryExportPath, m_BinaryExportName);
            if (!filename.EndsWith(&#34;.cer&#34;))
            {
                filename = filename &#43; &#34;.cer&#34;;
            }
            using (BinaryWriter binWriter = new BinaryWriter(File.Open(filename, FileMode.Create)))
            {
                binWriter.Write(cerByte);
                binWriter.Flush();
            }

            RunInfo = &#34;导出成功&#34;;
            return true;
        }
```

## 证书加密

![img](/dotnet/zhengshujiami.png)

```c#
/// &lt;summary&gt;
/// 公钥加密
/// &lt;/summary&gt;
public void PublicKeyEncryptionMethod()
{
    if (string.IsNullOrEmpty(m_PlainText))
    {
        RunInfo = &#34;请输入需要加密的信息&#34;;
        return;
    }

    //公钥加密
    byte[] infos = Encoding.UTF8.GetBytes(m_PlainText);
    RSACryptoServiceProvider myRSACryptoServiceProvider =
        (RSACryptoServiceProvider) selfCertificate.PublicKey.Key;
    Byte[] Cryptograph = myRSACryptoServiceProvider.Encrypt(infos, false);

    //所以明文不能把128字节占满，实际测试，明文最多为117字节，留下的空间用来填充随机数。
    publicKeyEncryption = Cryptograph;
    StringBuilder s =new StringBuilder();
    foreach (var b in Cryptograph)
    {
        s.Append(b.ToString(&#34;X2&#34;) &#43; &#34; &#34;);
    }

    s.Append(&#34;总字节数:&#34; &#43; Cryptograph.Length);
    PublicKeyEncryption = s.ToString();
}
```

```c#
/// &lt;summary&gt;
/// 使用私钥解密
/// &lt;/summary&gt;
public void PrivateKeyDecryptionMethod()
{
    if (publicKeyEncryption == null)
    {
    RunInfo = &#34;请先进行公钥加密&#34;;
    return;
    }
    //私钥进行解密
    RSACryptoServiceProvider myRSACryptoServiceProvider = (RSACryptoServiceProvider)selfCertificate.PrivateKey;

    byte[] plaintextByte = myRSACryptoServiceProvider.Decrypt(publicKeyEncryption, false);

    PrivateKeyDecryption = Encoding.UTF8.GetString(plaintextByte);
}
```

## 证书电子签名

![img](/dotnet/dianziqianming.png)

```c#
/// &lt;summary&gt;
/// 私钥签名
/// &lt;/summary&gt;
public void PrivateKeyEnSignatureMethod()
{
	if (string.IsNullOrEmpty(m_PlainText))
	{
		RunInfo = &#34;请输入需要加密的信息&#34;;
		return;
	}
	//明文数据
	byte[] infos = Encoding.UTF8.GetBytes(m_PlainText);
	//私钥进行摘要算法计算，并将计算值用私钥加密
	RSACryptoServiceProvider myRSACryptoServiceProvider =
		(RSACryptoServiceProvider)selfCertificate.PrivateKey;
	byte[] signs= myRSACryptoServiceProvider.SignData(infos, &#34;MD5&#34;);
	string signdata = System.Convert.ToBase64String(signs);
	privateKeyEnSignature = signs;
	StringBuilder s = new StringBuilder();
	foreach (var b in signs)
	{
		s.Append(b.ToString(&#34;X2&#34;) &#43; &#34; &#34;);
	}

	s.AppendLine(&#34;总字节数:&#34; &#43; signs.Length);

	s.AppendLine(&#34;ConvertBase64:&#34; &#43; signdata);

	PrivateKeyEnSignature = s.ToString();
}

```

```c#
/// &lt;summary&gt;
/// 验证签名方法
/// &lt;/summary&gt;
public void PublicKeyDeSignatureMethod()
{
	if (privateKeyEnSignature == null)
	{
		RunInfo = &#34;请先进行私钥加密&#34;;
		return;
	}
	//原始数据
	byte[] infos = Encoding.UTF8.GetBytes(m_PlainText);
	//获取公钥
	RSACryptoServiceProvider myRSACryptoServiceProvider = (RSACryptoServiceProvider)selfCertificate.PublicKey.Key;
	//验证数据,用公钥对加密的摘要值进行解密，并于原数据计算的摘要值进行比对
	if (myRSACryptoServiceProvider.VerifyData(infos, &#34;MD5&#34;, privateKeyEnSignature ))
	{
		PublicKeyDeSignature = &#34;验证成功&#34;;
		return;
	}
	PublicKeyDeSignature = &#34;验证失败&#34;;
}

```

![1591103031140](/dotnet/1591103031140.png)

## 参考

&lt;https://www.cnblogs.com/yank/p/DigitalCertification.html&gt;

&lt;https://www.cnblogs.com/yank/p/3533998.html&gt;

&lt;https://www.cnblogs.com/chnking/archive/2007/08/18/860983.html&gt;


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2020/06/dotnetbase12-certificate/  

