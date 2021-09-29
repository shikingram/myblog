{
  "title":"区块链学习（三）hyperledger & Fabric",
  "tags":[
    "blockchain"
  ],
  "date":"2021-09-29",
  "lastmod":"2021-09-29",
  "draft":"false",
  "author":"kingram"
}

## hyperledger简介
超级账本（hyperledger）是首个面向企业应用场景的分布式账本平台，包括了IBM,intel,Cisco,DAH,摩根大通,R3等在内的众多科技和金融巨头的贡献参与，在银行，供应链等领域得到了广泛的关注和发展。

## 社区组织结构

- 技术委员会（TSC）
- 管理董事会（Governing Board）
- Linux基金会（LF）

## 项目介绍
2015年12月，开源世界Linux基金会牵头联合30家初始企业共同成立了Hyperledger联合项目成立，作为一个联合项目，旗下由面向不同的场景的子项目构成，包括Fabric、Sawtooth、Iroha、Blockchain Explorer、Cello、Indy、Composer、Burrow八大顶级项目。

所有项目都遵循ApacheV2许可，并约定共同遵守如下的基本原则：
- 重视模块化设计
- 重视代码可读性
- 可持续化的演化路线

## 社区工作流
在社区开发中，需要了解一下社区协作过程中使用的工具
- linux foundation id
- jira--任务和进度管理
- Gerrit--代码仓库和Review管理
- RocketChat--在线沟通

## Fabric
Hyperledger Fabric是一个提供分布式账本解决方案的平台。他由模块化架构支撑，并具备极佳的保密性，可伸缩性，灵活性，和可扩展性。

Hyperledger Fabric被设计成支持不同的模块组件直接插拔启用，并能适应在经济生态系统中错综复杂的各种场景。

## Fabric应用场景
- 商业积分，利用区块链多方发行扩大参与者、使积分自由流通，吸引用户再次消费。
- 跨境支付与结算，减少机构之间的信任成本，降低手续费。
- 数据存证，版权保护，鉴别数据真伪。

## Fabric名词解释
- 成员服务，用来在许可的区块链网络上认证，授权和管理身份
- 排序或者共识服务，确认交易并将交易排序放入block
- 账本，交易状态的持久化
- 节点，网络实体用来维护账本，执行合约的容器
- sdk,用来和区块链网络进行交互

## Fabric的交易流程

![fabric交易过程](/img/blockchain_s/fabric_work.png)

- 应用向单个或多个peer节点发送对交易的背书请求
- 背书节点执行chaincode，但并不将结果提交到本地账本，只是将结果返回给应用
- 用用收集所有的背书节点的结果后，将结果广播给order
- order 执行共识过程，并打包成blocke ,通过消息通道批量的将block发布给peer节点
- 各个peer节点验证交易，并提交到本地账本中
















