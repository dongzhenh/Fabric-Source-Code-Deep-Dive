#4.4 MSP
Fabric提供了Membership Service Providers（MSP）组件，它将各个区块链节点进行证书签发／验证以及用户认证背后的加密机制和协议进行了抽象，提供了成员操作的模块化和不同成员结构间的互操作性。

每个Peer节点和Orderer节点都需要有一个MSP ID和其对应的一组文件。通常建议每个参与组织使用一个MSP，Orderer节点／集群作为一个独立的参与组织使用一个的MSP。每个MSP对应的文件组包含这个组织的CA根证书，
中间CA证书，每个组织成员（Peer）自己的身份证书，证书吊销列表（CRLs），TLS根证书等。Fabric的cryptogen工具可根据配置文件为区块链网络生成这一组文件，用户也可自行生成这些文件。

区块链网络管理员可在配置文件（config.yaml）中指定每个组织的MSP ID和其对应文件组的路径，Fabric的configtx工具根据config.yaml中的配置生成区块链网络的创世区块（Orderer启动时需提供）和某一通道的创始块
（创建通道时需提供）。这样创世区块中带有了节点根证书、管理员身份证书，管理policy等信息，进而可以验证某一个节点或者用户针对区块链网络或某一通道进行的操作是否合法。

```      
 	// Init the MSP
	var mspMgrConfigDir = config.GetPath("peer.mspConfigPath")
	var mspID = viper.GetString("peer.localMspId")
	var mspType = viper.GetString("peer.localMspType")
	if mspType == "" {
		mspType = msp.ProviderTypeToString(msp.FABRIC)
	}
	err = common.InitCrypto(mspMgrConfigDir, mspID, mspType) 
```     

MSP的核心代码分布如下：

* fabric/msp
 + msp.go : 定义MSP，MSPManager，Identity，SigningIdentity等主要接口
 + mspimpl.go : 实现MSP接口，结构为bccspmsp
 + mspmgrimpl.go : 实现MSPManager接口，结构为mspManagerImpl
 + identities.go : 实现Identity，SigningIdentity接口
 + configbuilder.go : 提供读取证书文件并将其组装成MSP等接口需要的数据结构，转换配置结构体（由FactoryOpts->MSPConfig）等工具函数
 + mgmt/mgmt.go : localMsp和mspMap都在这个文件，还有多个管理函数
 
 
