# Agent 算力分布网络 / Agent Compute Mesh

这个 skill 当前主攻的方向很明确：先把外部算力任务做成能验证价值的产品，再决定要不要把它扩成开放网络。现在的首选路线是 `local -> hosted -> community workers -> optional chain`。  
This skill now takes a clear path: first turn outside-compute jobs into a product with proven value, then decide whether it deserves to become an open network. The preferred rollout is `local -> hosted -> community workers -> optional chain`.

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

- 路线优先级：先做产品验证，再做去中心化。 / Rollout priority: validate the product before decentralizing it.
- 任务分发：广播的是脱敏工作头，不是完整 prompt。 / Task dispatch: the network broadcasts redacted work headers, not full prompts.
- 临时执行：远程节点接单后必须在临时线程和临时沙箱里运行。 / Ephemeral execution: remote nodes must run accepted work inside temporary threads and temporary sandboxes.
- 结果回收：返回的是签名结果包和计费回执，本地节点决定是否接受。 / Result return: the network returns signed result bundles and billing receipts, while the local node decides whether to accept them.
- 结算顺序：先用 credits 和内部账本，再讨论链上代币。 / Settlement order: use credits and internal ledgers first, then discuss an on-chain token later.
- 新人入网：晚加入节点默认拿更少的启动额度，额度和边际新增算力挂钩。 / Network entry: later-joining nodes receive smaller starter credits by default, and those credits track marginal added compute.

## 当前落地 / Current Rollout

第一阶段只做公开数据任务，比如官方文档核对、issue 汇总、版本差异提取、公开网页证据打包。真正需要私有代码、用户密钥、客户数据、主工作区写权限的任务，继续留在本地或自控托管环境。  
Stage 1 should handle only public-data jobs such as official-doc verification, issue summaries, version-diff extraction, and public-web evidence packaging. Tasks that need private code, user secrets, customer data, or write access to the main workspace should stay local or inside operator-controlled hosting.

推荐的任务粒度是一整个 `exploration job`，不是整段 agent 会话，也不是一条极小的搜索调用。一个 job 里包含一个问题、一个版本带、一个证据要求、一个预算、一个截止时间，必要时再拆成 `discovery / validation / synthesis` 三类 facet。  
The preferred unit is one `exploration job`, not a whole agent session and not one tiny search call. One job should contain one problem, one version band, one evidence requirement, one budget, and one deadline, then split into `discovery / validation / synthesis` facets only when needed.

## 验证指标 / Validation Metrics

- 用户是否愿意为一次 `exploration job` 付费。 / Whether users will pay for one `exploration job`.
- 单个 job 的中位成本和毛利。 / Median cost and margin per job.
- 被接受证据的质量。 / Quality of accepted evidence.
- 下一轮任务复用率。 / Next-turn reuse rate.
- 欺诈率、错配率和退款率。 / Fraud rate, mismatch rate, and refund rate.

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
- [references/rollout-plan.md](references/rollout-plan.md)
- [references/job-spec.md](references/job-spec.md)
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
