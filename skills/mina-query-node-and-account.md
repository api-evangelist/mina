---
name: Query Mina node health and account state
description: Check that a Mina daemon is synced, then read on-chain account balance, nonce, and recent blocks via the GraphQL API.
api: graphql/mina-graphql-schema.json
operations: [daemonStatus, syncStatus, account, bestChain, pooledUserCommands]
---

# Query Mina node health and account state

Read-only workflow against a running Mina daemon's GraphQL endpoint
(`http://localhost:3085/graphql` by default). No authentication is required to
read; the endpoint is bound to localhost.

## Steps

1. **Confirm the node is synced.** Query `syncStatus` and `daemonStatus`. Proceed
   only when `syncStatus` is `SYNCED`; a non-synced node returns stale or partial
   data.
2. **Read the account.** Query `account(publicKey: "<B62...>")` for
   `balance { total }`, `nonce`, and `delegate`. The `nonce` is required to build
   the next transaction.
3. **Inspect recent chain state (optional).** Query `bestChain(maxLength: N)` for
   recent block heights and creators. Only the last `k` blocks in the transition
   frontier are available; use an Archive Node for older history.
4. **Estimate a fee (optional).** Query `pooledUserCommands { id fee }` to see
   pending transaction fees and pick a competitive fee.

## Conventions & errors

- Pagination is a bounded window (`maxLength`), not cursors — see
  `conventions/mina-conventions.yml`.
- Failures return a GraphQL `errors[]` array, not HTTP problem+json — see
  `errors/mina-problem-types.yml`.
