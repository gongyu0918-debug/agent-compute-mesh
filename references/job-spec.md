# Job Spec

Use this file when `agent-compute-mesh` needs to decide task size, price, and split strategy.

## Preferred Unit

Use one `exploration job` as the preferred unit.

Each job should contain:

- one problem statement
- one host or product family
- one version band
- one evidence requirement
- one search or execution budget
- one deadline

## Facet Types

- `discovery`: find candidate sources or paths
- `validation`: check source quality, version fit, and official grounding
- `synthesis`: convert accepted evidence into a usable operator result

## Too Large

Do not ship these as one external job:

- full-agent session replay
- unrestricted codebase mutation
- cross-product incident investigation with many moving parts

## Too Small

Do not ship these as standalone priced jobs:

- one search query
- one fetch call
- one short summarization call

These are better as internal steps inside a larger `exploration job`.

## Pricing Inputs

- estimated latency
- evidence depth
- number of facets
- redundancy level
- review overhead
- privacy level
