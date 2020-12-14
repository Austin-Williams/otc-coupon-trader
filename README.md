# Warning
This is unaudited and untested code. Please exercise caution and do not use it with more than you are comfortable losing.

Also, DO NOT send any tokens or ETH to this contract. Any tokens or ETH sent here will be stuck forever. Only interact with the contract via its functions.

## What

This is a smart contract intended to allow untrusted parties to buy/sell ESD coupons in an OTC fashion. An instance of this contract can be found on mainnet [here](https://etherscan.io/address/0xc4D6D61da34446Dc9Adc540Db748d3b9CAEaf140#code).

There are no order books or AMMs here. Any negotiations over price, quantity, epochs, etc takes place off-chain. This is just a way for two people who have already come to an agreement to execute that agreement without a trusted intermediary.

## How

### Step 1: Sellers approve this contract
The seller of coupons must approve the `CouponTrader` contract to move their coupons via the `approveCoupons` function on the [ESD DAO contract](https://etherscan.io/address/0x443D2f2755DB5942601fa062Cc248aAA153313D3#code). This only has to be done once.

### Step 2: Sellers create an offer
Once the seller has approved the `CouponTrader` contract, they can create an offer by calling the [`setOffer` function](https://github.com/Austin-Williams/otc-coupon-trader/blob/main/CouponTrader.sol#L73) on the `CouponTrader` contract. They can specify a specific `_buyer` that is allowed to take their offer, or they can set the buyer as address `0x0000000000000000000000000000000000000001` to indicate that the offer is open for anyone to take.

*Remember that USDC uses 6 decimal places, not 18 like most other coins*, so be careful when setting the `_totalUSDCRequiredFromSeller` parameter to use the correct number of "decimals".

A seller can update their off by calling the `setOffer` function again, and they can revoke their offer altogether by calling the [`revokeOffer` function](https://github.com/Austin-Williams/otc-coupon-trader/blob/main/CouponTrader.sol#L87).

### Step 3: Buyers approve this contract
The buyer of coupons must approve the `CouponTrader` contract to move their USDC via the `approve` function of the [USDC contract](https://etherscan.io/address/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48#writeProxyContract). This only has to be done once.

### Step 4: Buyer takes the offer
Once the buyer has approved the `CouponTrader` contract, they can take an offer by calling the [`takeOffer` function](https://github.com/Austin-Williams/otc-coupon-trader/blob/main/CouponTrader.sol#L99) and passing in the details of the trade. The details must match the seller's current offer or else the transaction will revert (this is for the buyer's safety, as it prevents them from getting frontrun by a malicious seller).

The `CouponTrader` contract will take 1% of the `_totalUSDCRequiredFromSeller` a fee -- so the buyer will have `_totalUSDCRequiredFromSeller` pulled from their account and the seller will receive 99% of that. The coupons will be transferred from the seller to the buyer.

At no time does this contract take possession of the coupons or the USDC.
