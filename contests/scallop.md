## scallop
[Bug Bounty details](https://dashboard.hackenproof.com/user/programs/scallop-protocol-smart-contract)

### [Medium] Collateral Dust Poisoning Blocks Borrowing (protocol obligations)

**Target**
https://github.com/scallop-io/sui-lending-protocol 


**Vulnerability details**
`deposit_collateral` (https://github.com/scallop-io/sui-lending-protocol/blob/main/contracts/protocol/sources/user/deposit_collateral.move#L31-L79), and (https://github.com/scallop-io/sui-lending-protocol/blob/main/contracts/protocol/sources/user/borrow.move#L268-L276) never verifies that the caller owns the target obligation (`no obligation::assert_key_match`). Any whitelisted account—including everyone if `whitelist::allow_all` is enabled—can call `deposit_collateral<T>` on a victim’s obligation and add a tiny balance of asset `T`. Later, when the legitimate owner tries to borrow `T`, `borrow_internal` checks `obligation::has_coin_x_as_collateral` and aborts with `unable_to_borrow_a_collateral_coin`. The owner experiences a normal borrow flow that unexpectedly reverts because another address “poisoned” state with dust. The attacker can add 1 wei for every listed asset at negligible cost and keep the victim from borrowing those assets indefinitely, forcing manual withdrawal of the unwanted collateral first.

**Validation steps**
1. Open (or obtain) a victim obligation ID and key; attacker must only be whitelisted.

2. Mint/dust a tiny Coin (e.g. 1 wei of USDC) and call `deposit_collateral<T>(version, &mut victim_obligation, &mut market, coin, ctx)` owner key is never requested.

3. Victim now calls `borrow<T>` per the normal entry point: after passing all health checks, the flow hits `assert!(!obligation::has_coin_x_as_collateral(...))` in `borrow_internal` and reverts with `unable_to_borrow_a_collateral_coin`.

4. Dust persists; repeat per asset to grief more markets. Victim must withdraw each dust collateral (incurring oracle/clock overhead) before they can borrow that asset again.