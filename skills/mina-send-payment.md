---
name: Unlock an account and send a Mina payment
description: Unlock a Mina account with its passphrase, submit a signed payment via GraphQL, then poll the transaction status.
api: graphql/mina-graphql-schema.json
operations: [unlockAccount, sendPayment, transactionStatus, lockAccount]
---

# Unlock an account and send a Mina payment

Write workflow against a running Mina daemon's GraphQL endpoint. The daemon
signs on behalf of accounts it holds, so the endpoint MUST stay bound to
localhost — exposing it lets anyone spend from unlocked accounts.

## Steps

1. **Unlock the sender.** Call `unlockAccount(input: { publicKey, password })`.
   The account must be unlocked before it can sign.
2. **Read the current nonce.** Query `account(publicKey)` for `nonce` (see the
   read skill) so the payment uses the correct nonce.
3. **Send the payment.** Call `sendPayment(input: { from, to, amount, fee, nonce })`.
   The daemon signs and broadcasts it; the response returns the payment `id`.
4. **Poll status.** Query `transactionStatus(payment: "<id>")` until it resolves
   to `INCLUDED` (or handle `UNKNOWN`/failure).
5. **Re-lock the account.** Call `lockAccount(input: { publicKey })` when done.

## Safety, conventions & errors

- There is no idempotency key; the network deduplicates by `nonce`. Reusing a
  confirmed nonce is rejected — see `conventions/mina-conventions.yml`.
- A locked account, insufficient balance, or a fee below the pool minimum causes
  rejection — see `errors/mina-problem-types.yml`.
- Signing/authorization model: `authentication/mina-authentication.yml`.
