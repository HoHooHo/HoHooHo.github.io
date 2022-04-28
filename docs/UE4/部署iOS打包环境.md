## Windows远程打包
XCode 13.3.1
Unreal Engine 4.26

### 1、RemoteToolChainPrivate

!!! fail "fail"
    [Remote] Using private key at D:\RemoteToolChainPrivate.key
        ERROR: Unable to determine home directory for remote user. SSH output:

!!! help "help"
    需在打包机（Windows系统）上生成  RemoteToolChainPrivate.key  文件

### 2、-Werror

!!! fail "fail"
    error: variable 'xx' set but not used [-Werror,-Wunused-but-set-variable]

!!! help "help"
    XCode版本问题，可以使用 Xcode 13.2.1 版本


### 3、The Legacy Build System

!!! fail "fail"
    error: The Legacy Build System will be removed in a future release. You can configure the selected build system and this deprecation message in File > Workspace Settings.

!!! help "help"
    Unreal Engine 版本问题，可以使用 4.27



## Mac打包
jdk-11.0.15_osx-x64
Jenkins 2.332.2 LTS

Jenkins中配置 svn 账号验证不通过，后台报错

!!! fail "fail"
    SEVERE	h.s.SubversionSCM$ModuleLocation$DescriptorImpl#checkCredentialsId: svn: E175002: SSL handshake failed: 'The server selected protocol version TLS10 is not accepted by client preferences [TLS13, TLS12]


!!! help "help"
    在文件 /Library/Java/JavaVirtualMachines/jdk-11.0.15.jdk/Contents/Home/conf/security/java.security 中找到 jdk.tls.disabledAlgorithms，删除其中的 TLSv1、TLSv1.1、3DES_EDE_CBC 后重启系统。
    java.security文件没有权限修改，可以将其复制到有权限修改的目录，改完之后再覆盖该文件。