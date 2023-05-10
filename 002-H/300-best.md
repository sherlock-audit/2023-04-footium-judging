0x52

high

# Merkle leaf values for _clubDivsMerkleRoot are 64 bytes before hashing which can lead to merkle tree collisions

## Summary

FootiumAcademy hashes 64 bytes when calculating leaf allowing it to collide with the internal nodes of the merkle tree.

## Vulnerability Detail

MerkleProofUpgradeable.sol puts the following warning at the beginning of the contract:

     * WARNING: You should avoid using leaf values that are 64 bytes long prior to
     * hashing, or use a hash function other than keccak256 for hashing leaves.
     * This is because the concatenation of a sorted pair of internal nodes in
     * the merkle tree could be reinterpreted as a leaf value.

[FootiumAcademy.sol#L235-L240](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L235-L240)

        if (
            !MerkleProofUpgradeable.verify(
                divisionProof,
                _clubDivsMerkleRoot,
                keccak256(abi.encodePacked(clubId, divisionTier)) <- @audit-issue 64 bytes before hashing allows collisions with internal nodes
            )

This is problematic because FootiumAcademy uses clubId and divisionTier as the base of the leaf, which are both uint256 (32 bytes each for 64 bytes total). This allows collision between leaves and internal nodes. These collisions could allow users to mint to divisions that otherwise would be impossible.

## Impact

Users can abuse merkle tree collisions to mint in non-existent divisions and bypass minting fees

## Code Snippet

[FootiumAcademy.sol#L228-L272](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L228-L272)

## Tool used

Manual Review

## Recommendation

Use a combination of variables that doesn't sum to 64 bytes