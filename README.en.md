# Agent Compute Mesh

This skill now takes a clear path: first turn outside-compute jobs into a product with proven value, then decide whether it deserves to become an open network. The preferred rollout is `local -> hosted -> community workers -> optional chain`.

It does not require remote nodes to see the whole task, and it does not let remote nodes pollute their own main context. It asks for stricter boundaries instead: bounded task slices, temporary execution leases, signed result bundles, and traceable settlement receipts.

Technical invocation name: `$agent-compute-mesh`.

## Experimental Status

- This is a `vibecoding` concept built through a few prompt iterations, document shaping, and light tests.
- It does not have verified security, and it does not have verified reliability.
- The protocol, token model, scheduling, execution isolation, and settlement logic here are still design drafts.
- Before any real use, it needs independent security review, adversarial testing, fault injection, economic simulation, and long-run validation.
- If someone uses this design directly and it breaks, that is their own responsibility.

## Design Focus

- Rollout priority: validate the product before decentralizing it.
- Task dispatch: the network broadcasts redacted work headers, not full prompts.
- Ephemeral execution: remote nodes must run accepted work inside temporary threads and temporary sandboxes.
- Result return: the network returns signed result bundles and billing receipts, while the local node decides whether to accept them.
- Settlement order: use credits and internal ledgers first, then discuss an on-chain token later.
- Network entry: later-joining nodes receive smaller starter credits by default, and those credits track marginal added compute.

## Current Rollout

Stage 1 should handle only public-data jobs such as official-doc verification, issue summaries, version-diff extraction, and public-web evidence packaging. Tasks that need private code, user secrets, customer data, or write access to the main workspace should stay local or inside operator-controlled hosting.

The preferred unit is one `exploration job`, not a whole agent session and not one tiny search call. One job should contain one problem, one version band, one evidence requirement, one budget, and one deadline, then split into `discovery / validation / synthesis` facets only when needed.

## Validation Metrics

- Whether users will pay for one `exploration job`.
- Median cost and margin per job.
- Quality of accepted evidence.
- Next-turn reuse rate.
- Fraud rate, mismatch rate, and refund rate.

## Late Join Decay

Later-joining nodes should receive less `warm_start_credit` by default. The more mature the network becomes, the smaller the marginal share that a single new node usually adds to total compute.

A stable default is:

`warm_start_credit = base_credit * era_decay * sqrt(join_bond / (active_bonded_compute + join_bond))`

This makes later, smaller, lower-impact joins start with a more conservative starter allowance.

The “every new join triggers a network-wide airdrop” path turns every join into a global inflation event and makes sybil splitting attractive. The stable path is to fund newcomer starter credits from `genesis_treasury` or a public treasury, while keeping incumbent rewards tied to real work, validation, relay, and archival duties.

## Execution Isolation

When a solver accepts work, the center of the protocol is isolation.

1. Open a fresh worker thread.
2. Open a temporary sandbox or isolated worktree.
3. Mount only the facet capsule, scoped tool tokens, and resource budgets for that lease.
4. Keep the main conversation, long-term memory, standing prompts, and unrelated workspace state out.
5. Return `result_bundle`, `sandbox_receipt`, and `billing_receipt`.
6. Destroy the worker thread and sandbox immediately.

## Protocol Files

- [SKILL.md](SKILL.md)
- [SKILL.en.md](SKILL.en.md)
- [references/travelnet-protocol.md](references/travelnet-protocol.md)
- [references/rollout-plan.md](references/rollout-plan.md)
- [references/job-spec.md](references/job-spec.md)
- [scripts/validate_travelnet_packet.py](scripts/validate_travelnet_packet.py)
- [assets/travelnet_join_example.json](assets/travelnet_join_example.json)
- [assets/travelnet_job_example.json](assets/travelnet_job_example.json)
- [assets/travelnet_settlement_example.json](assets/travelnet_settlement_example.json)

## Design Inputs

- [Bitcoin whitepaper](https://bitcoin.org/bitcoin.pdf)
- [Proof-of-stake rewards and penalties | ethereum.org](https://ethereum.org/developers/docs/consensus-mechanisms/pos/rewards-and-penalties/)
- [x/mint | Cosmos Docs](https://docs.cosmos.network/sdk/latest/modules/mint/README)
- [x/slashing | Cosmos Docs](https://docs.cosmos.network/sdk/latest/modules/slashing/README)
- [libp2p docs](https://libp2p.io/docs/)

## License

MIT
