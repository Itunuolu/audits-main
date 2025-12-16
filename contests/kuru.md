## Kinetiq
[Contest Details](https://code4rena.com/audits/2025-04-kinetiq)

### [High] Flip Order Maker Rebate Accounting Black Hole

**Finding description**

The `_matchAggressiveBuy` and `_matchAggressiveSell` functions calculate fees using a split: `feeCollected += ((_feeDebit * (_takerFeeBps - makerFeeBps)) / _takerFeeBps)`. This calculation implicitly assumes the remaining `_feeDebit * makerFeeBps / _takerFeeBps` is credited as a maker rebate via the `_creditMaker` function.

However, for flip orders, `_creditMaker` is explicitly skipped in `_fillOrder` because "Flip orders are not eligible for maker rebates". The `_handleFlipOrderUpdate` function manages the flip order's state but does not contain logic to handle the `makerFeeBps` portion of the fee. Consequently, this portion of the taker fee, deducted from the taker, is neither credited to the maker nor added to the protocol's `baseFeeCollected` or `quoteFeeCollected` balances.

This breaks the fundamental accounting security guarantee that all funds entering or leaving the system are correctly tracked and assigned. Funds equivalent to the unclaimed maker rebate for flip fills exist within the contract's balance but are unaccounted for in the internal records, creating a "black hole".

A malicious user cannot directly exploit this to steal funds, but it represents a silent loss of funds for makers and protocol revenue, and a persistent accounting imbalance.

**Impact**

- Silent Financial Loss: Makers whose standard orders are filled by flip orders do not receive their entitled maker rebate.
- Protocol Revenue Understatement: The protocol collects less revenue than expected, as the `makerFeeBps` portion for flip fills is not added to `feeCollected`.
- Accounting Inaccuracy: Funds are present in the contract but unaccounted for, leading to potential long-term PnL drift and reconciliation difficulties.
- While it doesn't lead to direct theft, the persistent misallocation of funds and incorrect accounting represent a significant issue.


**Recommendation**

Modify the fee calculation logic in `_matchAggressiveBuy` and `_matchAggressiveSell` when the matching order is identified as a flip order (i.e., `s_orders[_orderId].flippedPrice != 0`). In this specific case, the entire `_feeDebit` should be attributed to the protocol, as no maker rebate is intended for flip orders.

Essentially, change the fee split logic to check if the matched order is a flip order. If it is, the protocol should collect the full `_feeDebit`. If not, the existing split logic applies.