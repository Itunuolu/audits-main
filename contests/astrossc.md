## astrossc
[Contest details](https://dashboard.hackenproof.com/user/programs/astros-smart-contracts)

### [High] Cross-vault withdrawal signature replay drains vault balances.

**Target**
https://github.com/naviprotocol/astros-contracts/tree/main/perp_vault, ActiveVault.withdraw signature verification flow.
`perp_vault/sources/active_vault.move:214`, `perp_vault/sources/active_vault.move:609`


**Vulnerability details**
The keccak message used for multi-sig withdrawal approval omits the `ActiveVault` address. Each vault keeps independent `used_hashes` / `withdrawal_order_ids`, so a single approved signature set for coin type T can be replayed against every other `ActiveVault<T>` sharing the same Config. An attacker who receives one legitimate withdrawal authorization can resubmit the identical payload across richer vaults, draining all liquidity without any privileged access.

**Validation step**s
1. Deploy two `ActiveVault<T>` instances under the same `Config`; fund both.
2. Obtain a valid signature bundle for withdrawing amount X from vault A (normal cooperative flow).
3. Before vault B signs anything, call `user_entry::withdraw` on vault B with the exact same (`order_id, order_time, amount, recipient, signatures, message`); observe B honors the withdrawal.
4. Repeat across additional vaults until balances are exhausted.