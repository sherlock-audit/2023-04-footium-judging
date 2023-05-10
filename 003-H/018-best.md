ctf_sec

high

# Ineffective accounting result in lock fund or let user claim sub-optimal amount fund when user claimETHPrice and claimERC20Price

## Summary

Ineffective accounting result in lock fund when user claimETHPrice and claimERC20Price

## Vulnerability Detail

This is the accounting when user claimERC20Price or ClaimETHPrice in FootiumPriceDistributor.sol

```solidity
function claimERC20Prize(
	address _to,
	IERC20Upgradeable _token,
	uint256 _amount,
	bytes32[] calldata _proof
) external whenNotPaused nonReentrant {
	if (_to != msg.sender) {
		revert InvalidAccount();
	}

	if (
		!MerkleProofUpgradeable.verify(
			_proof,
			erc20MerkleRoot,
			keccak256(abi.encodePacked(_token, _to, _amount))
		)
	) {
		revert InvalidERC20MerkleProof();
	}

	uint256 value = _amount - totalERC20Claimed[_token][_to];

	if (value > 0) {
		totalERC20Claimed[_token][_to] += value;
		_token.transfer(_to, value);
	}

	emit ClaimERC20(_token, _to, value);
}
```

the main accounting issue is in the code below:

```solidity
uint256 value = _amount - totalERC20Claimed[_token][_to];

if (value > 0) {
	totalERC20Claimed[_token][_to] += value;
	_token.transfer(_to, value);
}
```

suppose user A acquire two merkle prove,

the first merkle prove let user A claim 1000 USDC

the second merkle prove let user A claim 500 USDC

at first, user A claim no USDC, claim amount is 1000 USDC

so value = 1000 USDC - 0 = 1000 USDC, this is fine

and 

totalERC20Claimed[_token][_to] = 1000 USDC

and user A claim 1000 USDC

then later user A want to use use the merkle proof to claim another 500 USDC,

user A found his transaction revert when calling

```solidity
uint256 value = _amount - totalERC20Claimed[_token][_to];
```

beause 500 USDC - 1000 USDC triggers arithemetic underflow.

Same issue exists if 

the first merkle proof let user claim 1000 USDC

and second merkle proof let user claim 1000 USDC

user can cannot the second 1000 USDC because when running

```solidity
uint256 value = _amount - totalERC20Claimed[_token][_to];
```

value goes to 0

when

the first merkle proof let user claim 1000 USDC

and second merkle proof let user claim 2000 USDC

when doing the second claim, user can only claim 1000 USDC instead of 2000 USDC because of the accounting

```solidity
uint256 value = _amount - totalERC20Claimed[_token][_to];
```

2000 USDC - 1000 USDC = 1000 UDSC

same issue applies in claimETHPrize

## Impact

Certainly user should not claim the fund using the same prove multiple times, but in the current codebase

Ineffective accounting result in lock fund or let user claim sub-optimal amount fund when user claimETHPrice and claimERC20Price

## Code Snippet

https://github.com/sherlock-audit/2023-04-footium/blob/11736f3f7f7efa88cb99ee98b04b85a46621347c/footium-eth-shareable/contracts/FootiumPrizeDistributor.sol#L125-L173

## Tool used

Manual Review

## Recommendation

We recommend the protocol just mark the same merkle hash as used and not let user reuse merkle hash and treat each claim independently.
