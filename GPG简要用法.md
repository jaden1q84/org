# GPG简要用法
Jaden Liang
11/27/2016
## GPG简介
GPG是用于加密和签名的工具，基于OpenGPG标准实现。基础原理是RSA和对称加密技术。
## 生成密钥对
使用GPG做非对称加密，首先要生成一对密钥对。密钥对包括公钥和私钥，私钥是由自己持有的，公钥则公开发布。用公钥加密的文件只能使用对应的私钥解密，从而达到加密的效果。私钥用于对文件进行签名，签名文件只能使用公钥验证真伪。生成密钥对的命令如下：
```bash
~ gpg --gen-key
```
这里会问几个问题：
* 密钥对的位数，默认是2048，越高越安全。
* 密钥对的期限，默认不超期
* 名字、email、注释，本地密钥对是通过名字识别的。
* 该密钥对的口令密码。
最后命令gpg需要产生足够的随机数据，这时可以动一下鼠标或键盘以便随机数足够随机。
## 查看密钥对
### 查看公钥
    
```bash
~ gpg --list-key
/home/test/.gnupg/pubring.gpg
-----------------------------
pub   4096R/D8D4723A 2016-11-27
uid       [ultimate] tester (test key pair) <tester@test.com>
sub   4096R/61FD3E7F 2016-11-27
```
### 查看私钥
    
```bash
~ gpg --list-secret-key
/home/test/.gnupg/secring.gpg
-----------------------------
sec   4096R/D8D4723A 2016-11-27
uid                  tester (test key pair) <tester@test.com>
ssb   4096R/61FD3E7F 2016-11-27
```
公钥和私钥是通过 `D8D4723A/61FD3E7F` 来标识。
## 加解密
### 通过公钥加密
    
通过gpg命令加密需要指定接收人的公钥，通过 `-r` 参数指定。
```bash
~ gpg -e -r tester testfile
```
执行命令后得到 `testfile.gpg` 加密文件。
    
### 通过私钥解密
    
通过gpg命令解密时，本地必须要有加密时公钥对应的私钥，并输入私钥的解密口令。
```bash
~ gpg -d -o testfile testfile.gpg
```
如果不指定 `-o` 参数，`gpg` 默认输出到标准输出。
## 文件签名和校验
`gpg` 可以使用私钥对文件生成签名文件，接收人可以使用公钥对签名文件进行验证。
### 生成签名文件
    
通过 `-s -b` 参数可以生成独立的签名文件，`-u` 制定签名用的私钥本地用户。
```bash
~ gpg -s -b -u tester testfile
```
生成 `testfile.sig` 文件，但这样生成的签名文件是二进制文件，不方便人工检查。可以加 `-a` 指定生成ascii格式的签名文件。

```bash
~ gpg -a -s -b -u tester testfile
```
生成 `testfile.asc`
### 校验文件
    
通过 `--verify` 参数执行文件签名校验，注意验证的文件参数是 `.asc` 或 `.sig` 文件，并且本地需要要待校验的文件。

```bash
~ gpg --verify testfile.asc
gpg: assuming signed data in 'testfile'
gpg: Signature made 日 11/27 21:55:34 2016 CST using RSA key ID D8D4723A
gpg: Good signature from "tester (test key pair) <tester@test.com>" [ultimate]
```
输出如上说明校验通过。
## 密钥导出导入
本地生成密钥后，需要导出公钥发布或者导出公私钥文件用于备份。
### 导出密钥
要导出公钥，首先要知道计划导出公钥的ID，执行 `gpg --list-key` 查询本地所有公钥，找到目标用户的公钥ID，如：

```
~ gpg --list-key
/Users/lzj/.gnupg/pubring.gpg
-----------------------------
pub   4096R/D8D4723A 2016-11-27
uid       [ultimate] tester (test key pair) <tester@test.com>
sub   4096R/61FD3E7F 2016-11-27
``` 

获取 `tester` 的ID为 `D8D4723A`。然后使用 `--export` 参数导出该公钥。如果想导出ascii格式的公钥再加上 `-a` 参数，`gpg` 默认输出到标准输出，可以通过加 `-o` 参数指定导出的文件名。

```
~ gpg -a --export D8D4723A
```

要导出私钥，则把 `--export` 换成 `--export-secret-keys`。导出的私钥要妥善安全保管，避免泄露。


### 导入密钥
要导入公钥，通过 `--import` 参数指定要导入的文件即可。然后通过 `--list-key` 查看导入结果。

