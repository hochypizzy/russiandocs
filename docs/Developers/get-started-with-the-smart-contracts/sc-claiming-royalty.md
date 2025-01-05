---
title: Claiming Royalty
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
Let's say you have a parent IP Asset that specifies in its [License Terms](doc:license-terms) that any derivatives must share 10% of their revenue with it. A child IPA then agrees to those, mints a [License Token](doc:license-token), and registers as a derivative.

In the below example, we will see how the parent IPA can claim its due 10% revenue from a child IPA that earns revenue.

## Prerequisites

* Understand the [💸 Royalty Module](doc:royalty-module)
* A child IPA and a parent IPA, for which their license terms have a commercial revenue share to make this example work

## Claim Royalty

Create a new file under `./test/5_Royalty.t.sol` and paste the following:

> 📘 Contract Addresses
>
> We have filled in the addresses from the Story contracts for you. However you can also find the addresses for them here: [Deployed Smart Contracts](doc:deployed-smart-contracts)

```sol 5_Royalty.t.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.26;

import { Test } from "forge-std/Test.sol";
// for testing purposes only
import { MockIPGraph } from "@storyprotocol/test/mocks/MockIPGraph.sol";
import { IPAssetRegistry } from "@storyprotocol/core/registries/IPAssetRegistry.sol";
import { LicenseRegistry } from "@storyprotocol/core/registries/LicenseRegistry.sol";
import { PILicenseTemplate } from "@storyprotocol/core/modules/licensing/PILicenseTemplate.sol";
import { RoyaltyPolicyLAP } from "@storyprotocol/core/modules/royalty/policies/LAP/RoyaltyPolicyLAP.sol";
import { PILFlavors } from "@storyprotocol/core/lib/PILFlavors.sol";
import { PILTerms } from "@storyprotocol/core/interfaces/modules/licensing/IPILicenseTemplate.sol";
import { ILicensingModule } from "@storyprotocol/core/interfaces/modules/licensing/ILicensingModule.sol";
import { LicenseToken } from "@storyprotocol/core/LicenseToken.sol";
import { IRoyaltyWorkflows } from "@storyprotocol/periphery/interfaces/workflows/IRoyaltyWorkflows.sol";
import { RoyaltyModule } from "@storyprotocol/core/modules/royalty/RoyaltyModule.sol";

import { SimpleNFT } from "../src/mocks/SimpleNFT.sol";
import { SUSD } from "../src/mocks/SUSD.sol";

// Run this test:
// forge test --fork-url https://odyssey.storyrpc.io/ --match-path test/5_Royalty.t.sol
contract RoyaltyTest is Test {
    address internal alice = address(0xa11ce);
    address internal bob = address(0xb0b);

    // For addresses, see https://docs.story.foundation/docs/deployed-smart-contracts
    // Protocol Core - IPAssetRegistry
    IPAssetRegistry internal IP_ASSET_REGISTRY = IPAssetRegistry(0x28E59E91C0467e89fd0f0438D47Ca839cDfEc095);
    // Protocol Core - LicenseRegistry
    LicenseRegistry internal LICENSE_REGISTRY = LicenseRegistry(0xBda3992c49E98392e75E78d82B934F3598bA495f);
    // Protocol Core - LicensingModule
    ILicensingModule internal LICENSING_MODULE = ILicensingModule(0x5a7D9Fa17DE09350F481A53B470D798c1c1aabae);
    // Protocol Core - PILicenseTemplate
    PILicenseTemplate internal PIL_TEMPLATE = PILicenseTemplate(0x58E2c909D557Cd23EF90D14f8fd21667A5Ae7a93);
    // Protocol Core - RoyaltyPolicyLAP
    RoyaltyPolicyLAP internal ROYALTY_POLICY_LAP = RoyaltyPolicyLAP(0x28b4F70ffE5ba7A26aEF979226f77Eb57fb9Fdb6);
    // Protocol Core - LicenseToken
    LicenseToken internal LICENSE_TOKEN = LicenseToken(0xB138aEd64814F2845554f9DBB116491a077eEB2D);
    // Protocol Periphery - RoyaltyModule
    RoyaltyModule internal ROYALTY_MODULE = RoyaltyModule(0xEa6eD700b11DfF703665CCAF55887ca56134Ae3B);
    // Protocol Periphery - RoyaltyWorkflows
    IRoyaltyWorkflows internal ROYALTY_WORKFLOWS = IRoyaltyWorkflows(0xAf922379B8e1abc6B0D78547128579221C7F7A22);
    // Mock - SUSD
    SUSD internal SUSD_TOKEN = SUSD(0xC0F6E387aC0B324Ec18EAcf22EE7271207dCE3d5);

    SimpleNFT public SIMPLE_NFT;
    uint256 public tokenId;
    address public ipId;
    uint256 public licenseTermsId;
    uint256 public startLicenseTokenId;
    address public childIpId;

    function setUp() public {
        // this is only for testing purposes
        // due to our IPGraph precompile not being
        // deployed on the fork
        vm.etch(address(0x0101), address(new MockIPGraph()).code);

        SIMPLE_NFT = new SimpleNFT("Simple IP NFT", "SIM");
        tokenId = SIMPLE_NFT.mint(alice);
        ipId = IP_ASSET_REGISTRY.register(block.chainid, address(SIMPLE_NFT), tokenId);

        licenseTermsId = PIL_TEMPLATE.registerLicenseTerms(
            PILFlavors.commercialRemix({
                mintingFee: 0,
                commercialRevShare: 10 * 10 ** 6, // 10%
                royaltyPolicy: address(ROYALTY_POLICY_LAP),
                currencyToken: address(SUSD_TOKEN)
            })
        );

        vm.prank(alice);
        LICENSING_MODULE.attachLicenseTerms(ipId, address(PIL_TEMPLATE), licenseTermsId);
        startLicenseTokenId = LICENSING_MODULE.mintLicenseTokens({
            licensorIpId: ipId,
            licenseTemplate: address(PIL_TEMPLATE),
            licenseTermsId: licenseTermsId,
            amount: 2,
            receiver: bob,
            royaltyContext: "" // for PIL, royaltyContext is empty string
        });

        // Registers a child IP (owned by Bob) as a derivative of Alice's IP.
        uint256 childTokenId = SIMPLE_NFT.mint(bob);
        childIpId = IP_ASSET_REGISTRY.register(block.chainid, address(SIMPLE_NFT), childTokenId);

        uint256[] memory licenseTokenIds = new uint256[](1);
        licenseTokenIds[0] = startLicenseTokenId;

        vm.prank(bob);
        LICENSING_MODULE.registerDerivativeWithLicenseTokens({
            childIpId: childIpId,
            licenseTokenIds: licenseTokenIds,
            royaltyContext: "" // empty for PIL
        });
    }

    /// @notice Pays SUSD to Bob's IP. Some of this SUSD is then claimable
    /// by Alice's IP.
    /// @dev In this case, this contract will act as the 3rd party paying SUSD
    /// to Bob (the child IP).
    function test_transferToVaultAndSnapshotAndClaimByTokenBatch() public {
        // ADMIN SETUP
        // We mint 100 SUSD to this contract so it has some money to pay.
        SUSD_TOKEN.mint(address(this), 100);
        // We approve the Royalty Module to spend SUSD on our behalf, which
        // it will do using `payRoyaltyOnBehalf`.
        SUSD_TOKEN.approve(address(ROYALTY_MODULE), 10);

        // This contract pays 10 SUSD to Bob's IP.
        ROYALTY_MODULE.payRoyaltyOnBehalf(childIpId, address(0), address(SUSD_TOKEN), 10);

        // Now that Bob's IP has been paid, Alice can claim her share (1 SUSD, which
        // is 10% as specified in the license terms)
        IRoyaltyWorkflows.RoyaltyClaimDetails[] memory claimDetails = new IRoyaltyWorkflows.RoyaltyClaimDetails[](1);
        claimDetails[0] = IRoyaltyWorkflows.RoyaltyClaimDetails({
            childIpId: childIpId,
            royaltyPolicy: address(ROYALTY_POLICY_LAP),
            currencyToken: address(SUSD_TOKEN),
            amount: 1
        });
        (uint256 snapshotId, uint256[] memory amountsClaimed) = ROYALTY_WORKFLOWS
            .transferToVaultAndSnapshotAndClaimByTokenBatch({
                ancestorIpId: ipId,
                claimer: ipId,
                royaltyClaimDetails: claimDetails
            });

        // Check that 1 SUSD was claimed by Alice's IP Account
        assertEq(amountsClaimed[0], 1);
        // Check that Alice's IP Account now has 1 SUSD in its balance.
        assertEq(SUSD_TOKEN.balanceOf(ipId), 1);
        // Check that Bob's IP now has 9 SUSD in its Royalty Vault, which it
        // can claim to its IP Account at a later point if he wants.
        assertEq(SUSD_TOKEN.balanceOf(ROYALTY_MODULE.ipRoyaltyVaults(childIpId)), 9);
    }
}
```

Run `forge build`. If everything is successful, the command should successfully compile.

To test this out, simply run the following command:

```shell
forge test --fork-url https://rpc.odyssey.storyrpc.io/ --match-path test/5_Royalty.t.sol
```
