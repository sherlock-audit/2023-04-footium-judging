GalloDaSballo

medium

# `changeMaxGenerationId` allows to mint tokens from older generations retroactively

## Summary

`changeMaxGenerationId` is meant to allow more minting for the current generation, however, because of the fact that merkleProofs do not validate for the current season, whenever the limit will be raised, the limit will allow to mint players from older seasons

It will unlock a lot more minting than it may be intended

## Vulnerability Detail

If for any reason `maxGenerationId` is increased, because of the fact that `MerkleProofUpgradeable.verify` doesn't check for `generationIds` and `seasonId` then not only new players from the current season can be minted, but also players from older seasons.

## Impact

Minting of X tokens where X is the number of old seasons

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L257-L269

## Tool used

Manual Review

## Recommendation

Enforce minting exclusively for the current season or change validation to validate the generations and seasons being minted