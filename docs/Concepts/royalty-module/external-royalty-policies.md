---
title: External Royalty Policies
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
There can be many flavors and variations of royalty distribution rules as we observe in the real world. The same can be expected onchain. Whenever a use case requires unique and specific royalty rules, then those set of rules can be registered as an **External Royalty Policy**.

## 1. What is an External Royalty Policy?

It is a smart contract that inherits a specific interface called `IExternalRoyaltyPolicy`, which defines the view function below:

```sol IExternalRoyaltyPolicy
    /// @notice Returns the amount of royalty tokens required to link a child to a given IP asset
    /// @param ipId The ipId of the IP asset
    /// @param licensePercent The percentage of the license
    /// @return The amount of royalty tokens required to link a child to a given IP asset
    function getPolicyRtsRequiredToLink(address ipId, uint32 licensePercent) external view returns (uint32);
```

After developing your smart contract make sure it inherits the interface above and you can register your new External Royalty Policy by calling `registerExternalRoyaltyPolicy` function in RoyaltyModule.sol.

## 2. How does it work?

Let's follow an example of a new External Royalty Policy called "Policy X".

### External Royalty Policies are selected by users

An IPA owner decides the royalty policy he/she wants to allow the IP to be remixed with. There are multiple options of royalty rules that can be chosen such as LAP, LRP and other External Royalty Policies. Let's say the user decides to mint a license token with "Policy X". After that, IP2 remixes IP1 and IP3 remixes IP2 and we have the situation as the image below:

![](https://files.readme.io/412377155f8b31081aa77a2bf8dfe86ac40795ebc9b2f963471f8e4d6cacf559-image.png)

Every time there is a remix - the link between the parent and derivative has 2 data points associated:

1. The royalty policy address
   1. "Policy X" address in the example
2. The percentage of royalty tokens the parent demands from derivatives. This percentage can have different meanings depending on the royalty policy being used - ie. it can be a relative percentage, an absolute percentage, an adjusted percentage according to specific rules, etc.
   1. 10% between IP1 and IP2
   2. 50% between IP2 and IP3

### External Royalty Policies receive royalty tokens from their users' IPs

Following the example, when each remix is made and during the `onLinkToParents` function call in RoyaltyModule.sol the function

`getPolicyRtsRequiredToLink(address ipId, uint32 licensePercent) external view returns (uint32)`

is called on the "Policy X" address. It should return the % of derivative's royalty tokens that the royalty policy demands for the link to happen. That share of royalty tokens are sent to the "Policy X" contract. In the example case:


* "Policy X" receives 3% of RT2 token supply that it can then redistribute to its userbase. IP1 owner wanted 10%, however - let's assume for the sake of the example - that due to the specific use case of "Policy X" and its custom logic, the IP2 owner is granted a special status in the platform in which it has a 70% discount on the % share it has to give parent IPs due to having a very large distribution network to promote IPs.
* "Policy X" receives 50% of RT3 token supply that it can then redistribute to its userbase.
=======
* "Policy X" receives 3% of RT2 token supply that it can then redistributed to its userbase. IP1 owner wanted 10%, however - let's assume for the sake of the example - that due to the specific use case of "Policy X" and its custom logic, the IP2 owner is granted a special status in the platform in which it it has a 70% discount on the % share it has to give parent IPs due to having a very large distribution network to promote IPs. Therefore, instead of having to give 10% as the license percentage indicated it only gives 3%.
* "Policy X" receives 50% of RT3 token supply that it can then redistributed to its userbase.


![](https://files.readme.io/33efb951a9be1339e849eb025d183a0f8d4f949f634ee5dfe1f13dac52c79bb0-image.png)

### External Royalty Policies redistribute value back to their users according to custom rules

There are two ways in which an External Royalty Policy can redistribute value back to its users:

1. Send Royalty Tokens directly to its users
2. Keep the Royalty Tokens in the External Royalty Policy contract and have users claim Revenue Tokens through the said contract

Let's explore both in the context of "Policy X". Let's say that from the 50% of RT3 token supply "Policy X" received - 40% are kept in the "Policy X" contract and 10% are sent to an ancestor royalty vault (IP1).

![](https://files.readme.io/4a8c08eac060fdaad89116ee26a60939938298a563d526d8835e5064e3f02e28-image.png)Now let's imagine there is a 1M payment made to IP3 - an example of how the flow would be:

![](https://files.readme.io/f39ebfc34c5b276df0ac1c32f3e014969d2c152daa41f099be4a75a5dd219734-image.png)

From the 1M USDC inflow to IP3 Royalty Vault:

* 500k USDC are claimed by the IP Account 3 which had 50% of RT3 token supply
* 100k USDC are claimed by the IP1 Royalty Vault which has 10% of RT3 token supply via `claimByTokenBatchAsSelf`  function
* 400k USDC are claimed by "Policy X" which has 40 of RT3 token supply. This amount is further split by "Policy X" custom contract according to its specific rules - which define y% and z% - to its users.