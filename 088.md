MiloTruck

medium

# Users might lose funds as `claimERC20Prize()` doesn't revert for no-revert-on-transfer tokens

## Summary

Users can call `claimERC20Prize()` without actually receiving tokens if a no-revert-on-failure token is used, causing a portion of their claimable tokens to become unclaimable.

## Vulnerability Detail

In the `FootiumPrizeDistributor` contract, whitelisted users can call `claimERC20Prize()` to claim ERC20 tokens. The function adds the amount of tokens claimed to the user's total claim amount, and then transfers the tokens to the user:

[FootiumPrizeDistributor.sol#L128-L131](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L128-L131)

```solidity
if (value > 0) {
    totalERC20Claimed[_token][_to] += value;
    _token.transfer(_to, value);
}
```

As the the return value from `transfer()` is not checked, `claimERC20Prize()` does not revert even when the transfer of tokens to the user fails.

This could potentially cause users to lose assets when:
1. `_token` is a no-revert-on-failure token.
2. The user calls `claimERC20Prize()` with `value` higher than the contract's token balance.

As the contract has an insufficient balance, `transfer()` will revert and the user receives no tokens. However, as `claimERC20Prize()` succeeds, `totalERC20Claimed` is permanently increased for the user, thus the user cannot claim these tokens again.

## Impact

Users can call `claimERC20Prize()` without receiving the token amount specified. These tokens become permanently unclaimable for the user, leading to a loss of funds.

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L128-L131

## Tool used

Manual Review

## Recommendation

Use `safeTransfer()` from Openzeppelin's [SafeERC20](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20#SafeERC20) to transfer ERC20 tokens. Note that [`transferERC20()`](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L105-L111) in `FootiumEscrow.sol` also uses `transfer()` and is susceptible to the same vulnerability.