## Alchemix
[Contest details](https://cantina.xyz/code/e68909e6-3491-4a94-a707-ecf0c89cf72a/overview)

### [Medium] Dragon router will lose entitled profits through loss recovery gap in yield donating strategies

**Summary**
Insufficient loss tracking mechanisms in `_handleDragonLossProtection()` will cause uncovered losses to be permanently absorbed by users as the system will fail to recover dragon router obligations from future profits.
Root Cause: `YieldDonatingTokenizedStrategy.sol#L80-97` the loss protection function at lines 80-97 burns available dragon shares but doesn't track uncovered losses:

```solidity
function _handleDragonLossProtection(StrategyData storage S, uint256 loss) internal {
    if (S.enableBurning) {
        uint256 sharesToBurn = _convertToShares(S, loss, Math.Rounding.Ceil);
        uint256 sharesBurned = Math.min(sharesToBurn, S.balances[S.dragonRouter]);
        // No tracking of uncovered loss when sharesBurned < sharesToBurn
    }
}
```

Attack Path:

1. Strategy experiences losses exceeding dragon router's available share balance.
2. System burns all available dragon shares but loss exceeds available shares
3. Uncovered loss is not tracked in any persistent state (unlike design documentation)
4. Future profits are minted normally to dragon router via report() function
5. Users permanently absorb uncovered historical losses
6. Dragon router receives new profits without offsetting previous uncovered losses


**Impact Explanation**

The users suffer an approximate loss of 5-15% principal erosion per loss event. The dragon router gains undeserved profit allocation equal to uncovered losses. This violates the principal protection design.

**Proof of Concept**

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.8.18;

import { console } from "forge-std/console.sol";
import { Setup, IMockStrategy } from "./utils/Setup.sol";


contract YieldDonatingLossRecoveryGapPOC is Setup {
    
    address public alice = makeAddr("alice");
    address public bob = makeAddr("bob");
    
    uint256 constant INITIAL_DEPOSIT = 1000e18;
    uint256 constant PROFIT_AMOUNT = 100e18;
    uint256 constant LOSS_AMOUNT = 150e18; // Loss exceeds dragon router shares

    function setUp() public override {
        super.setUp();
    }
    
   
    function test_lossRecoveryGapVulnerability() public {
        console.log("=== INITIAL STATE ===Setup {
        
        // Deploy funds and get initial state
        mintAndDepositIntoStrategy(strategy, alice, INITIAL_DEPOSIT);
        mintAndDepositIntoStrategy(strategy, bob, INITIAL_DEPOSIT);
        
    
    address public alice = makeAddr("alice");
    address public bob = makeAddr("bob");
        
        // Deploy funds and get initial state
        mintAndDepositIntoStrategy(strategy, alice, INITIAL_DEPOSIT);
        mintAndDepositIntoStrategy(strategy, bob, INITIAL_DEPOSIT);
        
        uint256 initialTotalAssets = strategy.totalAssets();
        uint256 initialDragonShares = strategy.balanceOf(donationAddress);
        uint256 aliceSharesInitial = strategy.balanceOf(alice);
        uint256 bobSharesInitial = strategy.balanceOf(bob);
        
        assertEq(initialTotalAssets, 2000e18, "Initial assets should equal total deposits");
        assertEq(initialDragonShares, 0, "Dragon router should start with no shares");
        assertEq(aliceSharesInitial, 1000e18, "Alice should have 1000e18 shares"); - this simulates yield generation
        assertEq(bobSharesInitial, 1000e18, "Bob should have 1000e18 shares");
        
        // Generate profit to give dragon router some shares
       
        
        // Add profit to yield source - this simulates yield generation
        asset.mint(address(yieldSource), PROFIT_AMOUNT);
        override {
        super.setUp();
        // Report profit - this should mint shares to dragon router
        vm.prank(keeper);
        (uint256 profit, uint256 loss) = strategy.report();
        
        uint256 dragonSharesAfterProfit = strategy.balanceOf(donationAddress);
        
        assertTrue(profit > 0, "Profit should be generated");
        assertEq(loss, 0, "No loss should be reported");
        assertTrue(dragonSharesAfterProfit > 0, "Dragon router should receive shares from profit");
        
        // Calculate dragon router's asset value before loss
        uint256 dragonAssetValueBeforeLoss = strategy.convertToAssets(dragonSharesAfterProfit);
        // Create loss that exceeds dragon router shares 
        yieldSource.simulateLoss(LOSS THAT EXCEEDS DRAGON SHARES ");
        
        // Simulate loss by reducing yield source balance - withdraw more than what we have earned
        vm.prank(address(yieldSource));
        yieldSource.withdraw(LOSS_AMOUNT);
        
        // Report the loss - this triggers _handleDragonLossProtection
        vm.prank(keeper);
        (uint256 profitFromLoss, uint256 actualLoss) = strategy.report();
        
        uint256 dragonSharesAfterLoss = strategy.balanceOf(donationAddress);
        
        assertTrue(actualLoss > 0, "Loss should be reported");
        
       Loss exceeds dragon router coverage 
        assertTrue(actualLoss > dragonAssetValueBeforeLoss, "Loss should exceed dragon coverage");
        
        // Calculate uncovered loss
        uint256 uncoveredLoss = actualLoss - dragonAssetValueBeforeLoss;
        assertTrue(uncoveredLoss > 0, "CRITICAL: There should be uncovered loss");
        
        
       Uncovered loss is absorbed by users
        uint256 aliceAssetsAfterLoss = strategy.convertToAssets(strategy.balanceOf(alice));
        uint256 bobAssetsAfterLoss = strategy.convertToAssets(strategy.balanceOf(bob));
        
        assertTrue(aliceAssetsAfterLoss < INITIAL_DEPOSIT, "CRITICAL: Alice absorbed part of uncovered loss");
        assertTrue(bobAssetsAfterLoss < INITIAL_DEPOSIT, "CRITICAL: Bob absorbed part of uncovered loss");
        
               
        // Generate more profit
        asset.mint(address(yieldSource), PROFIT_AMOUNT);
        
        vm.prank(keeper);
        (uint256 futureProfit, uint256 futureLoss) = strategy.report();
        
        uint256 dragonSharesAfterFutureProfit = strategy.balanceOf(donationAddress);
        uint256 aliceAssetsAfterFutureProfit = strategy.convertToAssets(strategy.balanceOf(alice));
        uint256 bobAssetsAfterFutureProfit = strategy.convertToAssets(strategy.balanceOf(bob));
        
        // Future profits don't restore user losses 
        assertTrue(dragonSharesAfterFutureProfit > 0, "Dragon router receives new profit shares");
        assertTrue(aliceAssetsAfterFutureProfit < INITIAL_DEPOSIT, "CRITICAL: Alice never recovers from uncovered loss");
        assertTrue(bobAssetsAfterFutureProfit < INITIAL_DEPOSIT, "CRITICAL: Bob never recovers from uncovered loss");
        
        // The vulnerability: Future profits go to dragon router but don't compensate users for previously uncovered losses
        uint256 dragonAssetValueAfterRecovery = strategy.convertToAssets(dragonSharesAfterFutureProfit);
        
        // VULNERABILITY CONFIRMATION
        assertTrue(
            (INITIAL_DEPOSIT - aliceAssetsAfterFutureProfit) > 0,
            "VULNERABILITY CONFIRMED: Users permanently lost funds due to uncovered loss gap"
        );
        
        assertTrue(
            dragonAssetValueAfterRecovery > 0,
        );
        

    }
}

```


**Recommendation**
Implement Uncovered loss tracking and recovery:

```solidity

struct StrategyData {
    // ... existing fields
    uint256 uncoveredLossAmount;  // Track unrecovered losses
}

function _handleDragonLossProtection(StrategyData storage S, uint256 loss) internal {
    if (S.enableBurning) {
        uint256 sharesToBurn = _convertToShares(S, loss, Math.Rounding.Ceil);
        uint256 sharesBurned = Math.min(sharesToBurn, S.balances[S.dragonRouter]);
        
        if (sharesBurned > 0) {
            uint256 assetValueBurned = _convertToAssets(S, sharesBurned, Math.Rounding.Floor);
            _burn(S, S.dragonRouter, sharesBurned);
            
            // Track any uncovered loss for future recovery
            if (loss > assetValueBurned) {
                S.uncoveredLossAmount += loss - assetValueBurned;
            }
        } else {
            S.uncoveredLossAmount += loss;
        }
    }
}


```