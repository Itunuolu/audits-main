## Nadosc
[Contest details](https://dashboard.hackenproof.com/user/programs/nado-smart-contracts)

### [High] Slow‑mode transaction deletion can permanently drop user actions and funds

**Target**
https://github.com/nadohq/nado-contracts

**Vulnerability details**

Code Reference: https://github.com/nadohq/nado-contracts/blob/main/core/contracts/Endpoint.sol#L286-L328 and
https://github.com/nadohq/nado-contracts/blob/main/core/contracts/Endpoint.sol#L300-L307

1. The slow‑mode queue deletes the transaction before attempting execution: `SlowModeTx memory txn = slowModeTxs[txUpTo]; delete slowModeTxs[txUpTo++]`.

2. `processSlowModeTransaction` is then called inside a `try/catch` where all reverts (except gas heuristics) are swallowed; no refund or requeue is done.

3. If the slow‑mode action reverts (e.g., transient health check failure, delisted product, bad params), the queued withdrawal/deposit/insurance action is permanently lost. Any slow‑mode fee (and insurance deposit taken at enqueue) cannot be recovered, violating liveness and causing user fund loss.


**Validation steps**
1. Queue a slow‑mode withdrawal via `submitSlowModeTransaction` (e.g., `WithdrawCollateral`).

2. Before the 3‑day delay elapses, manipulate account health or product state so the withdrawal will revert inside `clearinghouse.withdrawCollateral` (e.g., drop health below maintenance or delist the product).

3. After `executableAt`, call `executeSlowModeTransaction` (or let sequencer include `ExecuteSlowMode`).

4. Observe `_executeSlowModeTransaction` deletes the entry, the call reverts internally, catch block swallows the error, and the queue advances.

5. Verify the withdrawal never executes, the queue slot is gone, and the slow‑mode fee/funds taken at enqueue are unrecoverable.