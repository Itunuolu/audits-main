## Lumia
[Contest Details](https://dashboard.hackenproof.com/user/programs/lumia-smart-contract-audit-contest)

### [Critical] Direct-stake exit leaves principal locked in `DepositFacet`

**Target**
https://github.com/LumiaChain/HyperStaking-contracts/tree/master/contracts/hyperstaking

**Vulnerability details**
Users selecting a direct-stake strategy deposit tokens that remain escrowed inside the HyperStaking diamond. When `claimWithdraws` executes, it calls the direct strategy’s `claimExit`, which only marks the request claimed and returns an amount; no token transfer occurs anywhere in the flow. As a result, withdrawals appear to succeed (events emit, accounting updates), yet user funds never leave the contract, permanently locking principal and breaching liveness expectations.

**Vulnerability Category:**
`contracts/hyperstaking/facets/DepositFacet.sol:108`, `contracts/hyperstaking/strategies/DirectStakeStrategy.sol:57`, `contracts/hyperstaking/strategies/DirectStakeStrategy.sol:75`

**Validation Steps**
1. Configure a direct-stake vault and deposit stake through `directStakeDeposit`.
2. Queue a withdrawal via the protocol’s exit flow.
3. Invoke `claimWithdraws` after the unlock delay completes; observe that the ERC20 balance of the beneficiary never increases despite emitted `WithdrawClaimed`.

