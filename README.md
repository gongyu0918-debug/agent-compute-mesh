# Agent 算力分布网络 / Agent Compute Mesh

这个 skill 描述的是一个全球分布式 agent 算力网络草案：本地节点把任务拆开、脱敏、广播，再让远程节点在临时线程和临时沙箱里代为执行。结果回来后，本地节点再决定是否接受、是否合并、是否结算。  
This skill describes a draft for a global distributed agent compute network: the local node splits a task, redacts it, broadcasts it, lets remote nodes execute inside ephemeral threads and sandboxes, then decides locally whether to accept, merge, or settle the result.

它不要求远程节点看到完整任务，也不让远程节点污染自己的主上下文。它要求的是更严格的边界：局部任务切片、临时执行租约、签名结果包、可追溯结算回执。  
It does not require remote nodes to see the whole task, and it does not let remote nodes pollute their own main context. It asks for stricter boundaries instead: bounded task slices, temporary execution leases, signed result bundles, and traceable settlement receipts.

技术调用名：`$agent-compute-mesh`。  
Technical invocation name: `$agent-compute-mesh`.

## 实验状态 / Experimental Status

- 这是一次 `vibecoding` 构想，基于几轮 prompt 调整、文档整理和轻量测试。 / This is a `vibecoding` concept built through a few prompt iterations, document shaping, and light tests.
- 它还不具备可验证的安全性，也不具备可验证的可靠性。 / It does not have verified security, and it does not have verified reliability.
- 这里的协议、代币、调度、执行隔离和结算机制都还是设计稿。 / The protocol, token model, scheduling, execution isolation, and settlement logic here are still design drafts.
- 真正投入使用前，至少还需要独立安全审计、对抗测试、故障注入、经济学仿真和长期运行验证。 / Before any real use, it needs independent security review, adversarial testing, fault injection, economic simulation, and long-run validation.
- 如果有人直接拿这套设计去跑，出了问题自己负责。 / If someone uses this design directly and it breaks, that is their own responsibility.

## 设计重点 / Design Focus

- 任务分发：广播的是脱敏工作头，不是完整 prompt。 / Task dispatch: the network broadcasts redacted work headers, not full prompts.
- 临时执行：远程节点接单后必须在临时线程和临时沙箱里运行。 / Ephemeral execution: remote nodes must run accepted work inside temporary threads and temporary sandboxes.
- 结果回收：返回的是签名结果包和计费回执，本地节点决定是否接受。 / Result return: the network returns signed result bundles and billing receipts, while the local node decides whether to accept them.
- 代币结算：`TRV` 是工作量凭证，主要由 `reward_lock` 驱动，补充发行只做 treasury 回填。 / Token settlement: `TRV` is proof of work contribution, driven mainly by `reward_lock`, while fresh issuance only refills treasury pools.
- 新人入网：晚加入节点默认拿更少的启动额度，额度和边际新增算力挂钩。 / Network entry: later-joining nodes receive smaller starter credits by default, and those credits track marginal added compute.

## 晚加入衰减 / Late Join Decay

晚加入节点默认应该拿更少的 `warm_start_credit`。理由很简单：网络越成熟，新节点对总算力的边际提升占比通常越小。  
Later-joining nodes should receive less `warm_start_credit` by default. The reason is simple: the more mature the network becomes, the smaller the marginal share that a single new node usually adds to total compute.

一个稳定的默认公式是：  
A stable default is:

`warm_start_credit = base_credit * era_decay * sqrt(join_bond / (active_bonded_compute + join_bond))`

这会让更晚加入、质押更小、边际贡献更低的节点拿到更保守的启动额度。  
This makes later, smaller, lower-impact joins start with a more conservative starter allowance.

“每来一个新节点，就给全网每个节点发一轮币”这条路会把每次入网都变成一次全网通胀事件，也会让 sybil 分身更有利可图。网络更稳定的做法，是把启动额度从 `genesis_treasury` 或公共 treasury 里按规则发给新节点，把既有节点的收益继续绑定在真实工作、验证、中继和归档上。  
The “every new join triggers a network-wide airdrop” path turns every join into a global inflation event and makes sybil splitting attractive. The stable path is to fund newcomer starter credits from `genesis_treasury` or a public treasury, while keeping incumbent rewards tied to real work, validation, relay, and archival duties.

## 执行隔离 / Execution Isolation

接单节点执行任务时，协议中心不是搜索，中心是隔离。  
When a solver accepts work, the center of the protocol is isolation.

1. 新建临时 worker 线程。  
2. 新建临时沙箱或隔离 worktree。  
3. 只注入这一单的分面胶囊、能力范围内的工具令牌、时间和资源配额。  
4. 禁止挂载主对话、长期记忆、常驻 prompt 和无关工作区状态。  
5. 返回 `result_bundle`、`sandbox_receipt`、`billing_receipt`。  
6. 线程和沙箱立刻销毁。  

1. Open a fresh worker thread.  
2. Open a temporary sandbox or isolated worktree.  
3. Mount only the facet capsule, scoped tool tokens, and resource budgets for that lease.  
4. Keep the main conversation, long-term memory, standing prompts, and unrelated workspace state out.  
5. Return `result_bundle`, `sandbox_receipt`, and `billing_receipt`.  
6. Destroy the worker thread and sandbox immediately.

## 协议文件 / Protocol Files

- [SKILL.md](SKILL.md)
- [SKILL.en.md](SKILL.en.md)
- [references/travelnet-protocol.md](references/travelnet-protocol.md)
- [scripts/validate_travelnet_packet.py](scripts/validate_travelnet_packet.py)
- [assets/travelnet_join_example.json](assets/travelnet_join_example.json)
- [assets/travelnet_job_example.json](assets/travelnet_job_example.json)
- [assets/travelnet_settlement_example.json](assets/travelnet_settlement_example.json)

## 设计输入 / Design Inputs

- [Bitcoin whitepaper](https://bitcoin.org/bitcoin.pdf)
- [Proof-of-stake rewards and penalties | ethereum.org](https://ethereum.org/developers/docs/consensus-mechanisms/pos/rewards-and-penalties/)
- [x/mint | Cosmos Docs](https://docs.cosmos.network/sdk/latest/modules/mint/README)
- [x/slashing | Cosmos Docs](https://docs.cosmos.network/sdk/latest/modules/slashing/README)
- [libp2p docs](https://libp2p.io/docs/)

## License

MIT
