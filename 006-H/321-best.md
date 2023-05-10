GalloDaSballo

high

# Newer proofs can mint older division and seasonIds

## Summary

No check is performed on the merkle root for `generationIds` and `seasonId` this allows minting of Players from older `seasonIds`

## Vulnerability Detail

Since the merkleRoot verification doesn't check for `generationIds` nor enforces that the mint is happening for the current `seasonId`, newer proofs can be used to mint Players from any generation and any season and not just the current one.

If for any reason it becomes better to mint players from older generations or older seasons, then lack of that check will allow to perform that minting, in spite of the fact that from a RolePlaying Point of View the players should have already matured away from the academy

## Impact

Older players can be minted

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L235-L243

```solidity
        if (
            !MerkleProofUpgradeable.verify(
                divisionProof,
                _clubDivsMerkleRoot,
                keccak256(abi.encodePacked(clubId, divisionTier))
            )
        ) {
            revert ClubNotInDivision(clubId, divisionTier);
        }
```


## Tool used

Manual Review

## Recommendation

Enforce that minting can only be performed on the current `seasonId`
