PRAISE

high

# Spender can frontrun setApprovalForERC20() in  FootiumEscrow.sol

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