# 共识机制


公链区块链都运行在公开的P2P网络上，维护着共同的全网账本。如何激励节点提供安全可靠的服务，解决拜占庭容错问题，以及提供高性能的处理。是当下所有区块链技术要解决的问题。去中心化，可扩展性，安全性，已成为目前“不可能三角问题”。

众多区块链在共识机制上，大致分为下面几种：
- POW（工作量证明）：以比特币为代表，利用计算能力来解决拜占庭容错问题，安全性最高，但是效率最低，非常消耗能源。
- POS（权益证明）：以比特股为代表，按照持有者币龄获取奖励，同时任意节点可以参与挖矿过程。
- DPOS（委托权益证明）： 以EOS为代表，社区投票选择21个节点行使记账权，轮流出块。当有作恶节点时，没收担保金和重新投票更换节点。

NEO实现了一种委托的改进拜占庭容错算法，具备⌊(n-1)/3⌋的节点容错能力，交易吞吐量实测环境中达到1000tps，未来有能力做到10000tps，支持大规模商业应用。结合POS与DPOS特性，持有NEO的用户根据币龄分红NeoGas（NEO网络费用代币），共识节点由持有NEO的用户实时投票选出，负责行使记账权，从而克服了POS的全节点可出块导致的性能低下，但共识节点的奖励并非区块奖励，而是网络手续费奖励，平衡生态激励。


