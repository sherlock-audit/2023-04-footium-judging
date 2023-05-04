ctf_sec

medium

# Certain ERC20 token does not return bool from approve and transfer and transaction revert

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