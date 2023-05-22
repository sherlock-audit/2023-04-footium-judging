# Issue H-1: Merkle leaf values for _clubDivsMerkleRoot are 64 bytes before hashing which can lead to merkle tree collisions 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/300 

## Found by 
0x52, 0xRan4212, GimelSec, cergyk, qpzm, shaka

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

# Issue H-2: Malicious users can honeypot other users by transferring out ERC20 and ERC721 tokens right before sale 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/291 

## Found by 
0x52, 0xAsen, 0xRobocop, BAHOZ, Dug, J4de, ast3ros, igingu, kiki\_dev, sashik\_eth, shogoki

## Summary

Since the club and escrow are separate and tokens can be transferred at any time by the owner, it allows malicious users to honeypot victims. 

## Vulnerability Detail

Tokens can be transferred out of the escrow by the owner of the club at anytime. This includes right before (or even in the same block) that the club is sold. This allows users to easily honeypot victims when selling clubs:

1) User A owns Club 1
2) Club 1 has players worth 5 ETH
3) User A lists Club 1 for 2.5 ETH
4) User B buys Club 1
5) User A sees the transaction in the mempool and quickly transfers all the players out
6) User A maintains all their players and User B now has an empty club

## Impact

Malicious users can honeypot other users

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L105-L111

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L120-L126

## Tool used

Manual Review

## Recommendation

Club/escrow system needs a redesign

# Issue H-3: Escrow approvals are not cleared when club is transferred allowing for abuse after transfer 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/289 

## Found by 
0x52, BenRai, Brenzee, CMierez, GalloDaSballo, J4de, MiloTruck, PokemonAuditSimulator, Quantish, cergyk, ctf\_sec, mstpr-brainbot, pengun, sashik\_eth, shaka, shogoki, toshii

## Summary

Escrow approvals remain even across club token transfers. This allows a malicious club owners to sell their club then drain everything after sale due to previous approvals.

## Vulnerability Detail

ERC20 and ERC721 token approval persist regardless of the owner of the club. The result is that approvals set by one owner can be accessed after a token has been sold or transferred. This allows the following attack:

1) User A owns clubId = 1
2) User A sets approval to themselves
3) User A sells clubId = 1 to User B
4) User A uses persistent approval to drain all players and tokens

## Impact

Malicious approvals can be used to drain club after sale

## Code Snippet

[FootiumEscrow.sol#L75-L81](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L75-L81)

[FootiumEscrow.sol#L90-L96](https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L90-L96)

## Tool used

Manual Review

## Recommendation

Club escrow system needs to be redesigned

# Issue M-1: Minting inconsistencies on FootiumPlayer and FootiumClub 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/342 

## Found by 
0xAsen, 0xHati, 0xLook, 0xPkhatri, 0xRobocop, 0xStalin, 0xeix, 0xhacksmithh, BAHOZ, Bauchibred, Bauer, Dug, GalloDaSballo, Koolex, PTolev, Phantasmagoria, TheNaubit, Tricko, ali\_shehab, cergyk, chaithanya\_gali, ctf\_sec, deadrxsezzz, descharre, indijanc, jasonxiale, kiki\_dev, lewisbroadhurst, nzm\_, oualidpro, sashik\_eth, shame, shogoki, tsueti\_, tsvetanovv, wzrdk3lly

## Summary

The `FootiumClub.sol` contract when minting uses `_mint()` instead of `_safeMint()` which can cause to mint a club to a contract who does not support nfts. On the other hand `FootiumPlayer.sol` uses `_safeMint()`.

## Vulnerability Detail

See summary.

## Impact

`FootiumClub.sol` might mint a club NFT to a contract that cannot handle nfts.

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumClub.sol#L65

## Tool used

Manual Review

## Recommendation

Use `_safeMint()` as in FootiumPlayer.

# Issue M-2: `changeMaxGenerationId` allows to mint tokens from older generations retroactively 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/319 

## Found by 
Dug, GalloDaSballo

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



## Discussion

**logiclogue**

This is an interesting one. A decision needs to be made whether this is the intended behaviour. Marking as 'Sponsor Confirmed'.

# Issue M-3: Users can bypass Player royalties on EIP2981 compatible markets by selling clubs as a whole 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/293 

## Found by 
0x52, 0xRobocop

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

# Issue M-4: Clubs can mint +1 players more than maxGenerationId 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/152 

## Found by 
Phantasmagoria, Sulpiride, alexzoid, descharre, juancito, mstpr-brainbot, ni8mare, santipu\_, shaka

## Summary
As stated in doc here
<img width="1007" alt="image" src="https://user-images.githubusercontent.com/120012681/236145880-57018bcf-d25d-4c4f-a3f0-b8bfcdc4f57a.png">
Initially clubs should be able to mint 10 players. However, clubs can always mint +1 players more than `maxGenerationId` 
## Vulnerability Detail
An off-by-one error is existed in academy contract when checking the generation ID while minting players. The code currently allows clubs to mint up to `maxGenerationId + 1` players, which is inconsistent with the documentation. If clubs send generation IDs in the following sequence: 0-1-2-3-4-5-6-7-8-9-10, all of these numbers will be valid, and the club will be able to mint `maxGenerationId + 1 ` players instead of the intended `maxGenerationId`

## Impact
Since this is not intended behaviour according to the protocol docs, I'll label it as high.
## Code Snippet
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L186-L188
here code checks whether the generation id is higher than maxGeneration or not, and it can be equal to maxGeneration.
## Tool used

Manual Review

## Recommendation
Use `generationId >= _maxGenerationId` or do not count the "0" index



## Discussion

**logiclogue**

Marked as 'Will Fix'. I believe the solution here is instead to update the documentation to say "At game launch this value will be 9". Because the documentation as far as I'm aware is correct in that it says `maxGenerationId` is the maximum generation ID, it's just that it does not represent the total players per cohort, which was the assumption with the default value being 10.

# Issue M-5: Users might lose funds as `claimERC20Prize()` doesn't revert for no-revert-on-transfer tokens 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/86 

## Found by 
0xAsen, 0xGoodess, 0xGusMcCrae, 0xPkhatri, 0xRobocop, 0xStalin, 0xeix, 0xmuxyz, 0xnirlin, 8olidity, ACai7, AlexCzm, BAHOZ, Bauchibred, Bauer, Cryptor, DevABDee, Diana, GalloDaSballo, GimelSec, J4de, Koolex, MiloTruck, PRAISE, PTolev, Phantasmagoria, Piyushshukla, PokemonAuditSimulator, Polaris\_tow, Proxy, Quantish, R-Nemes, SanketKogekar, Sulpiride, TheNaubit, Tricko, \_\_141345\_\_, abiih, alliums8520, ast3ros, berlin-101, cergyk, ctf\_sec, cuthalion0x, dacian, ddimitrov22, deadrxsezzz, djxploit, favelanky, georgits, holyhansss, innertia, jasonxiale, josephdara, jprod15, juancito, kiki\_dev, l3r0ux, lewisbroadhurst, nzm\_, oot2k, oualidpro, peanuts, ravikiran.web3, sach1r0, santipu\_, sashik\_eth, shaka, shame, thekmj, tibthecat, tsvetanovv, whoismatthewmc1, wzrdk3lly, yy

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

# Issue M-6: Certain ERC20 token does not return bool from approve and transfer and transaction revert 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/14 

## Found by 
0xGoodess, Dug, TheNaubit, ctf\_sec, deadrxsezzz, mstpr-brainbot, n1punp

## Summary

Certain ERC20 token does not return bool from approve and transfer and transaction revert

## Vulnerability Detail

According to

https://github.com/d-xo/weird-erc20#missing-return-values

Some tokens do not return a bool on ERC20 methods and use IERC20 token interface will revert transaction

Certain ERC20 token does not return bool from approve and transfer and transaction revert

```solidity
   function setApprovalForERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.approve(to, amount);
    }
```

and

```solidity
function transferERC20(
	IERC20 erc20Contract,
	address to,
	uint256 amount
) external onlyClubOwner {
	erc20Contract.transfer(to, amount);
}
```

the transfer / approve can fail slienlty

## Impact

Some tokens do not return a bool on ERC20 methods and use IERC20 token interface will revert transaction

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumEscrow.sol#L80

https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumEscrow.sol#L95

## Tool used

Manual Review

## Recommendation

Use Openzeppelin SafeTransfer / SafeApprove

# Issue M-7: Spender can frontrun setApprovalForERC20() in  FootiumEscrow.sol 

Source: https://github.com/sherlock-audit/2023-04-footium-judging/issues/8 

## Found by 
0xAsen, 0xRobocop, 0xStalin, Diana, Dug, PRAISE, PokemonAuditSimulator, ast3ros, berlin-101, ddimitrov22, deadrxsezzz, favelanky, innertia, kiki\_dev, nobody2018, pontifex, santipu\_, shaka, shame, wzrdk3lly

## Summary
The approve function is frontrunnable.

## Vulnerability Detail

Lets say Alice is the owner of these tokens and Bob is the spender:
- Now Alice approves Bob to spend his tokens by passing in Bob's address
- she approves him to transfer 1000 tokens.
- After sometime she decide to change from 1000 tokens to 500 tokens 
- Bob notices the Alice's second transaction before it was mined and quickly sends another transaction that calls the .transferFrom method to transfer 1000 tokens from Alice somewhere
- If the Bob's transaction will be executed before the Alice's transaction, then Bob will successfully transfer 1000 tokens from Alice and will gain an ability to transfer another 500 tokens too
- Before Alice noticed that something went wrong, Bob calls the .transferFrom method again, this time to transfer 500 tokens from  Alice.
- So, Alice's attempt to change the Bob's allowance from 1000 to 500 made it possible for Bob to transfer 1000+500 of Alice's tokens, while Alice never wanted to allow so many of her tokens to be transferred by Bob.

## Impact
It can result in losing tokens when club owner approves ERC20 tokens to any malicious Spender.
## Code Snippet
```solidity
    function setApprovalForERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.approve(to, amount);
    }
```

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L75-L81
## Tool used

Manual Review

## Recommendation
Use increaseAllowance and decreaseAllowance instead of approve as OpenZeppelin ERC20 implementation. Please see details here:

https://forum.openzeppelin.com/t/explain-the-practical-use-of-increaseallowance-and-decreaseallowance-functions-on-erc20/15103/4



## Discussion

**logiclogue**

This is a duplicate for another issue in this list. However, this one has a better suggestion

