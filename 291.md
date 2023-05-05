0x52

high

# Malicious users can honeypot other users by transferring out ERC20 and ERC721 tokens right before sale

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