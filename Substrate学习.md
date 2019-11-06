# Substrate学习

### 多挖的PoC设想

当前做区块链，发行的Token，相对于比特币都是在一个非强共识的前提下来发展。比特币的共识不断强大起来，十来年不断吸引新的人进来将比特币的流通量减少，比特币的价格才能不断得到支撑。如果要做一个新的Token，也要遵循这样基本的逻辑，那就是要让流通量得到有效的减少，从而让币价上涨，从而不断强化共识，让弱共识转变成强共识。PoC是非常不错的方向，有下面几个不可忽略的优点：

- PoC是经历过市场验证的成功案例。
- 挖矿软件是现成的。
- P盘软件是现成的。
- 支持智能合约的PoC是巨大的创新。

PoC挖矿软件：https://github.com/PoC-Consortium/scavenger
P盘工具：https://github.com/PoC-Consortium/engraver
PoC的挖矿文档：https://burstwiki.org/en/mining/
PoC的BHD实现（C++）：https://github.com/btchd/btchd/blob/master/src/poc/poc.cpp

CPoC的挖矿模式，比如每分钟出一个区块，奖励原生代币，通过智能合约能发行新的代币，账号可以抵押多个代币，然后奖励可以是多个代币。智能合约发行新的代币，需要锁仓一定比例的代币，比如需要锁仓的量是现在已发行原生代币的10%，锁仓期为2年，可以逐渐释放出来，2年全部释放完。

发行新的代币所需参数：

- 代币symbol
- 初始区块奖励量 T
- 每T抵押代币数量 M
- 每次减半的区块数量（也就是减半周期）P

> 合约发行代币需要记录区块高度H'（刻录区块高度），当前区块高度H减去刻录区块高度除减半周期区块数量，向下取整后得到减半次数n，当前奖励代币数量为 T/(2^n)

验证是否有足够抵押代币的逻辑：根据过去一段时间（难度调整周期），比如过去7天的出块数量，推算出其出块占比，根据全网挖矿难度推算出总算力，然后可算出其有多少T的算力，然后根据每T抵押代币数量计算出所需要的抵押代币数量，判断是否抵押足够。

初始出块无抵押攻击：当接入的算力第一次出块，没有历史出块数据可考察，无法判断其抵押代币是否足够，按照上面逻辑，是不需要抵押的。可以直接锁住，等到第二次出块的时候，验证其抵押足够时，一起发给矿工。

##### 奖励规则设计：

- 抵押不够，奖励出块奖励的30%。剩下70%归入国库。
- 共轭模式，如果两个代币抵押都足够，从国库中拿出出块奖励的1%作为共轭奖励，前提条件是国库中要有足够的代币。任何两个代币都能够共轭挖矿来获取共轭奖励。

> 共轭设计的好处是拉动后来的币，为了共轭奖励，更多的发币方将获得代币的买盘力量，付出的代价是要更多的原生代币来锁仓获得智能合约发币权力。

##### 治理设计

- 原生代币的每T抵押数量，社区提案，全民投票通过决定。
- 智能合约发行新共轭代币，需要锁仓代币占总发行量比例，社区提案，全民投票通过决定。
- 智能合约发行新共轭代币，锁仓期长短，社区提案，全民投票通过决定。
- 抵押不足，罚扣比例，社区提案，全民投票通过决定。
- 共轭奖励基准（默认1%），社区提案，全民投票通过决定。


##### 版本迭代规划

- 第一个版本，能挖矿的PoC的链
- 第二个版本，需要抵押的PoC的链
- 第三个版本，出块奖励和国库模块实现
- 第四个版本，智能合约和锁仓规则实现
- 第五个版本，多挖和共轭规则实现
- 第六个版本，治理委员会，选举和治理规则实现



### 创建新项目

	curl https://raw.githubusercontent.com/paritytech/substrate-up/4f3d476d2271a1cae6014a22255d0c7aa85692e7/substrate-node-new -sSf | sh -s conjugate-poc Tom

创建多个节点网络：

	cargo run --release -- \ 
	--base-path data/node1 \
	--chain=local \
	--alice \
	--node0-key 00000000000000000000000000000000000000000000000000000001 \
	--telemetry-url ws://telemetry.polkadot.io:1024 \
	--validator
	
创建第二个节点网络：

	cargo run --release -- \ 
	--base-path data/node2 \
	--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/QmaMHd82KFsVV4d9tNsACmdBfzuFoKw1fBtA9XFaBeuKY5 \
	--chain=local \
	--bob \
	--port 30334 \
	--telemetry-url ws://telemetry.polkadot.io:1024 \
	--validator
	
Chain client created in conjugate-poc.
To start a dev chain, run:
$ conjugate-poc/target/release/conjugate-poc --dev
To create a basic Bonds UI for your chain, run:
$ substrate-ui-new conjugate-poc
To push to a newly created GitHub repository, inside conjugate-poc, run:
$ git remote add origin git@github.com:myusername/myprojectname && git push -u origin master

### 命令行参数

	--chain local
	--chain dev
	--chain fir
	
	--validator
	--node-key
	--base-path
	--port
	--bootnodes multiaddr
	--telemetry-url url
	--name Silver
	--ws-port 9944
	--rpc-port 9933
	--ws-external
	--rpc-external
	--rpc-cors all
	--pruning mode
	--force-authoring
	--execution native
	--log trace
	--log sync,afg=trace
	--keystore-path
	
	purge-chain
	--chain <chain name or path>
	build-spec
	--chain <chain name or path>
	--raw
	
	
### Substrate 架构

![](https://upload.cc/i1/2019/09/23/RUcqxb.jpeg)

- BABE/Grandpa混合共识机制
- p2p连接和广播系统
- 通用交易池
- Metadata原数据系统
- SRML（Substrate Runtime Module Library）


#### Runtime

- 数据结构的定义
	- Block Header区块头
	- Block 区块
	- Extrinsic 外部消息
		- Transaction 用户交易（用户需要签名）
		- Inherent 固有消息 （用户不需要签名）

- Runtime的API接口
	- version
	- execute_block
	- initilize_block
	- metadata
	- apply_extrinsic
	- finalize_block
	- inherent_extrinsics
	- check_inherents
	- random_seed
	- validate_transaction
	- offchain_worker
	- authorities
	- sign
	- verify

- SRML
 	- 核心模块
 		- Executive
 		- System
 	- 共识机制
 		- Babe
 		- Authorship
 		- Finality Tracker
 		- Grandpa
 		- Session
 		- Offences

 - 自治管理
		- Council
		- Democracy
		- Sudo
		- Treasury

- 时间
		- Timestamp

- 资金账号管理
		- Indicies
		- Balances
		- GenericAsset
		- Staking
	
- 智能合约
		- Contracts

- 辅助
		- Supoort
		- Metadata

	
### Substrate Module组成

- Trait
		- 定义模块相关类型
- decl_event
		- 定义模块事件
- decl_storage
		- 定义存储数据
- decl_module
		- 包括了dispatchable method可外部调用的函数
- on_initialize/on_finalize 区块初始/结束接口
- decl_error
		- 定义了错误

> todolist： substrate下面的srml的example阅读
> https://substrate.dev 学习

#### 笔记补充

- Substrate：新一代的区块链开发框架，我称之为区块链操作系统。基于WebAssembly
，Libp2p和Grandpa算法的组合。
- ink!：基于Rust语言的合约编写平台，构建与Substrate合约模块兼容的WebAssembly智能合约。
- Polkadot-JS/API：通过RPC调用与基于Substrate的链进行交互。
- Polkadot-JS/Apps：用于与Substrate节点的所有功能进行交互的基本UI。跟Polkadot-JS/API结合。
- oo7-substrate：基于 ReactJS 的 API，通过 RPC 调用来与基于 Substrate 的链进行交互。 也被称为“Bonds”库
- Substrate UI：基于 ReactJS 的基础 UI，用于快速构建 Substrate 应用链的用户界面。 由 oo7-substrate 提供支持。

#### Substrate解耦合

Substrate作为区块链操作系统，将系统分为两个部分：

- Substrate Core
- Runtime

Substrate Core：链的系统基础部分

- 共识系统
- 通信系统p2p
- 交易池
- rpc
- Metadata存储
- 。。。

Runtime

- 资金管理，转账
- 账户系统
- 合约虚拟机
- 民主投票
- 权益计算
- 国库
- 。。。

对运行结果进行共识的功能部分都应该归于Runtime。”链上功能“来描述Runtime，需要对结果进行全网共识。

对于某个功能，若只改动一个节点的代码对于所有的逻辑运行的结果与其他不该动的节点运行的结果相同，则认为这个部分应该放在Runtime之外，如果运行结果不同，则认为应该放在Runtime之内。

#### 区块链系统升级

既然Substrate作为区块链的操作系统，就能有系统升级，而这个功能来自Runtime的抽象。

中心化的互联网开发，即便是中心组织来进行升级，也需要移动端的APP自己进行升级，而且还会有应用版本碎片化的问题。

去中心化的区块链开发，区块链第一代BTC需要分叉造成社区分裂，第二代ETH也是困难很大，对开发迭代的速度是很大的打击，2.5代的EOS有部分中心化的优势，但也有硬分叉的恶性事件，区块链系统升级是”全局升级“。

区块链系统升级需要解决一个兼容老数据的问题，这些老数据去中心化保存，如果同步，需要区分不同高度下的系统版本。

Substrate的升级方案，”链上代码“思维，整个Runtime做成可更新的组件，强制所有节点运行最新的Runtime代码。

- native代码，本地运行，快速高效。
- wasm链上代码，wasm直接部署到链上，所有人可获取，wasm通过构建以恶搞wasm的运行时环境执行。

关键是如何定义最新的代码，需要通过”民主提议“，”sudo控制权限“，”开发者自定义一种部署条件“等方式进行。很明显，民主提议适合联盟链，sudo控制权限适合私有链和本地开发链，自定义适合公链。





#### 核心的数据类型



- Hash，给数据做摘要用的，就是一个256位的长度。
- BlockNumber，从祖先区块开始到当前区块的数量，一个32位的长度。
- DigestItem，这种数据类型是这两种数据类型组成的Item，一种是跟共识和变更跟踪有关的许多“硬连线”，另外一种是与运行时特定模块有关的任何数目的“软编码”变量。
- Digest，基本上只是DigestItem的序列化s，将对与轻节点相关的所有信息进行编码。
- Header，区块头信息，包括父区块hash，存储根和外部数根，摘要和区块高度。
- Extrinsic，一种表示区块链外部可识别的数据的类型。通常涉及一个或者多个签名，以及某种编码指令（例如，用于转移资金所有权或者调用智能合约）。
- Block，基本上只是区块头和一系列外部消息的组合，然后使用特定的hash算法。


#### 外部消息

外部消息不仅仅是交易，可以分为两大类，一类就是交易，另外一类是inherent extrinsic或者inherents，翻译成固有。

不需要被用户签名的，比如时间戳就是固有。

区块和外部消息世界状态树

外部消息被打包进一个区块，在运行时中被定义后执行打包。在区块头中有一个外部消息根，其实是外部消息的一个摘要，两个简单的作用，第一个是防止恶意篡改，第二个是轻节点进行验证，不需要进入到外部消息内部验证所有消息细节。

在SRML中的外部消息

SRML提供两种基本外部消息：```UncheckedExtrinsic```,```UncheckedMortalExtrinsic```.


#### 交易的生命周期

交易事务有两种来源：

- 从网络上接收到区块，然后执行execute_block，整个区块成功或者失败。
- 自己生产一个区块并广播。
	- 网络上监听交易。
	- validate_transaction函数验证每个交易，并将交易放入交易池中。
	- 交易池对事务进行排序，并返回准备好包含在区块中的事务，使用现成的交易来构造一个BlockBuilder。
	- BlockBuilder使用apply_extrinsic函数执行事务并将状态更改应用内存进行保存。
	- 构造的区块广播到网络中，其他节点接收到区块执行execute_block函数。

#### Offchain Worker

脱链操作，一般通过预言机，将外部服务通过监听区块链事件并相应地触发任务，这些任务完成后，将结果使用事务提交回区块链。

链下工人允许系统执行长时间运行且可能不确定的任务。在Substrate运行时之外具有自己的wasm执行环境。确保来了区块生产不受长期运行任务的影响，链下工人可以轻松访问链上状态进行计算。

链下工人的结果不需要定期达成共识。链上验证机制，以确定哪些信息进入链上。

![](https://substrate.dev/docs/assets/off-chain-workers.png)

相比于预言先知机，链下工人具有如下的优点：

- 作为Substrate节点的一部分，链下工人提供链更多的分散性。
- 借助沙盒式wasm执行环境，链下工人可以完全安全，同时仍可以使用外部消息与运行时进行无缝集成。
- 无需维护外部“胶水式”的服务和基础设施，减少了节点运营商的维护和基础设施成本。
- 链下工人的代码也保存链上，允许通过标准的治理机制和无缝升级进行链外逻辑更新。

链下工人API：

- 加密和解密
- 签名
- 本地存储-获取并更新
- HTTP请求
- 密钥生成
- 随机数生成
- 时间戳

#### SRML体系结构

![](https://substrate.dev/docs/assets/srml-arch.png)

有四个主要的框架组件来支持运行时模块。

- 系统模块

系统模块提供链低级别API，将其视为SRML的"std"库，系统模块为Substrate运行时定义了所有核心类型，还定义了外部事件（成功/失败）。

- 执行模块

执行模块充当运行时编排层，在运行时将传入的外部调用分派到各个模块。

- 支持宏

支持宏是Rust宏，帮助实现一个模块中最常见的组件的集合。这些宏在运行时扩展以生成类型（调用，模块，存储，事件等），运行时使用这些类型与模块进行通信。一些常见的支持宏```decl_storage```,```decl_module```,```decl_event```,```ensure```，等。

- 运行

运行时将所有框架组件和模块组合在一起，它扩展了支持宏，以获取每个模块的类型和特征实现。它还调用执行模块将调用调度到各个模块。

SRML附带的模块：

- Assets
- Aura
- Balances
- Consensus
- Contracts
- Council
- Democracy
- Finality Tracker
- Grandpa
- Indices
- Session
- Staking
- Sudo
- Timestamp
- Treasury

> asset模块阅读


### 测试

	cargo test -p substrate-kitties-runtime
	
如果要在测试的时候打印东西出来：

	cargo test -p substrate-kitties-runtime -- --nocapture owned_kitties_can_append_values
	
也可以直接测试所有

	cargo test --all
	
### 依赖注入

- 保证模块的抽象化和重用性
- 去除模块之间不必要的耦合
- 依赖于Trait而不是Module
- 更加方便的测试

### balances模块

Balances模块提供以下方法：

- 获取和设置free balances。
- 索取总的，保留的和未保留的balances。
- 把保留的balances发送回某个存在的受益人账号。
- 在账户间进行balance的转账，当然他们要没有被reserved。
- 惩罚掉一个账号的balance。
- 账号创建和移除。
- 管理总的发行量。
- 设置和管理资产锁。

balances模块的术语：

- Existential Deposit：存现存款，开设或保留账号所需的最低余额，这可以防止“尘埃账号”来填满存储。
- Total Issuance：总发行量，系统中存在的货币的单位总数。
- Reaping an account:通过重置账号来删除账号的行为，在其平衡设置为零之后发生。
- Free Balance:自由余额，余额中没有被保留的部分。自由余额对于大多数操作来说是唯一重要的余额。当自由余额低于存现存款的时候，账号的大部分功能将被移除。当它和保留余额都被删除时，该账号就被认为是死的。
- Reserved Balance:保留余额，保留余额仍属于账号持有人，但已暂停。保留余额仍然可以被惩罚掉，但只有在所有的自由余额都被惩罚掉之后。如果保留余额低于存现存款，那么它和任何相关的功能都将被删除。当它和自由余额都被删除时，这个账号就被认为是死的。
- Imbalance:不平衡，某些资金credited或者debited没有相等或相反的会计平衡（即发行总量和所有账号余额之和之间的差额）的情况。导致不平衡的函数将返回一个可以在运行时逻辑中管理的`Imbalance`的trait对象。（如果不平衡只是简单地减少了，它应该自动保留任何账簿记录，例如发行总额。）
- Lock:冻结指定金额的账号自由余额，直到指定的区块号为止。多个锁总是对相同的资金进行操作，所以这些锁“overlay”而不是“stack”。
- Vesting:兑现，类似于锁，这是另外一种情况，但其能够独立的随着时间线性减少流动性约束。


### balances实现

Balances模块提供了下面的一些实现traits，如果这些traits提供的方法是开发者需要的，那么开发者就尽量要避免跟Balances模块耦合。

- Currecy：提供处理一个可互换资产的系统。
- ReservableCurrency：处理一个账号中被保留下来的资产。
- LockableCurrency：处理允许流动资金限制的帐户的功能。
- Imbalance：处理在总发行量和账号余额之间出现的不平衡，当创建一个新的资产（例如奖励）或者销毁一些资产（手续费）需要用到这个功能。
- IsDeadAccout：判断一个账号是否用过。

## Interface

### Dispatchable Functions

- `transfer`:从一个账号的可流动的自由余额转账到另外一个账号。
- `set_balance`:给一个账号设定余额，这个操作需要ROOT用户。

### Public Functions

- `vesting_balance`:获取当前正在使用的金额，并且不能从该帐户中转移。


### Signed Extensions

这个balances模块定义了如下的扩展：

- [`TakeFees`]：根据交易的长度和重量来相应比例的消耗交易费。


## Staking模块

Staking模块被网络管理维护人员用来管理资产用来抵押。

抵押模块是一组网络维护人员（可以称为authorities,也可以叫validators）根据那些自愿把资产存入抵押池中来选择的方法模块。这些在抵押池中的资金，正常情况下会获得奖励，如果发现维护人员没有正确履行职责，则会被没收。


### 抵押模块术语

- 抵押：将资产锁定一段时间，使其面临大幅惩罚损失掉的风险，从而使其成为一个有回报的网络维护者的过程。
- 验证：运行一个节点来主动维护网络的过程，通过出块或保证链的最终一致性。
- 提名：将抵押的资金置于一个或者多个验证者背后，以分享他们所接受的奖励和惩罚的过程。
- 隐藏账号：持有一个所有者用于抵押的资金的账号。
- 控制账号：控制所有者资金的账号。
- 周期：验证者集合（和每个验证者的活跃提名集合）是在一个周期后需要重新计算的，并在那里支付奖励。
- 惩罚：通过减少抵押着的资产来惩罚抵押者。

### 抵押模块的目标

抵押系统在Substrate的NPoS共识机制下，被设计用来使得下面几个目标成为可能：

- 抵押资产被一个冷钱包所控制。
- 在不影响实体作用角色的情况下，提取或存入部分资产。
- 以最小的开销可以在角色（提名者、验证者和空闲）之间切换。

### Validating 验证

一个验证者它的角色是验证区块和保证区块最终一致性，维护网络的诚实。验证者应该避免任何恶意的错误行为和离线。声明有兴趣成为验证者的绑定账号不会立即被选择为验证者，相反，他们被宣布为候选人，他们可能在下届选举中被选为验证者。选举结果由提名者及其投票决定。


### Nominator 提名者

一个提名者在维护网络时不承担任何直接的角色，而是对要选举的一组验证人进行投票。账号一旦声明了提名的利益，将在下一轮选举中生效。提名人stash账号中的资产表明其投票的重要性，验证者获得的奖励和惩罚都由验证者和提名者共享。这条规则鼓励提名者尽可能不要投票给行为不端/离线的验证者，因为如果提名者投错了票，他们也会失去资产。

### 奖励和惩罚

奖励和惩罚是抵押模块的核心，试图包含有效的行为，同时惩罚任何不当行为或缺乏可用性的行为。一旦有不当行为被报告，惩罚可以发生在任何时间点。一旦确定了惩罚，一个惩罚的值将从验证者的余额中扣除，同时也将从所有投票给该验证者的提名者的余额中扣除。与惩罚类似，奖励也在验证者和提名者之间共享。然而，奖励资金并不总是转移到stash账号。

### Chilling 冷却

最后，上面的任何角色都可以选择暂时退一步，冷静一下。这意味着，如果他们是提名者，他们将不再被视为选民，如果他们是验证者，他们将不再是下次选举的候选人。



## DAO

结合Substrate，继承其优秀的模块化设计，利用链上runtime升级来进行动态治理。

DAO的模块化设计允许runtime逻辑和利益相关者集合根据DAO的目的进行扩展和收缩。整个体系有利于进化并鼓励自我完善。

考虑到需要不断自我完善，DAO需要保持以下的设计标准：

- 可访问性：可用性决定于用户的多样性以及采用率。
- 可分叉：模块化和可扩展性使DAO可以根据利益相关者的利益进行适应和发展。
- 动态：明确的链上runtime升级允许该机制适应和发展。

> I’m increasingly of the belief that incentive design is the most important problem to work on. Humans are good at overcoming obstacles when they have reason to do so — the worst problems tend to be ones of cooperation, not technological or scientific understanding. ~ Devon Zuegel

DAO是一种协调机制，通过将争端解决方案委派给部署在抗审查性区块链上的代码来调整利益相关者的激励。

像长期存在的人类组织一样，DAO必须灵活而强大，尽管现代区块链可能提供足够的审查阻力来引导基本的组织，但现代区块链平台限制了已部署代码的演变，通常导致模棱两可的升级路径。

相反，Substrate鼓励变革，从一开始就致力于实现灵活，动态的治理系统。链上升级为实时系统提供了直观的升级途径。

通过将runtime编译为WASM并将其存储在链上，此体系结构可建立直接的升级路径。为了进行升级，在将新的runtime编译为WASM blob并更新链上存储的WASM之前，将调用runtime方法来更改存储。此过程消除了智能合约升级模式常见的歧义。

### Runtime架构

runtime将配置下面四个模块：

- membership
- voting
- fund
- committee

设计这些模块的目的是根据利益相关者的偏好配置一个资金协调DAO，以对合并资本进行细致的治理。runtime配置时有一些特定功能，但是每个模块本身都设计为通用且灵活的。这些功能包括：

- 灵活的会员资格要求
- 派驻资金管理委员会的代表团
- 捐赠和投资的资金管理
- 基于元治理的Futarchy
	- 元治理想要做到，利益相关者的预测保持稳定性，抑制激进的变化。在货币极端时期对货币政策实施被动元治理。
- 实施细节

### 会员资格治理

membership模块将包含用于向资金协调DAO添加/删除会员资格的逻辑。

DAO使用一个治理代币可以称为shares，用于内部信号的传递和加权决策。特别是内部治理下信号的传递需要Share s的抵押。

为了轻松去抵押这些shares，把他们在runtime中声明为ReservableCurrency类型，但还是把他们跟Currency类型分开，因为Currency可能是DOT或者其它某种通用的资产。

	pub trait Trait: system::Trait {
		// the governance token
		type Shares: ReservableCurrency<Self::AccountId> + Currency<Self::AccountId>;
		// the balances
		type Currency: LockableCurrency<Self::AccountId>;
		
	}
	
在DAO成员投票表决之前，会根据赞助要求过滤成员资格申请。更具体来说，每个提案都必须由DAO成员赞助，该成员必须锁定share s以支持该应用。申请人将Application向ApplicationPool提交后，该Application必须通过DAO成员赞助才能触发投票。

该Application必须详细说明申请人向DAO的总赞助额，以及DAO要求提供的share。如果批准并执行，则将捐赠总额从申请人转移到DAO，并将所请求的share从DAO铸造到申请人。也可以根据DAO资金池中相应比例的烧掉share。

> 用户界面必须要清楚传达出铸造新share将如何增加/减少每个share的价值。

	pub struct Application<AccountId, BalanceOf<T>, BlockNumber> {
		// application accountId
		application: AccountId,
		// the application's donation to the DAO
		donation: BalanceOf<T>,
		// membership shares requested from the DAO
		shares_requested: Shares,
		// start time,end time
		clock: (BlockNumber,BlockNumber),
	}
	
##### 如何管理提案吞吐量

通过抵押进行调整，DAO成员认为某个提案合理，可以锁定Share s作为提案的发起人，这样需要锁定抵押的方式来对提案进行过滤，从而保证提案不会像垃圾邮件一样被攻击。任何没有在最短时间内得到赞助的Application都会被从ApplicationPool中清理出去。

### 投票表决

提案获得足够的赞助后，一个Proposal将会被实例化，一个简单的实现将使用有状态的提案，将会在里面记录所有投票数据。

	pub struct Proposal<AccountId,BlockNumber> {
		// the proposal sponsor must be a member
		sponsor: AccountId,
		// the sponsor's share bond
		sponsor_bond: Shares,
		// the application
		application: AccountId,
		// donation to the DAO
		donation: BalanceOf,
		// mint request for membership shares
		shares_requested: Shares,
		// threshold for passage
		threshold: VoteThreshold,
		// shares in favor,shares against
		scoreboard: (Shares,Shares),
		// supporting voters (voted yes)
		ayes: Vec<(AccountId, Shares)>,
		// against voters (voted no)
		nays: Vec<(AccountId,Shares)>,	
	}


### 执行

如果Proposal获得足够的支持，则将其通过，将申请人的捐赠转移到DAO，然后将请求的share铸造并授予申请人。

	pub struct Member<AccountId> {
		pub account: AccountId,
		/// total shares owned
		pub share: Shares,
		/// total remaining shares
		pub remaining_shares: Shares,
		/// registered as a voter
		pub voter: bool,
	}
	
投票成员注册需要锁定一定的Share s。


### 基金治理

资金管理和会员资格分开，提供对潜在资金池的更细致的治理。这意味着基金提案与会员资格提案是分开的。

基金治理使用一个委员会来降低成员投票比例，也不完全放弃对基金治理过程的控制。该委员会由DAO成员使用基于Phragmem的委托通过Share加权投票选出。

#### 基金提案生命周期

应用程序通过双院制治理系统触发投票有两种方式：

- DAO众筹成员的提案抵押（相对昂贵，旨在要求众多成员的支持）。
- 委员会成员可以赞助数量有限的提案。

如果应用程序通过以上两个条件中的任何一个将获得通过支持。则投票将通过以下方式触发治理主体：

- DAO成员对基金提案进行投票。
- 委员会对基金提案进行投票。

当选民投票率低时，委员会的决定会对投票结果占据较重比重，但在高投票率的情况下不影响决定，这种权力平衡有利于在乐观的情况下进行责任下放，而不会在发生激烈争执时放弃任何控制权。

如果委员会被假设为“被俘虏”并试图对成员利益进行恶意投票，则成员可以以很高的投票率在全民投票中拒绝该提议。

#### 基金提案对象

目前定义的基金提案对象：

	pub enum FundProposal<'a, T: 'a> {
		Donation,
		Investment,
		phantom: std::marker::PhantomData<&'a T>,
	}
	


### Offchain Worker

当全节点接受并成功验证一个区块后执行，以异步方式执行，通过丰富的offchain worker API进行链下计算和交互。

代码位于runtime wasm之中，可以通过链上治理进行升级和部署。

#### Offchain Worker API

- sign
- submit_transaction
- random_seed
- timestamp
- sleep_until
- local_storage_get/local_storage_set
- http_request_start/http_response_read_body

Offchain Worker

- 与任何传统服务通过HTTP交互
- 链下计算
- 预言机
- 多方隐私计算
- 硬件钱包交互

srml-im-online

permil & perbil

Big Numbers



https://github.com/cennznet/tree/master/prml/fees

交易费的初步设计讨论：

https://github.com/paritytech/substrate/issues/1515

交易费模块可以适配所有交易费用处理逻辑，能让所有客户端轻松去查询交易费用的收取，现在客户端需要自己去计算交易手续费。

能够给验证者一些交易费奖励的实现。交易费在一个区块的所有交易中，最终都会被追踪到存储，on_finalize方法能够将这些交易费部分奖励给验证者。

这样一个交易费模块，不能够跟balances模块进行耦合，需要通用设计出来。这样一个交易费模块要能够跟其它模块，比如token模块和多资产模块进行协同工作。

能够支持动态费用的模块，（例如给合同的gas amount & gas price），就好比现在的静态费用模块（每个call固定的费用）。

https://github.com/paritytech/substrate/issues/1993

https://github.com/paritytech/substrate/issues/2910

Substrate不是一个固定方法的状态转移矿机，所以交易费不是非常清晰被定义了，在有些场景，交易费会被销毁掉，有些场景作为奖励给到验证者。



如何合理测量交易费大小：

https://github.com/paritytech/substrate/issues/2431




通过交易费控制区块大小：

https://github.com/paritytech/substrate/issues/2430


合约模块的接口复杂度分析：

https://github.com/paritytech/substrate/blob/master/srml/contracts/COMPLEXITY.md


治理：

https://wiki.polkadot.network/docs/en/learn-governance

原子交易和批量交易： 

https://github.com/paritytech/substrate/issues/1791

撤回机制：

https://github.com/paritytech/substrate/issues/2980

### assets模块

	decl_event!(
		pub enum Event<T>
			where <T as system::Trait>::AccountId,
			      <T as Trait>::Balance,
			      <T as Trait>::AssetId {
			/// Some assets were issued.
			Issued(AssetId, AccountId, Balance),
			/// Some assets were transferred.
			Transferred(AssetId, AccountId, AccountId, Balance),
			/// Some assets were destroyed.
			Destroyed(AssetId, AccountId, Balance),
		}
	);

- 发行资产 fn issue(origin,#[compact] total: T::Balance)
- 转账 fn transfer(origin,#[compact] id: T::AssetId,target: <T::Lookup as StaticLookup>::Source, #[compact] amount: T::Balance)
- 销毁资产 fn destroy(origin, #[compact] id: T::AssetId)
- 查询某个用户某个资产的余额 pub fn balance(id: T::AssetId, who: T::AccountId) -> T::Balance 
- 获取某个资产的发行总量 pub fn total_supply(id: T::AssetId) -> T::Balance


### babe模块

core中的inherents模块，翻译成“固有”，每一个固有在生产区块的时候被添加进去，每一个runtime决定在哪个固有上去固定其区块。所有在运行时所需要的用来被创建inherents的数据都被存储在`InherentData`，这个`InherentData`是节点构建出来的用来给到runtime。inherents是validator产生出来的，在runtime，模块需要实现`ProvideInherent`。



#### inherents模块

InherentData结构体

- pub fn new()
- pub fn put_data(I: codec::Encode>(&mut self,identifier:InherentIdentifier,inherent: &I,) -> Result<(),RuntimeString>
- pub fn replace_data(I: codec:Encode>(&mut self,identifier:InherentIdentifier,inherent: &I,)
- pub fn get_data<I: codec::Decode>(&self,identifier: &InherentIdentifier,) -> Result<Option<I>,RuntimeString)

CheckInherentsResult结构体

- fn default() -> Self
- pub fn new() -> Self
- pub fn put_error<E: codec::Encode + IsFatalError>(&mut self,identifier: InherentIdentifier,error: &E) -> Result<(),RuntimeString>
- pub fn get_error<E: codec::Decode>(&self,identifier:&InherentIdentifier) -> Result<Option<E>,RuntimeString>
- pub fn into_errors(self) -> IntoIter<InherentIdentifier,Vec<u8>>
- pub fn ok(&self> -> bool
- pub fn fatal_error(&self) -> bool

InherentDataProviders结构体

- pub fn new() -> Self
- pub fn register_provider<P: ProvideInherentData + Send + Sync + 'static>(&self,provider: P) -> Result<(),RuntimeString>
- pub fn create_inherent_data(&self) -> Result<InherentData,RuntimeString>
- pub error_to_string(&self,identifier: &InherentIdentifier,error: &[u8]) -> String





### elections模块

Election模块，用于抵押-权重的集体会员选举。理事会成员的选举。
	
	decl_event!(
		pub enum Event<T> where <T as system::Trait>::AccountId {
			/// reaped voter, reaper
			VoterReaped(AccountId, AccountId),
			/// slashed reaper
			BadReaperSlashed(AccountId),
			/// A tally (for approval votes of seat(s)) has started.
			TallyStarted(u32),
			/// A tally (for approval votes of seat(s)) has ended (with one or more new members).
			TallyFinalized(Vec<AccountId>, Vec<AccountId>),
		}
	);
	
- fn set_approvals(origin, votes: Vec<bool>, #[compact] index: VoteIndex, hint: SetIndex) -> Result
	- 批准候选人，在这段时间内候选人注册了，批准将是有效的。将无限期锁定调用者的所有余额。只有[`retract_voter`]和[`reap_inactive_voter`]能解锁余额。
- fn proxy_set_approvals(origin,votes: Vec<bool>, #[compact] index: VoteIndex,hint: SetIndex) -> Result 
	- 代理批准候选人。
- fn reap_inactive_voter(origin,#[compact] reporter_index: u32,who: <T::Lookup as StaticLookup>::Source,#[compact] who_index: u32,#[compact] assumed_vote_index: VoteIndex)
	- 移除一个投票者，所有被批准的候选人索引要么未注册，要么在投票人给出他们最后的批准集合后注册成为一个候选人。
- fn retract_voter(origin,#[compact] index:u32)
	- 移除一个投票者，所有投票者都能够取消并会返回其锁住的存款。
- fn submit_candidacy(origin,#[compact] slot:u32)
	- 某人申请候选资格。
- fn present_winer(origin,candidate: <T::Lookup as StaticLookup>::Source,#[compact] total: BalanceOf<T>,#[compact] index: VoteIndex) -> Result
	- 声称自己是top Self::carry_count()+current_vote().1 候选人中的一个。
- fn set_desired_seats(origin, #[compact] count:u32)
	- 设置所需要的成员数量。
- fn remove_member(origin,who: <T::Lookup as StaticLookup>::Source)
	- 将某个特定的成员移除。
- fn set_presentation_duration(origin,#[compact] count: T::BlockNumber)
	- 设置presentation时长。
- fn set_term_duration(origin,#[compact] count: T::BlockNumber)
	- 设置presentation时长。

-  pub fn presentation_active() -> bool
	- 查询现在是否在一个presentation阶段

-  pub fn is_a_candidate(who: &T::AccountId) -> bool
   - 查询某个账号在此时是否是一个候选人

-  pub fn will_still_be_member_at(who: &T::AccountId, n: T::BlockNumber) -> bool
	- 在区块高度n的时候，某个账号是否还有席位

-  pub fn next_vote_from(n: T::BlockNumber) -> T::BlockNumber
	- 在不低于区块高度n的时候，返回具体区块高度能让投票开始

-  pub fn next_tally() -> Option<T::BlockNumber>
	- 返回区块高度，能让为下一次选举的计数开始

-  pub fn bool_to_flag(x: Vec<boll>) -> Vec<ApprovalFlag>
	- 将布尔值的vec转成flags的vec

-  pub fn flag_to_bool(chunk: Vec<ApprovalFlag>) -> Vec<bool>
	- 将flags的vec转化成布尔值的vec


### democracy模块

Democracy系统：处理抵押投票用户的行政管理。

	decl_event!(
		pub enum Event<T> where
			Balance = BalanceOf<T>,
			<T as system::Trait>::AccountId,
			<T as system::Trait>::Hash,
			<T as system::Trait>::BlockNumber,
		{
			Proposed(PropIndex, Balance),
			Tabled(PropIndex, Balance, Vec<AccountId>),
			ExternalTabled,
			Started(ReferendumIndex, VoteThreshold),
			Passed(ReferendumIndex),
			NotPassed(ReferendumIndex),
			Cancelled(ReferendumIndex),
			Executed(ReferendumIndex, bool),
			Delegated(AccountId, AccountId),
			Undelegated(AccountId),
			Vetoed(AccountId, Hash, BlockNumber),
		}
	);
	
- fn propose(origin,proposal: Box<T::Proposal>,#[compact] value:BalanceOf<T>
	- 发起一个提案。
- fn second(origin,#[compact] proposal: ProposalIndex) 
	- 对一个提案进行附议。
- fn vote(origin,#[compact] ref_index: ReferendumIndex,vote:Vote) -> Result
	- 对一个公民投票进行投票。
- fn proxy_vote(origin,#[compact] ref_index: ReferendumIndex,vote:Vote) -> Result
	- 对一个公民投票进行投票，如果是赞成票，就支持议案，否则就维持现状。
- fn emergency_cancel(origin,ref_index: ReferendumIndex)
	- 对一个公民投票进行紧急取消。
- fn external_propose(origin,proposal: Box<T::Proposal>)
	- 一旦安排的外部公投是合法的，那么公投将被tabled。
- fn external_propose_majority(origin,proposal: Box<T::Proposal>)
	- 一个大多数通过的公投被tabled。
- fn external_propose_default(orgin,proposal:Box<T::Proposal>)
	- 一个negative-turnout-bias公投被tabled。
- fn fast_track(origin,proposal_hash: T::Hash,voting_period: T::BlockNumber, delay: T::BloclNumber)
	- 将目前外部提议的多数通过的公投立即提上日程。如果目前没有外部提议的公投，或者有一个但不是多数通过的公投，那么公投就失败了。
- fn veto_external(origin,proposal_hash: T::Hash)
	- 将外部提案hash进行否决和加入黑名单。
- fn cancel_referendum(origin,#[compact] ref_index: ReferendumIndex)
	- 取消一个公投。
- fn cancel_queued(origin,#[compact] when: T::BlockNumber,#[compact] which: u32,#[compact] what: ReferendumIndx)
	- 取消等待通过的提案。
- set_proxy(origin,proxy: T::AccountId)
	- 设置一个代理，被隐藏的账号所调用。
- fn resign_proxy(origin)
	- 清除代理。
- fn remove_proxy(origin,proxy: T::AccountId)
	- 清除代理，被隐藏账号所调用。
- fn undelegate(origin)
	- 不再代理投票。

- pub fn new(end:BlockNumber,proposal:Proposal,threshold:VoteThreshold,delay:BlockNumber) -> Self {ReferendumInfo{end,proposal,threshold,delay}}
	- 创建一个democracy实例

-  pub fn delegate(origin,to:T::AccountId,conviction:Conviction)
	- 代理投票

-  pub fn locked_for(proposal:Proposal) -> Option<BalanceOf<T>>
	- 获取在proposal中锁定的总额

-  pub fn is_active_referendum(ref_index: ReferendumIndex) -> bool
	- 查询是否是某个正在进行中的公民投票

-  pub fn active_referenda() -> Vec<(ReferendumIndex,ReferendumInfo<T::BlockNumber,T::Proposal>)>
	- 获取所有正在进行的公民投票

-  pub fn maturing_referenda_at(n: T::BlockNumber) -> Vec<(ReferendumIndex,ReferendumInfo<T::BlockNumber,T::Proposal>)>
	- 获取所有在区块n之前准备好了的公民投票

-  pub fn tally(ref_index: ReferendumIndex) -> (BalanceOf<T>,BalanceOf<T>,BalanceOf<T>)
	- 从现在的提案中获取所有的投票支持数

-  pub fn force_proxy(stash: T::AccountId,proxy: T::AccountId)
	- 强制代理

-  pub fn internal_start_referendum(proposal: T::Proposal,threshold: VoteThreshold, delay: T::BlockNumber) -> result::Result<ReferendumIndex, &'static str>
	- 开启一个公民投票

-  pub fn internal_cancel_referendum(ref_index: ReferendumIndex)
	- 移除一个公民投票


### council模块

Council系统，处理投票和维护议会成员。被重构成了collective和elections几个小的模块

- fn on_members_changed(new: &[AccountId], old: &[AccountId)
	- 新理事会成员进来替换掉老的成员。


### staking模块

- 一个验证者控制的账号下能被惩罚掉的总余额 pub fn slashable_balance_of(stash: &T::AccountId) -> BalanceOf<T>

- 用验证者的stash账号ID，将奖励给到这些账号 pub fn reward_by_ids(validators_points: impl IntoIterator<Item = (T::AccountId, u32)>)

- 用验证者的索引，将奖励给到相应的验证者 pub fn reward_by_indices(validators_points: impl IntoIterator<Item = (u32,u32)>)

	decl_event!(
		pub enum Event<T> where BalanceOf<T>, <T as system::Trait>::AccountId {
			/// All validators have been rewarded by the given balance
			Reward(Balance),
			/// One validator (and its nominators) has been slashed by the given amount
			Slash(AccountId,Balance),
			/// an old slashing report from a prior era was discarded because it could not be processed
			OldSlashingReportDiscarded(SessionIndex),
		}
	);
	


### memebership模块

	decl_event!(
		pub enum Event<T, I=DefaultInstance> where
			<T as system::Trait>::AccountId,
			<T as Trait<I>>::Event,
		{
			/// The given member was added; see the transaction for who.
			MemberAdded,
			/// The given member was removed; see the transaction for who.
			MemberRemoved,
			/// Two members were swapped; see the transaction for who.
			MembersSwapped,
			/// The membership was reset; see the transaction for who the new set is.
			MembersReset,
			/// Phantom member, never used.
			Dummy(rstd::marker::PhantomData<(AccountId, Event)>),
		}
	);
	
- add_member(origin,who: T::AccountId)
	- 新增成员，需要root权限
- remove_member(origin,who: T::AccountId)
	- 移除成员，需要root权限
- swap_member(origin,remove: T::AccountId,add: T::AccountId)
	- 剔除某个成员，然后新增，需要root权限
- reset_members(origin,members: Vec<T::AccountId>)
	- 将成员更改到一个新的集合，需要root权限


### indices模块
	
	decl_event!(
		pub enum Event<T> where
			<T as system::Trait>::AccountId,
			<T as Trait>::AccountIndex
		{
			/// A new account index was assigned.
			/// This event is not triggered when an existing index is reassigned to another `AccountId`.
			NewAccountIndex(AccountId, AccountIndex),
		}
	);

- fn lookup_index(index: T::AccountIndex) -> Option<T::AccountId>
	- 根据AccountIndex来查找账号。
- fn can_reclaim(try_index: T::AccountIndex) -> bool
	- AccountIndex是否已经准备回收再利用
- fn lookup_address(a: address::Address<T::AccountId,T::AccountIndex> -> Option<T::AccountId>
	- 寻找账号
- fn on_new_account(who: &T::AccountId)

### offences模块

	decl_event!(
		pub enum Event {
			/// There is an offence reported of the given `kind` happened at the `session_index` and
			/// (kind-specific) time slot. This event is not deposited for duplicate slashes.
			Offence(Kind, OpaqueTimeSlot),
		}
	);

- report_offence(reporters: Vec<T::AccountId>,offence: O)
	- 报告犯罪
- fn report_id<O: Offence<T::IdentificationTuple>>(time_slot: &O::TimeSlot,offender: &T::IdentificationTuple,) -> ReportIdOf<T>
	- 从给定的报告特性计算出ID。这个报告id建立在犯罪种类，时间槽和罪犯id基础上。
- fn triage_offence_report<O: Offence<T::IdentificationTuple>>(reporters: Vec<T::AccountId>,time_slot: &O::TimeSlot,offenders: Vec<T::IdentificationTuple>,) -> Option<TriageOutcome<T>>
	- 对offence报告进行分类，并返回一组罪犯，他们涉及到同时发生的犯罪列表相关的独特的报告。

### balances模块

	decl_event!(
		pub enum Event<T, I: Instance = DefaultInstance> where
			<T as system::Trait>::AccountId,
			<T as Trait<I>>::Balance
		{
			/// A new account was created.
			NewAccount(AccountId, Balance),
			/// An account was reaped.
			ReapedAccount(AccountId),
			/// Transfer succeeded (from, to, value, fees).
			Transfer(AccountId, AccountId, Balance, Balance),
		}
	);
	
- pub fn transfer(origin,dest: <T::Lookup as StaticLookup>::Source,#[compact] value: T::Balance)
	- 转账，当然是要能有流动性的自由余额。
- fn set_balance(origin,who: <T::Lookup as StaticLookup>::Source,#[compact] new_free: T::Balance,#[compact] new_reserved: T::Balance)
	- root权限，设置余额。
- pub fn force_transfer(origin,source: <T::Lookup as StaticLookup>::Source,dest: <T::Lookup as StaticLookup>::Source,#[compact] value: T::Balance)
	- 转账，但转出者是root权限，转出账号需要指定的。
- fn vesting_balance(who: &T::AccountId) -> T::Balance
	- 从某个查找中查询出正在vested的总额。这部分资金是不能从这个账号中转出的。
- fn set_reserved_balance(who: &T::AccountId,balance: T::Balance) -> UpdateBalanceOutcome
	- 给一个账号设置保留余额。
- fn set_free_balance(who: &T::AccountId,balance: T::Balance) -> UpdateBalanceOutcome
	- 给一个账号设置自由余额。
- fn new_account(who: &T::AccountId,balance: T::Balance) 
	- 注册一个新的账号，还有余额。
- fn reap_account(who: &T::AccountId)
	- 注销掉一个账号。
- fn on_free_too_low(who: &T::AccountId)
	- 账号的自由余额太低，比最少的存在存款还要低，如果其保留余额已经是死的了，那这个账号的自由余额和账号都会被杀死。
- fn on_reserved_too_low(who: &T::AccountId)
	- 账号的保留余额比最小的存在存款还低，如果自由余额已经被杀死了，那么其保留余额和账号将被杀死。
- fn total_balance(who: &T::AccountId) -> Self::Balance
- fn can_slash(who :&T::AccountId,value: Self::Balance) -> bool
- fn total_issuance()
- fn minimum_balance()
- fn free_balance(who: &T::AccountId)
- fn burn(mut amount: Self::Balance) -> Self::PositiveTmbalance
- fn issue(mut amout: Self::Balance) -> Self::NegativeImbalance
- fn ensure_can_withdraw(who: &T::AccountId,_amount: T::Balance,reasom: WithdrawReason,new_balance: T::Balance,) -> Result
- fn transfer(transactor: &T::AccountId,dest: &T::AccountId, value: Self::Balance) -> Result
- fn withdraw(who: &T::AccountId,value: Self::Balance,reason: WithdrawReason,liveness: ExistenceRequirement,) -> result::Result<Self::NegativeImbalance, &'static str>
- fn slash(who &T::AccountId,value: Self::Balance)
- fn deposit_into_existing(who: &T::AccountId,value: Self::Balance) -> result::Result<Self::PositiveImbalance, &'static str>
- fn deposit_creating(who: &T::AccountId,value: Self::Balance) -> Self::PositiveImbalance
- fn make_free_balance_be(who: &T::AccountId,balance: Self::Balance) -> (SignedImbalance<Self::Balance,Self::PositiveImbalance>,UpdateBlanceOutcome)
- fn can_reserve(who: &T::AccountId,value: Self::Balance) -> bool
- fn reserved_balance(who: &T::AccountId) -> Self::Balance
- fn reserve(who: &T::AccountId,value: Self::Balance) -> result::Result<(),&'static str>
- fn unreserve(who: &T::AccountId,value: Self::Balance) -> Self::Balance
- fn slash_reserved(who: &T::AccountId,value: Self::Balance) -> (Self::NegativeImbalance,Self::Balance)
- repatriate_reserved(slashed: &T::AccountId,beneficiary: &T::AccountId,value: Self::Balance,) -> result::Result<Self::Balance,&'static str>
	- 将slash掉的金额遣返给某个账号。
- fn set_lock(id: LockIdentifier,who: &T::AccountId,amount: T::Balance,until: T::BlockNumber,reason: WithdrawReasons,) 
- fn extend_lock(id: LockIdentifier,who: &T::AccountId,amount: T::Balance,until: T::BlockNumber,reason: WithdrawReasons,)
- remove_lock(id:LockIdentifier,who:&T::AccountId,)
- pub fn from(fee: T::Balance) -> Self
- fn compute_fee(len: usize,info: DispatchInfo, tip: T::Balance) -> T::Balance
	- 为一笔单独的转账计算最终的手续费

### collective模块

Collective系统：从一个或两个专门的origins，调度调用来表达一组帐户ID的成员的集体感受。membership成员能够通过两种方来提供，一种是通过root权限可调度的方法set_members,另外一种是继承实现ChangeMembers.

	decl_event!(
		pub enum Event<T, I=DefaultInstance> where
			<T as system::Trait>::Hash,
			<T as system::Trait>::AccountId,
		{
			/// A motion (given hash) has been proposed (by given account) with a threshold (given
			/// `MemberCount`).
			Proposed(AccountId, ProposalIndex, Hash, MemberCount),
			/// A motion (given hash) has been voted on by given account, leaving
			/// a tally (yes votes and no votes given respectively as `MemberCount`).
			Voted(AccountId, Hash, bool, MemberCount, MemberCount),
			/// A motion was approved by the required threshold.
			Approved(Hash),
			/// A motion was not approved by the required threshold.
			Disapproved(Hash),
			/// A motion was executed; `bool` is true if returned without error.
			Executed(Hash, bool),
			/// A single member did some action; `bool` is true if returned without error.
			MemberExecuted(Hash, bool),
		}
	);
	
- fn set_members(origin,new_members: Vec<T::AccountId>)
- fn execute(origin,proposal: Box<<T as Trait<T>>::Proposal>)
- fn propose(origin,#[compact] threshold:MemberCount,proposal: Box<<T as Trait<T>>::Proposal>)
- fn vote(origin,proposal: T::Hash,#[compact] index: ProposalIndex,approve:bool)


### im-online模块

	decl_event!(
		pub enum Event<T> where
			<T as Trait>::AuthorityId,
		{
			/// A new heartbeat was received from `AuthorityId`
			HeartbeatReceived(AuthorityId),
		}
	);
	

- fn heartbeat(origin,heartbeat: Heartbeat<T::BlockNumber>,signature: <T::AuthorityId as RuntimeAppPublic>::Signature)
	- 验证节点给其它节点发送心跳包
- fn offchain_worker(now: T::BlockNumber)
	- 在每个区块后都会运行这个方法，如果是一个潜在的验证节点就只需要发送消息。
- pub fn is_online_in_current_session(authority_index: AuthIndex) -> bool
	- 如果收到心跳就返回true
- fn offchain(now: T::BlockNumber)

### session模块

Session模块能够让验证者去管理它们的session key，提供一个方法去改变session长度，和处理session轮流。

- Session：会话是一段有一组常量验证器的时间。验证器只能在会话更改时加入或退出验证器集。它是用块数来度量的。会话结束的块由“ShouldSessionEnd” Trait决定。当会话结束时，可以通过' OnSessionEnding '实现选择一个新的验证器集。

- Session key：一个会话密钥实际上是几个保存在一起的密钥，它们提供了网络权威机构/验证器在执行其职责时所需的各种签名功能。

- Validator ID：每个账号有一个相对应的validator ID，对于一些简单的抵押系统，这能够跟account ID发挥一样的作用，对于使用隐藏/控制器模型的抵押系统中，validator ID将是控制器的隐藏账号ID。

- Session key configuration process：使用' set_key '设置会话密钥以供下一个会话使用。它存储在“NextKeyFor”中，这是调用者的“ValidatorId”和提供的会话密钥之间的mapping。' set_key '允许用户在被选择为验证器之前设置他们的会话密钥。它是一个公共调用，因为它使用' ensure_signed '，它检查源帐户是否是签名帐户。因此，存储在' NextKeyFor '中的原始帐户ID不一定与块作者或验证器关联。帐户的会话密钥在帐户余额为零时被删除。

- Validator set session key configuration process：在每个会话中，我们迭代当前的验证器帐户id集，以检查是否在前一个会话中使用' set_key '为它创建了会话密钥。如果当时我们称之为“set_authority”并将其传递给一组会话密钥(每一个帐户ID)作为新的会话密钥验证器集。最后,如果当前的会话密钥的权威不匹配任何会话密钥存储在其验证器指数AuthorityStorageVec映射,然后更新映射会话密钥和更新保存的原始列表当局在必要时(见https://github.com/paritytech/substrate/issues/1290)。注意:权限存储在Consensus模块中。它们由来自会话模块的验证器帐户ID索引表示，并使用会话密钥分配会话长度。

- Session length change process：在下一个会话开始时，我们分配一个会话索引并记录会话启动时的时间戳。如果‘NextSessionLength’记录在前一个会话中，我们将它记录为新会话长度。另外，如果新会话的长度与下一个会话的长度不同，那么我们将记录“LastLengthChange”。

- Session rotation configration：配置为“normal”(可奖励的session，奖励将可以被应用)或“exceptional”(可惩罚)会话轮换。

- Session rotation process：使用' on_finalize '方法在当前会话的最后一个区块结束时更改会话。它可以由一个origin调用，也可以从每个区块末尾的另一个运行时模块内部调用。

Session模块在Substrate中被设计用来做如下的事情：

- 为验证者集合参与下一轮session设置session keys。
- 设置session的长度。
- 在normal或者exceptional session轮换中配置和切换。

- `set_key`:在下轮session中为验证者设置session key。
- `set_length`:在下一轮session改变的时候设置一个新的被应用的session长度。
- `force_new_session`:强制开启一个新的session，其应该被认为是normal或者exception轮换。
- `on_finalize`:当一个区块要finalized的时候被调用，如果这个区块是这个session的最后一个区块，那么要转换session。

- `validator_count`: 获取现在的验证者数量。
- `last_length_change`: 当session长度上一次改变时，获取当时的区块高度。
- `apply_force_new_session`:强制开启一个新的session，能够被其它runtime的modules调用。
- `set_validators`: 设置现在的验证者集合，只能够被抵押模块调用。
- `check_rotate_session`: 转换session和应用奖励，如果有必要的话。当抵押模块更新认证出块者到新的验证者人集合后，此方法能够被调用。
- `rotate_session`:更改到下一个session，注册新的认证出块人集合，更新session keys，如果可被应用的session长度被改变了，要颁布这种改变。
- `ideal_session_duration`:获取一个理想session的时间。
- `blocks_remaining`:现在这个session还剩下多少区块高度就要进入下一个session了。


- trait SholdEndSession 
	- fn should_end_session(now: BlockNumber) -> bool;
	
- trait OnSessionEnding<ValidatorId>
	- fn on_session_ending(ending_index:SessionIdex,will_apply_at:SessionIndex) -> Option<Vec<ValidatorId>>
	- fn on_genesis_session<Ks: OpaqueKeys>(validators: &[(ValidatorId,Ks)])
	- fn on_new_session<Ks: OpaqueKeys>(changed:bool,validators:&[(ValidatorId,Ks)],queued_validators:&[(ValidatorId,Ks)])
	- fn on_before_session_ending(){}
	- fn on_disabled(validator_index:usize)

- trait OneSessionHandler<validatorId>
	- fn on_genesis_session
	- fn on_new_session
	- fn on_before_session_ending
	- fn on_disabled

- trait SelectInitialValidators<ValidatorId>
	- fn select_inital_validators() -> Option<vec<ValidatorId>>


	decl_event!(
		pub enum Event {
			/// New session has happened. Note that the argument is the session index, not the block
			/// number as the type might suggest.
			NewSession(SessionIndex),
		}
	);

	decl_module! {
		fn set_keys(origin,keys:T::Keys,proof: vec<u8>) ->Result
		fn on_initialize(n: T::BlockNumber)
	}
	

### treasury模块

Treasury模块提供一个资金池，能够由抵押者们来管理，在这个国库系统中，能够发起发费提案从这个资金池中。

抵押者们可以去提案，批准或者拒绝开支。当然区块链需要提供一个方法（手续费或者通货膨胀）去收集这个资金到国库中。

##### 术语

- Proposal：一个从资金池中分配给受益者的提案。
- Beneficiary：一个账号能够接收到资金，当然这个是要从提案中被通过才能够的。
- Deposit：当发起一个提案，提案中要用到的资金将会被锁住，这个deposit将会被返回或者惩罚掉，如果这个提案通过或者被分别拒绝掉。
- Pot：国库模块计算出来的未花费的资金。

国库模块提供和实现了下面的特性trait：

- `Ondilution`：当新资金被铸造以奖励其他现有资金的部署时，相应数量的代币被铸造到国库中，这样被奖励的代币就不会占总供应量的更高比例。例如，在默认的的substrate节点中，当验证者因为抵押而获得新Token时，它们并不持有更高比例的token。相反，将token添加到库中以保持抵押token的部分固定不变。

	decl_module! {
		fn propose_spend(origin,#[compact] value: BalanceOf<T>, beneficiary:<T::Lookup as StaticLookup>::Source)
		fn reject_proposal(origin,#[compact] proposal_id: ProposalIndex)
		fn approve_proposal(origin,#[compact] proposal_id: ProposalIndex)
		fn on_finalize(n: T::BlockNumber){
			fn (n % T::SpendPeriod::get()).is_zero() {
				Self::spend_funds();
			}
		}
	}



### scored-pool模块

该模块维护一个得分的成员池。池中的每个实体都可以被赋予一个“分数”。从这个池构建一个集合“成员”。此集合包含' MemberCount '得分最高的实体。未得分的实体从来不是“成员”的一部分。

如果一个实体想要成为池的一部分，就需要存款。当该实体提款或被具有适当权限的实体取出时，该存款将被退还。

池中得分最高的成员，不管是否发生了更改，都会调用' T::MembershipChanged::set_members_sort '。在第一次加载时，' T::MembershipInitialized::initialize_members '被初始的' Members '集合调用。

任何时候都可以退出候选资格/退出会员资格。如果一个实体当前是成员，这将导致从“池”和“成员”中删除;该实体将立即被池中得分第二高的候选人(如果可用)替换。

#### Public Functions

- `submit_candidacy`:提交候选人去竞选会员，需要存款。
- `withdraw_candidacy`:存储取回。
- `score`:给实体贡献一个量化的分数。
- `kick`:将一个实体从候选池和会员中移除，退还存款。
- `change_member_count`:改变候选人，进入会员成员中。

核心代码：

	decl_event!(
		pub enum Event<T, I=DefaultInstance> where
			<T as system::Trait>::AccountId,
		{
			/// The given member was removed. See the transaction for who.
			MemberRemoved,
			/// An entity has issued a candidacy. See the transaction for who.
			CandidateAdded,
			/// An entity withdrew candidacy. See the transaction for who.
			CandidateWithdrew,
			/// The candidacy was forcefully removed for an entity.
			/// See the transaction for who.
			CandidateKicked,
			/// A score was attributed to the candidate.
			/// See the transaction for who.
			CandidateScored,
			/// Phantom member, never used.
			Dummy(rstd::marker::PhantomData<(AccountId, I)>),
		}
	);

	decl_module! {
		fn on_initialize(n: T::BlockNumber){
			if n % T::Period::get() == Zero::zero() {
				let pool = <Pool<T,I>::get();
				<Module<T,I>>::refresh_members(pool,ChangeReceiver::MembershipChanged);				
			}
		}
		pub fn submit_candidacy(origin)
		pub fn withdraw_candidacy(origin,index:u32)
		pub fn kick(origin,dest:<T::Lookup as StaticLookup>::Source,index:u32)
		pub fn score(origin,dest: <T::Lookup as StaticLookup>::Source,index:u32,score: T::Score)
		pub fn change_member_count(origin,count:u32)
	}
	
- fn refresh_members(pool:Pool<T,I>,notify:ChangeReceiver)
- fn remove_member(mut pool:Pool<T,I>,remove: T::AccountId,index: u32) -> Result<(),&'static str>
- fn ensure_index(pool: &PoolT<T,I>, who: &T::AccountId, index: u32) -> Result<(),&'static str>


### generic-asset模块

Generic Asset模块提供了处理多账号和资产余额的方法。

提供了如下的方法：

- 创建一种新的资产。
- 设置一种资产的权限。
- 获取和设置自由余额。
- 取回总的，保留的和未保留的余额。
- 遣返一个保留的余额给某个受益账号。
- 不同账号中的余额进行转账（不包括保留的余额）。
- 惩罚一个账号的余额。
- 管理一个总的发行。
- 设置和管理资金锁。

##### 术语

- Staking Asset:资产抵押，例如在网络中为了参与验证节点。
- Spending Asset:需要支付的资产，比如支付交易费和gas费。
- Permissions:一种资产的一个规则集合，定义了资产可以的操作，哪个账号能够被允许去处理这些操作。
- Total Issuance:总发行量。
- Free Balance:自由余额。
- Reserved Balance:保留余额。
- Imbalance:不平衡。
- Lock:资产锁。

核心代码：

	/// Asset creation options
	pub struct AssetOptions<Balance: HasCompact, AccountId> {
		pub initial_issuance: Balance,
		pub permissions: PermissionLatest<AccountId>,
	}
	
	/// Owner of an asset
	pub enum Owner<AccountId> {
		None,
		Address(AccountId),
	}
	
	/// Asset permissions
	pub struct PermissionV1<AccountId> {
		pub update: Owner<AccountId>,
		pub mint: Owner<AccountId>,
		pub burn: Owner<AccountId>,
	}
	
	/// Asset permission types
	pub enum PermissionType {
		Burn,
		Mint,
		Update,
	}
	
	decl_module! {
		fn create(origin,options: AssetOptions<T::Balance,T::AccountId>) -> Result
		pub fn transfer(origin,#[compact] asset_id: T::AssetId, to: T::AccountId, #[compact] amount: T::Balance)
		fn update_permission(origin,#[compact] asset_id: T::AssetId,new_permission: PermissionLatest<T::AccountId>) -> Result
		fn mint(origin,#[compact] asset_id: T::AssetId, to: T::AccountId,amount: T::Balance) -> Result
		fn burn(origin,#[compact] asset_id: T::AssetId, to: T::AccountId, amount: T::Balance) -> Result
		fn create_erserved(origin,asset_id: T::AssetId, options: AssetOptions<T::Balance, T::AccountId>) -> Result
	}
	
	decl_event!(
		pub enum Event<T> where
			<T as system::Trait>::AccountId,
			<T as Trait>::Balance,
			<T as Trait>::AssetId,
			AssetOptions = AssetOptions<<T as Trait>::Balance, <T as system::Trait>::AccountId>
		{
			/// Asset created (asset_id, creator, asset_options).
			Created(AssetId, AccountId, AssetOptions),
			/// Asset transfer succeeded (asset_id, from, to, amount).
			Transferred(AssetId, AccountId, AccountId, Balance),
			/// Asset permission updated (asset_id, new_permissions).
			PermissionUpdated(AssetId, PermissionLatest<AccountId>),
			/// New asset minted (asset_id, account, amount).
			Minted(AssetId, AccountId, Balance),
			/// Asset burned (asset_id, account, amount).
			Burned(AssetId, AccountId, Balance),
		}
	);

	
- pub fn total_balance(asset_id: &T::AssetId,who: &T::AccountId) -> T::Balance
- pub fn free_balance(asset_id: &T::AssetId, who: &T::AccountId) -> T::Balance
- pub fn reserved_balance(asset_id: &T::AssetId, who: &T::AccountId) -> T::Balance
- pub fn create_asset(asset_id: Option<T::AssetId>,from_account: Option<T::AccountId>,options: AssetOptions<T::Balance,T::AccountId>) -> Result
- pub fn reserve(asset_id: &T::AssetId, who: &T::AccountId, amount: T::Balance) -> Result
- pub fn unreserve(asset_id: &T::AssetId, who: &T::AccountId, amount: T::Balance) -> T::Balance
- fn slash(asset_id: &T::AssetId, who: &T::AccountId, amount: T::Balance) -> Option<T::Balance>
- fn slash_reserved(asset_id: &T::AssetId, who: &T::AccountId, amount: T::Balance) -> Option<T::Balance>
- fn repatriate_reserved(asset_id: &T::AssetId, who: &T::AccountId, beneficiary: &T::AccountId, amount: T::Balance) -> T::Balance
- fn set_reserved_balance(asset_id: &T::AssetId, who: &T::AccountId, balance: T::Balance)
- fn set_free_balance(asset_id: &T::AssetId, who: &T::AccountId, balance: T::Balance)
- fn set_lock(id: LockIdentifier,who: &T::AccountId, amount: T::Balance,until: T::BlockNumber, reason: WithdrawReasons)
- fn extend_lock(id: LockIdentifier,who: &T::AccountId, amount: T::Balance,until: T::BlockNumber, reason: WithdrawReasons)
- fn remove_lock(id: LockIdentifier, who: &T::AccountId)


### authorship模块

Authorship tracking 追踪现在的出块者和最近的叔块。

	
	/// Find the author of a block.
	type FindAuthor: FindAuthor<Self::AccountId>;
	/// The number of blocks back we should accept uncles.
	type UncleGenerations: Get<Self::BlockNumber>;
	/// A filter for uncles within a block.This is for implementing further constraints on what uncles can be included,other than their ancestry.
	type FilterUncle: FilterUncle<Self::Header,Self::AccountId>;
	type EventHandler: EventHandler<Self::AccountId,Self::BlockNumber>;
	
	
	decl_module! {
		fn on_initialize(now: T::BlockNumber)
		fn on_finalize()
		fn set_uncles(origin,new_uncles: Vec<T::Header>) -> DispatchResult
	}
	
	pub fn author() -> T::AccountId
	fn verify_and_import_uncles(new_uncles: Vec<T::Header>) -> DispatchResult
	fn verify_uncle<'a, I: IntoIterator<Item=&'a T::Hash>>
	fn prune_old_uncles(minimun_height: T::BlockNumber)
	

ProvideInherent 

	fn create_inherent(data: &InherentData) -> Option<Self::Call>
	

### authority-discovery 模块

Authority discovery 模块，这个模块使用了 `core/authority-discovery`去检索现在的出块者集合，学习它自己的出块权限id，以及对来自其他出块者的消息进行签名和验证。

- fn authority_id() -> Option<T::AuthorityId>
- pub fn authorities() -> Vec<T::AuthorityId>
- pub fn sign(payload: &Vec<u8>) -> Option<(<<T as Trait>::AuthorityId as RuntimeAppPublic>::Signature,T::AuthorityId)>
- pub fn verify(payload: &Vec<u8>,signature: <<T as Trait>::AuthorityId as RunmtimeAppPublic>::Signature,autority_id: T::AuthorityId) -> bool


### contracts模块

Contracts模块提供了方法给runtime去部署和执行WebAssembly智能合约。

这个模块扩展账号是基于`Currency` trait，拥有智能合约的功能。其它模块只要也是基于`Currency`的账号也是可以使用此智能合约的。这些智能合约账号有能力去举例说明智能合约和能够对其它合约和非合约账号发起调用。


	pub trait ContractAddressFor<CodeHash,AccountId> {
		fn contract_address_for(code_hash: &CodeHash, data: &[u8], origin: &AccountId) -> AccountId;
	}
	
	pub trait ComputeDispatchFee<Call,Balance> {
		fn compute_dispatch_fee(call: &Call) -> Balance;
	}
	


### consensus中select_chain

SelectChain trait定义了策略，在对一个最佳区块的定义是比较模糊的区块链中，SelectChain定义了选择forks的策略。

有三个方法：

	fn leaves(&self) -> Result<Vec<<Block as BlockT>::Hash>, Error>;
	fn best_chain(&self) -> Result<<Block as BlockT>::Header, Error>;
	fn finality_target(&self,target_hash: <Block as BlockT>::Hash, _maybe_max_number: Option<NumberFor<Block>>) -> Result<Option<<Block as BlockT>::Hash>, Error>
	
- leaves() 获取区块链所有叶子节点，block hashes目前没有子节点，叶子节点在没有被finalized之前是不会被返回的。
- best_chain() 在这些“叶子”中，决定论地选择一个链作为创建新块的最佳链，并可能完成finalize。
- finality_target() 获取了最佳祖先的`target_hash`后，尝试去进行finalzie。

### consensus中import_queue

Import Queue primitivies: 可以验证和导入区块的东西。

这充当同步和导入之间的中间和抽象步骤。每种共识模式都有自己的区块验证需求。一些算法可以并行验证，而另一些只能按顺序验证。

`ImportQueue` trait允许例如验证策略去实例化，例如 `BasicQueue`和`BasicVerifier` trait允许串行队列被简单实例化。

	//	 验证一个区块的公正性
 	pub trait Verifier<B: BlockT>: Send + Sync {
 		// 验证给定的数据，返回BlockImportParams，然后一个可能新的验证者集合。
		fn verify()
	}

	// 区块导入队列API
	// ``import_*`方法被调用，为了去发送elements给导入对列去验证，因此,`poll_actions`方法去决定如何去响应这些elements。
	pub trait ImportQueue<B: BlockT>: Send {
		fn import_blocks()
		fn import_justification()
		fn import_finality_proof()
		fn poll_actions()	
	}


### PoC consensus

- primitives
	- pub trait TotalDifficulty 
		- fn increment()

	- decl_runtime_apis! 
		- pub trait TimestampApi
			- fn timestam()
		- pub trait DifficultyApi
			- fn difficulty()



> 要使用此引擎，您可能需要一个实现“PocAlgorithm”的结构。然后，将结构的一个实例以及其他必要的客户端引用传递给`import_queue`来设置队列。使用`start_mine`函数进行基本的CPU挖矿。

> PoC引擎的辅助存储只存储总难度。对于特定PoC算法的其他存储需求(例如每个特定块的实际难度)，可以在“算法”实现中采用客户端引用，并为辅助存储使用单独的前缀。也可以只使用运行时作为存储，但不建议这样做，因为它不适用于轻量级客户机。

- src
	- pub struct PocAux<Difficulty>
		- pub difficulty
		- pub total_difficulty
		- pub fn read()
	- pub trait PocAlgorithm<B: BlockT>
		- fn difficulty()
		- fn verify()
		- fn mine()
	- pub struct PocVerifier
		- pub fn new()
		- pub fn check_header()
		- fn check_inherents()
		- fn verify()
	- pub fn register_poc_inherent_data_provider()
	- pub type PocImportQueue
	- pub fn import_queue()
	- pub fn start_mine()
	

核心代码：

	let (difficulty,seal) = {
		let difficulty = algorithm.difficulty(&BlockId::Hash(best_hash))?;
		loop {
			let seal = algorithm.mine(&BlockId::Hash(best_hash),&header.hash(),difficulty,round)?;
			if let(seal) = seal {
				break (difficulty,seal)
			}
			if best_hash != client.info().best_hash {
				continue 'outer
			}
		}
	};
	

### scavenger


getMiningInfo

- height: (integer) 下一个区块的高度
- generationSignature (string) 现在区块生成的签名
- baseTarget (string) 现在区块base target，用来衡量全网总容量，也就是全网总算力，如果BaseTarget越小，说明全网容量越大。
- targetDeadline (number) 最大可接受的deadline

submitNonce

- account_id: (u64) 又可以称为Plotter ID，通过账号来P盘，是通过Curve25519算法来生成的字符串，又可以称为passphrase。
- nonce: (u64) 每一个nonce提供一个deadline，4个nonces是1M大小，每一个deadline是一个64bit无符号整数，是在0到2^64-1的范围内。deadlines是均匀且独立同分布的IID。扫描deadlines意味着寻找最小的deadline（best deadline）。
- height: (u64)
- block: (u64)
- deadline_unadjusted: (u64)
- deadline: (u64)
- gen_sig: ([u8;32])

plotfile存储的hash数据：

哈希数据有两个维度，分为行和列。每个数据列由两个32字节的哈希数据组成，总共64字节。每一行都有一个名为nonce的“行号”，总共有4096个列，256KiB。散列数据存储在plot文件中。

这些hash数据是由shabal256算法生成的。

	net difficulty(t) =  4398046511104 / 240 / baseTarget(t)
	genesis_base_target =  4398046511104 / 240 
	genesis_base_target = baseTarget(0)
	the expected minimum deadline of such a distribution is E(X) = (b+a*n)/(n+1)
	(a,b) is the range of possibile values for a deadline,n is the number of deadlines being scanned. 
	出块时间如果是240秒，那么希望E(X) = 240.
	basetarget 取决于出块时间：
	nonces = (2^64-1) / 240-1 = 76.861.433.640.456.500
	Or in TiB: 76.861.433.640.456.500 / 4 / 1024 / 1024 = 18.325.193.796 TiB
	

Seek Trait

提供一个游标，这个游标可以在字节流中移动。流通常具有固定的大小，允许相对于末端或当前偏移量进行查找。

seek_addr = u64::from(scoop) * nonces as u64 * SCOOP_SIZE; // SCOOP_SIZE = 64

Plot 

	pub struct Plot {
	    pub meta: Meta,
	    pub path: String,
	    pub fh: File,
	    read_offset: u64,
	    use_direct_io: bool,
	    sector_size: u64,
	    dummy: bool,
	}
	pub fn new()
	pub fn prepare()
	pub fn read()
	pub fn seek_random()
	

poc_hashing

	pub fn decode_gensig(gensig: &str) -> [u8;32]
	pub fn calculate_scoop(height:u64,gensig: &[u8;32]) -> u32
	pub fn find_best_deadline_rust(data: &[u8],number_of_nonces: u64,gensig: &[u8;32]) -> (u64,u64)
	
	
shabal256

	hash = shabal256_hash_fast(data,term) // data: [u8;64]  term: [u32,16]
	
	
第一步，get_mining_info()

第二步，根据signature和height计算scoop

第三步，根据height，block，base_target，scoop，signature来扫描读取

第四步，扫描出来nonce_data，计算出来的deadline，这个deadline比已经算出来的都要小，更新然后提交submit_nonce

### 算法流程：

- 第一步：钱包将通过Shabal256 hash function运行上一代签名和上一代区块生产者账号来创建新一代签名。
- 第二步：签名给到矿工，跟baseTarget和下一个区块高度。
- 第三步：在下一步中，通过对生成签名和从钱包接收到的区块高度运行Shabal256函数，矿工将生成下一个区块的生成散列。
- 第四步：生成散列用作modulo 4096函数的参数，以便获得将用于处理plot文件的scoop编号。
- 第五步：在scoop编号被计算之后，它被用来读取所有plot文件中所有nonces中的所有scoops，对于所有nonces的处理是单独完成的，通过使用新生成的签名通过Shabal256散列函数运行所有nonces。其结果是一个称为target的散列。目标除以步骤(2)中从钱包中获得的基本目标。除法结果的前8个字节为截止日期。
- 第六步：为了防止所谓的“nonce垃圾邮件”，矿工将检查最新发现的截止日期是否低于目前发现的最低截止日期，并继续进行，直到找到一个较低的截止日期。矿池通常会设置一个最大的截止日期限制(即池所接受的最大截止日期值)，超过这个限制的截止日期将被池丢弃，用于历史份额计算。矿工将把最后期限提交给钱包，连同绑定到plot文件的数字帐户ID，以及包含用于生成最后期限的scoop数据的nonce number。在单独挖掘的情况下，从矿工传递到钱包的信息还将包括绑定到plot文件的帐户的秘密密码(对于池挖掘，使用池帐户的密码)。
- 第七步：当钱包收到矿工的信息后，它将创建nonce来查找和验证矿工提交的最后期限。如果验证了最后期限，钱包将等待最后期限过期。
- 第八步：并检查新的有效块是否已经在网络上公布。
- 第九步：如果矿工提交了新的信息(一个新的截止日期)，钱包将创建nonce来检查截止日期的有效性，如果新提交的截止日期低于之前的截止日期。如果是这种情况，钱包将使用较低的截止日期值(即等待它过期)。如果新的区块还没有公布，并且截止日期已经过期，钱包将会锻造一个新的区块。

CalculateDeadline(prevBlockIndex,blockHeader) -> deadline

CalDL(int height, u256 generationSignature, u64 account_id, u64 nonce)

CalculateBaseTarget(prevBlockIndex,blockHeader) -> baseTarget


### deadlinecheck

height,scoop,deadline,elapsed,gensig_ok,poc_ok
108045,1772,105995016906613,18446744073709551210,false,false
133590,2469,116108231804711,18446744073709551387,false,false
96023,2574,40906671665829,18446744073709551224,false,false
84002,1576,498279513,18446744073707646628,false,false
93017,1084,89871299365285,18446744073709551220,false,false
138098,984,101448614857391,18446744073709551558,false,false


height,time,baseTarget,plotterId,nonce,generationSignature,deadline
0,1531292789,18325193796,0,0,0000000000000000000000000000000000000000000000000000000000000000,0
1,1531292790,18325193796,0,0,0000000000000000000000000000000000000000000000000000000000000000,0
2,1531292791,18325193796,0,0,0000000000000000000000000000000000000000000000000000000000000000,0
3,1531292792,18325193796,0,0,0000000000000000000000000000000000000000000000000000000000000000,0
4,1531292793,18325193796,0,0,0000000000000000000000000000000000000000000000000000000000000000,0

black animal clean inch clue grandma hill ugly sort message worry survive


./target/release/poc_checker check -h 577152 -i 10790126960500947771 -n 221298176 -f "10790126960500947771_221296000_4096" -w "https://burst.megash.it/burst"

基于Substrate的核心共识模块PoC（Proof of Capacity）取得重大突破。IPSE在打造底层基础共识的过程中，从基本的事实出发，上层应用在目前阶段无法充分使用机器存储空间，应用存储空间，应用PoC共识构建安全的底层共识层是IPSE追求的。同时在经济模型上，能够让矿工参与者更加分散和更高的收益。


p盘： engraver_gpu -i 10790126960500947771 -s 1 -n 4100


### 矿工请求挖矿信息

矿工向节点请求 `get_mine_info()` ，节点给矿工返回 `height`,`generation_signature`，`base_target`三个参数。

- height：下一区块高度。
- generation_signature：生成签名，是根据先前区块merkle root hash和区块高度，生成签名长度为32字节。
- base_target：根据最后n个区块计算出来的，用来调整矿工的难度。

### 矿工挖矿

矿工计算挖矿过程：`generation_signature和` `plot_id`（扫盘得到）`shabal256` 创建生成签名,矿工对哈希进行小规模数学计算，通过散列对4096取模，得到`scoop_number`。

读取plot文件，从所有的nonce中，获得scoop，处理这些scoop：

shabal256(scoop.hash,generation_signature)生成目标哈希，称为target，Target除base_target，得到的前8个字节就是deadline。

### 矿工提交挖矿信息

矿工向节点提交挖矿信息，submit_nonce(deadline,plot_id,scoop_number)。

### 节点处理deadline

节点收到提交信息后，创建对应的nonce，验证deadline，节点检查dealine对应时间的流逝，直到deadline对应的时间用光。如果在deadline之前在网络上收到其他节点的有效区块，则节点丢弃提交的Mining信息，当然，在这段时间内，还可以接收到其他的挖矿信息，验证后，只要最小的deadline。最后其他节点没出块，自己就可以出块。


难题一：节点rpc服务如何添加到pow，使用jsonrpc模块。

难题二：下一个区块高度和merkle root如何获取？

难题三：pow如何接收矿工提交的挖矿信息？







	


	




	
	
	
	
	
	