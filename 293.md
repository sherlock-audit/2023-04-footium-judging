0x52

medium

# Users can bypass Player royalties on EIP2981 compatible markets by selling clubs as a whole

## Summary

Players have a royalty built in but clubs do not. This allows bulk sale of players via clubs to bypass the fee when selling players.

## Vulnerability Detail

[FootiumPlayer.sol#L16-L23](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPlayer.sol#L16-L23)

    contract FootiumPlayer is
        ERC721Upgradeable,
        AccessControlUpgradeable,
        ERC2981Upgradeable,
        PausableUpgradeable,
        ReentrancyGuardUpgradeable,
        OwnableUpgradeable
    {

FootiumPlayer implements the EIP2981 standard which creates fees when buy/selling the players.

[FootiumClub.sol#L15-L21](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumClub.sol#L15-L21)

    contract FootiumClub is
        ERC721Upgradeable,
        AccessControlUpgradeable,
        PausableUpgradeable,
        ReentrancyGuardUpgradeable,
        OwnableUpgradeable
    {

FootiumClub on the other hand never implements this standard. This allows users to sell players by selling their club to avoid any kind of fee on player sales.

## Impact

Users can bypass fees on player sales by selling club instead

## Code Snippet

[FootiumClub.sol#L15-L21](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumClub.sol#L15-L21)

## Tool used

Manual Review

## Recommendation

Implement EIP2981 on clubs as well