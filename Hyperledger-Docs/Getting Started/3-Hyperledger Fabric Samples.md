
# Hyperledger Fabric Samples

在终端执行以下命令，下载fabric-samples（示例）:

    git clone https://github.com/hyperledger/fabric-samples.git
    cd fabric-samples

## Download Platform-specific Binaries

Next, we will install the Hyperledger Fabric platform-specific binaries. This process was designed to complement the Hyperledger Fabric Samples above, but can be used independently. If you are not installing the samples above, then simply create and enter a directory into which to extract the contents of the platform-specific binaries.

下一步，下载Hyperledger Fabric平台的特定二进制文件，这个文件是为了补充上面示，也可以单独使用。如果没有samples，可以创建一个目录来提取特定的平台内容。

在目录下执行该命令:

    curl -sSL https://goo.gl/iX9dek | bash


>如果Curl过程中出错，还请升级Curl到最新版本，不要使用旧版本的Curl.

Curl下载并执行脚本，该脚本将下载特定的二进制文件，检索4个特定于平台的二进制文件:

        cryptogen,
        configtxgen,
        configtxlator, and
        peer

将会放到当前(cd fabric-samples)`bin`目录下.

添加环境变量：

    export PATH=<path to download location>/bin:$PATH
> /etc/profile
> export FABRICSAM=/home/hsukk/fabric/src/github.com/fabric-samples
>export GOROOT=/usr/local/go
>export PATH=$PATH:$GOROOT/bin:$FABRICSAM:/bin

最后脚本将下载docker容器，并在本地注册为 ‘latest’.

如果是`X86_64-1.0.0`，还请tag为`latest`。
