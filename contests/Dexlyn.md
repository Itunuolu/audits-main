## Dexlyn
[Contest Details](https://dashboard.hackenproof.com/user/programs/dexlyn-smart-contract-audit-contest)

### [Critical] Unchecked reward asset during reward claim allows withdrawing the wrong token from pool reserves (affects pool’s paired assets and any FA held by the pool).

**Target**
https://github.com/DexlynLabs/CLMM_Dex/tree/a1bd65e84ceb354ea0fa6683d65a738700d82a63

**Vulnerability Details**

The reward-claim function trusts a user-supplied `asset_addr` when transferring owed rewards. Instead of enforcing the configured rewarder asset for the given `rewarder_index`, the function withdraws from whatever FA store matches the caller’s `asset_addr`. An LP with accrued rewards can therefore claim in asset A or asset B (or any FA the pool holds), draining pool reserves by up to the owed amount per claim.
**Impact**: Direct, non-privileged theft from pool reserves equal to the attacker’s accrued reward balance each claim; breaks asset-type invariant for rewarders; fund mis-accounting.


**Validation Steps**

1. Initialize a pool with tokens A/B.
2. Configure a rewarder that pays in token R and accrue rewards for an LP position.
3. Call `clmm_router::collect_rewarder(pool_id, rewarder_index, position_id, asset_addr = address_of_token_A)`.
4. Observe: LP receives A (not R); pool’s A reserve decreases by `amount_owed`; the rewarder index still corresponds to R.
5. Repeat with `asset_addr = address_of_token_B`; pool’s B reserve decreases accordingly.