Cryptor

high

# Changing the merkle root can result in a loss of funds for users

## Summary

The function claimETHPrize and the function claimERC20Prize allows a user to claim ETH or ERC20 tokens based on the merkle root set by the owner. However, there exists the functions setETHMerkleRoot and setERC20MerkleRoot. 

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L91-L95


which allows the owner to change the merkle root. If this is done before all of the prizes have been claimed by users, then prizes could be lost. 

## Vulnerability Detail

## Impact

Prizes earned by users could be lost if the owner calls setMerkleRoot before all prizes in the contract are claimed 

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L79-L83

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L91-L95

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L144-L148

https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L106-L111


## Tool used

Manual Review

## Recommendation

One possible mitigation would be to only allow the owner to change the merkle root when all of the funds have been claimed. Or perhaps add a time restriction so that if their prize is not claimed within a certain time period then it will be lost.
