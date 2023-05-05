GalloDaSballo

medium

# There is no on-chain connection between the `generationIds` and the `playerId` - The `playerId` can be manipulated

## Summary

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L162

In spite of the fact that the comment says that each player is uniquely identified by `generationIds`, in reality, they are identified by the order in which they are minted.

## Vulnerability Detail

Because of that, and because `generationIds` is not necessarily sorted, players with different `generationIds` can be minted to have interchangeable `tokenId`s based on the order in which they are minted.

This means:
1) The connection between the two values is non-existant on-chain and brittle even off-chain as we can manipulate it via sorting
2) This connection is in the hands of the minter and is not enforced by the contract

## Impact

Ordering can be altered to make sure that any `generationId` is matched with whichever `playerId` the caller wants

## Code Snippet

As you can see, no ordering is enforced since `generationId` is used at face value

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L183-L188

## Tool used

Manual Review

## Recommendation

Decide if you wish to mint a tokenId that is consistent with `generationId` instead of a counter
