DevABDee

medium

# `abi.encodePacked` Allows Hash Collision

## Summary
`abi.encodePacked` results in Hash Collision sometimes when two dynamic argumants are encoded with it.
From the [solidity documentation](https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=collisions#non-standard-packed-mode):
>If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of an into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c").

## Vulnerability Detail
As the solidity docs describe, two or more dynamic types are passed to abi.encodePacked. Moreover, these dynamic values are user-specified function arguments in external functions, meaning anyone can directly specify the value of these arguments when calling the function. 
Many users can fail an issue while claiming their prizes because of this.

## Impact
- Alice, a very deserving user, calls [claimERC20Prize()](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L106) or [claimETHPrize()](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L144) to claim it's prize
- But due to abi.encodePacked's hash collision [verification](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L117) fails
- And Alice is unable to cliam his prize.

## Code Snippet
1. https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L120
2. https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L157
```solidity
    function claimETHPrize(
        address _to,
        uint256 _amount,
        bytes32[] calldata _proof
    ) external whenNotPaused nonReentrant {
        if (_to != msg.sender) {
            revert InvalidAccount();
        }

        if (
            !MerkleProofUpgradeable.verify(
                _proof,
                ethMerkleRoot,
                //@audit M abi.encodePacked Allows Hash Collision
                keccak256(abi.encodePacked(_to, _amount))
            )
        ) {
            revert InvalidETHMerkleProof();
        }

        uint256 value = _amount - totalETHClaimed[_to];

        if (value > 0) {
            totalETHClaimed[_to] += value;

            (bool sent, ) = _to.call{value: value}("");
            if (!sent) {
                revert FailedToSendETH(value);
            }
        }

        emit ClaimETH(_to, value);
    }
```

## Tool used

Manual Review

## Recommendation
Use `abi.encode()` instead of `abi.encodePacked()`, which will prevent hash collisions

Reference: https://github.com/sherlock-audit/2022-10-nftport-judging/issues/118
