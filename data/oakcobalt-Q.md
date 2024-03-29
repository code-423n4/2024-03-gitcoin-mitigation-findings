# Lines of code

https://github.com/gitcoinco/id-staking-v2/blob/7c19717aeab91a0166fc1ca50f443ee2ce7483f0/contracts/IdentityStaking.sol#L363-L364


# Vulnerability details

In `multipleCommunityStakes()`, each for-loop iteration will call `_communityStake()` which performs an external call for token transfer from `msg.sender` to `address(this)`. This is wasteful because each external call (`token.transferFrom()`) incurs overhead gas (encoding function data + creating external call). 
```solidity
  function multipleCommunityStakes(
    address[] calldata stakees,
    uint88[] calldata amounts,
    uint64[] calldata durations
  ) external whenNotPaused {
...
    for (uint i = 0; i < stakees.length; i++) {
 |>     _communityStake(stakees[i], amounts[i], durations[i]);
    }
  }

  function _communityStake(address stakee, uint88 amount, uint64 duration) private {
...
|>  if (!token.transferFrom(msg.sender, address(this), amount)) {
      revert FailedTransfer();
    }
  }
```
Recommendations:
(1)In `multipleCommunityStakes()`, refactor `token.transferFrom()` call outside of `_communityStake()`and for-loop, similar to current `withdrawMultipleCommunityStakes()`.
(2)In `communityStake()`, add token transfer after `_communityStake`, similar to current `withdrawCommunityStake()`. 



## Assessed type

Other