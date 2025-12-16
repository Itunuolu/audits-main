## OpenEden USDO
[Contest details](https://dashboard.hackenproof.com/user/programs/openeden-usdo-express-smart-contract-audit-contest)

### [High] Dust-Wrapping Bricks cUSDO & Steals Rescue Liquidity

**Target**
https://github.com/OpenEdenHQ/openeden.usdoexpress.audit/tree/f3f31d2ac15e3253cba342229f9d05495f95d6fd, cUSDO.deposit first-mint logic

Vulnerability Category: Logic Error – `contracts/tokens/cUSDO.sol,:63-138` & `contracts/tokens/USDO.sol,:174-212`

**Vulnerability Details**
When the `USDO` bonus multiplier ever exceeds 1 (the normal post-`addBonusMultiplier` state), `convertToShares(1)` returns 0. A user can call cUSDO.deposit(1, attacker) while the vault still has `totalSupply()==0`. `ERC4626` mints exactly one share in the happy-path first deposit, but safeTransferFrom pulls 0 USDO because the share conversion underflows to zero. The vault now has `totalSupply() > 0` yet `totalAssets() == 0`, so every future `deposit/mint` divides by zero and bricks the wrapper. If the treasury or any user later transfers USDO into the vault to restore service, that single share redemptions into the entire balance, letting the attacker drain all recapitalization liquidity.
If the treasury later pushes any USDO into the vault to revive it (or a legitimate depositor succeeds after manual top-up), that lone attacker share can be redeemed for the entire injected balance, turning the rescue into a full drain. The exploit is counterintuitive because it exploits the interaction between rebase-style share math and ERC4626’s first-deposit branch; nothing looks suspicious in the transaction (everything emits the usual events), yet it irreversibly DoSes the product and enables theft of any future liquidity that is added.