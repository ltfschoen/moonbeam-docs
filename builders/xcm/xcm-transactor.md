---
title: Remote Execution Through XCM
description: How to do remote XCM calls in other chains using the XCM-Transactor pallet. The XCM-Transactor precompile enables access to core functions via the Ethereum API.
---

# Using the XCM-Transactor Pallet for Remote Executions

![XCM-Transactor Precompile Contracts Banner](/images/builders/xcm/xcm-transactor/xcmtransactor-banner.png)

## Introduction {: #introduction}

XCM messages are comprised of a [series of instructions](/builders/xcm/overview/#xcm-instructions){target=_blank} that are executed by the Cross-Consensus Virtual Machine (XCVM). Combinations of these instructions result in predetermined actions such as cross-chain token transfers and, more interestingly, remote cross-chain execution.

Nevertheless, building an XCM message from scratch is somewhat tricky. Moreover, XCM messages are sent to other participants in the ecosystem from the root account (that is, SUDO or through a democratic vote), which is not ideal for projects that want to leverage remote cross-chain calls via a simple transaction. 

To overcome these issues, developers can leverage wrapper functions/pallets to use XCM features on Polkadot/Kusama, such as the [XCM-transactor pallet](https://github.com/PureStake/moonbeam/blob/master/pallets/xcm-transactor/src/lib.rs){target=_blank}. In that aspect, the XCM-transactor pallet allows users to perform remote cross-chain calls through different methods that dispatch the call from different derivated accounts so that they can be easily executed with a simple transaction.

The two main extrinsics of the pallet are transacting through a sovereign derivative account or a derivative account calculated from a given multilocation. Each extrinsic is named accordingly.

The relevant [XCM instructions](/builders/xcm/overview/#xcm-instructions) to perform remote execution through XCM are, but not limited to:

 - [`WithdrawAsset`](https://github.com/paritytech/xcm-format#withdrawasset){target=_blank} - gets executed in the target chain. Removes assets and places them into the holding register
 - [`BuyExecution`](https://github.com/paritytech/xcm-format#buyexecution){target=_blank} - gets executed in the target chain. Takes the assets from holding to pay for execution fees. The fees to pay are determined by the target chain
 - [`Transact`](https://github.com/paritytech/xcm-format#transact){target=_blank} - gets executed in the target chain. Dispatches the encoded call data from a given origin

When the XCM message built by the XCM-transactor pallet is executed, fees must be paid. All the relevant information can be found in the [XCM-transactor fees section](/builders/xcm/fees/#xcm-transactor-fees){target=_blank} of the [XCM fees](/builders/xcm/fees/){target=_blank} page.

This guide will show you how to use the XCM-transactor pallet to send XCM messages from a Moonbeam-based network to other chains in the ecosystem (relay chain/parachains). In addition, you'll also learn how to use the XCM-transactor precompile to perform the same actions via the Ethereum API. 

**Note that there are still limitations in what you can remotely execute through XCM messages**.

**Developers must understand that sending incorrect XCM messages can result in the loss of funds.** Consequently, it is essential to test XCM features on a TestNet before moving to a production environment.

## Relevant XCM Definitions {: #general-xcm-definitions }

--8<-- 'text/xcm/general-xcm-definitions2.md'

 - **Derivative accounts** — an account derivated from another account. Derivative accounts are keyless (the private key is unknown). Consequently, derivative accounts related to XCM-specific use cases can only be accessed through XCM extrinsics. For such applications, there are two types:
     - **Sovereign-derivative account** — this produces a keyless account derivated from the parachain sovereign account in the destination chain. The derivation method uses the `utility.asDerivative` extrinsic for the remote call. When transacting through this derivative account, transaction fees are paid by the origin account (sovereign account in this case), but the transaction is dispatched from the derivative account. For more information, please refer to the [Derivative Accounts](/builders/pallets-precompiles/pallets/utility/){target=_blank} section of the utility pallet page
     - **Multilocation-derivative account** — this produces a keyless account derivated from the new origin set by the [Descend Origin](https://github.com/paritytech/xcm-format#descendorigin){target=_blank} XCM instruction, and the provided mulitlocation. For Moonbeam-based networks, [the derivation method](https://github.com/PureStake/moonbeam/blob/master/primitives/xcm/src/location_conversion.rs#L31-L37){target=_blank} is calculating the `blake2` hash of the multilocation, which includes the origin parachain ID, and truncating the hash to the correct length (20 bytes for Ethereum-styled account). The XCM call [origin conversion](https://github.com/paritytech/polkadot/blob/master/xcm/xcm-executor/src/lib.rs#L343){target=_blank} happens when the `Transact` instruction gets executed. Consequently, each parachain can convert the origin with its own desired procedure, so the user who initiated the transaction might have a different derivative account per parachain. This derivative account pays for transaction fees, and it is set as the dispatcher of the call
 - **Transact information** — relates to extra weight and fee information for the XCM remote execution part of the XCM-transactor extrinsic. This is needed because the XCM transaction fee is paid by the sovereign account. Therefore, XCM-transactor calculates what this fee is and charges the sender of the XCM-transactor extrinsic the estimated amount in the corresponding [XC-20 token](/builders/xcm/xc20/overview/){target=_blank}, to repay the sovereign account

## XCM-Transactor Pallet Interface {: #xcm-transactor-pallet-interface}

### Extrinsics {: #extrinsics }

The XCM-transactor pallet provides the following extrinsics (functions):

 - **deregister**(index) — deregisters a derivative account for a given index. This prevents the previously registered account from using a derivative address for remote execution. This extrinsic is only callable by *root*, for example, through a democracy proposal
 - **register**(address, index) — registers a given address as a derivative account at a given index. This extrinsic is only callable by *root*, for example, through a democracy proposal
 - **removeFeePerSecond**(assetLocation) — remove the fee per second information for a given asset in its reserve chain. The asset is defined as a multilocation
 - **removeTransactInfo**(location) — remove the transact information for a given chain, defined as a multilocation
 - **setFeePerSecond**(assetLocation, feePerSecond) — sets the fee per second information for a given asset on its reserve chain. The asset is defined as a multilocation. The `feePerSecond` is the token units per second of XCM execution that will be charged to the sender of the XCM-transactor extrinsic
 - **setTransactInfo**(location, transactExtraWeight, maxWeight) — sets the transact information for a given chain, defined as a multilocation. The transact information includes:
     - **transactExtraWeight** — weight to cover XCM instructions executions fees (`WithdrawAsset`, `BuyExecution`, and `Transact`), which is estimated to be at least 10% over what the remote XCM instructions execution uses 
     - **maxWeight** — maximum weight units allowed for the remote XCM execution
     - **transactExtraWeightSigned** — (optional) weight to cover XCM instructions executions fees (`DescendOrigin`, `WithdrawAsset`, `BuyExecution`, and `Transact`), which is estimated to be at least 10% over what the remote XCM instructions execution uses 
 - **transactThroughDerivative**(destination, index, fee, innerCall, weightInfo) — sends an XCM message with instructions to remotely execute a given call in the given destination (wrapped with the `asDerivative` option). The remote call will be signed by the origin parachain sovereign account (who pays the fees), but the transaction is dispatched from the sovereign-derivative account for the given index. The XCM-transactor pallet calculates the fees for the remote execution and charges the sender of the extrinsic the estimated amount in the corresponding [XC-20 token](/builders/xcm/xc20/overview/){target=_blank} given by the asset ID
 - **transactThroughSigned**(destination, fee, call, weightInfo) — sends an XCM message with instructions to remotely execute a given call in the given destination. The remote call will be signed and executed by a new account that the destination parachain must derivate. For Moonbeam-based network, this account is the `blake2` hash of the descended multilocation, truncated to the correct length. The XCM-transactor pallet calculates the fees for the remote execution and charges the sender of the extrinsic the estimated amount in the corresponding [XC-20 token](/builders/xcm/xc20/overview/){target=_blank} given by the asset ID
 - **transactThroughSovereign**(destination, feePayer, fee, call, originKind, weightInfo) — sends an XCM message with instructions to remotely execute a given call in the given destination. The remote call will be signed by the origin parachain sovereign account (who pays the fees), but the transaction is dispatched from a given origin. The XCM-transactor pallet calculates the fees for the remote execution and charges the given account the estimated amount in the corresponding [XC-20 token](/builders/xcm/xc20/overview/){target=_blank} given by the asset multilocation

Where the inputs that need to be provided can be defined as:

 - **index** — value to be used to calculate the derivative account. In the context of the XCM-transactor pallet, this is a derivative account of the parachain sovereign account in another chain
 - **assetLocation** — a multilocation representing an asset on its reserve chain. The value is used to set or retrieve the fee per second information
 - **location** — a multilocation representing a chain in the ecosystem. The value is used to set or retrieve the transact information
 - **destination** — a multilocation representing a chain in the ecosystem where the XCM message is being sent to
 - **fee** — an enum that provides developers two options on how to define the XCM execution fee item. Both options rely on the `feeAmount`, which is the units of the asset per second of XCM execution you provide to execute the XCM message you are sending. The two different ways to set the fee item are: 
     - **AsCurrencyID** — is the ID of the currency being used to pay for the remote call execution. Different runtimes have different ways of defining the IDs. In the case of Moonbeam-based networks, `SelfReserve` refers to the native token, and `ForeignAsset` refers to the asset ID of the XC-20 (not to be confused with the XC-20 address)
     - **AsMultiLocation** — is the multilocation that represents the asset to be used for fee payment when executing the XCM
 - **innerCall** — encoded call data of the call that will be executed in the destination chain. This is wrapped with the `asDerivative` option if transacting through the sovereign derivative account
 - **weightInfo** — a structure that contains all the weight related information. If not enough weight is provided, the execution of the XCM will fail, and funds might get locked in either the sovereign account or a special pallet. Consequently, **it is essential to correctly set the destination weight to avoid failed XCM executions**. The structure contains two fields:
     - **transactRequiredWeightAtMost** — weight related to the execution of the `Transact` call itself. For transacts through sovereign-derivative, you have to take into account the weight of the `asDerivative` extrinsic as well. However, this does not include the cost (in weight) of all the XCM instructions
     - **overallWeight** — the total weight the XCM-transactor extrinsic can use. This includes all the XCM instructions plus the weight of the call itself (**transactRequiredWeightAtMost**)
 - **call** — similar to `innerCall`, but it is not wrapped with the `asDerivative` extrinsic
 - **feePayer** — the address that will pay for the remote XCM execution in the transact through sovereign extrinsic. The fee is charged in the corresponding [XC-20 token](/builders/xcm/xc20/overview/){target=_blank}
 - **originKind** — dispatcher of the remote call in the destination chain. There are [four types of dispatchers](https://github.com/paritytech/polkadot/blob/0a34022e31c85001f871bb4067b7d5f5cab91207/xcm/src/v0/mod.rs#L60){target=_blank} available

### Storage Methods {: #storage-methods }

The XCM-transactor pallet includes the following read-only storage method:

 - **destinationAssetFeePerSecond**() - returns the fee per second for an asset given a multilocation. This enables conversion from weight to fee. The storage element is read by the pallet extrinsicts if `feeAmount` is set to `None`
 - **indexToAccount**(index) — returns the origin chain account associated with the given derivative index
 - **palletVersion**() — returns current pallet version from storage
 - **transactInfoWithWeightLimit**(location) — returns the transact information for a given multilocation. The storage element is read by the pallet extrinsicts if `feeAmount` is set to `None`

### Pallet Constants {: #constants }

The XCM-transactor pallet includes the following read-only functions to obtain pallet constants:

- **baseXcmWeight**() - returns the base XCM weight required for execution, per XCM instruction
- **selfLocation**() - returns the multilocation of the chain

## XCM-Transactor Transact Through Derivative {: #xcmtransactor-transact-through-derivative}

This section covers building an XCM message for remote executions using the XCM-transactor pallet using the `transactThroughDerivative` function.

!!! note
    You need to ensure that the call you are going to execute remotely is allowed in the destination chain!

### Checking Prerequisites {: #xcmtransactor-derivative-check-prerequisites}

To be able to send the extrinsics in Polkadot.js Apps, you need to have:

 - An [account](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/accounts){target=_blank} in the origin chain with [funds](/builders/get-started/networks/moonbase/#get-tokens){target=_blank}
 - The account from which you are going to send the XCM through the XCM-transactor pallet must also be registered in a given index to be able to operate through a derivative account of the sovereign account. The registration is done through the root account (SUDO in Moonbase Alpha), so [contact us](https://discord.gg/PfpUATX){target=_blank} to get it registered. For this example, Alice's account was registered at index `42`
 - Remote calls through XCM-transactor require the destination chain fee token to pay for that execution. Because the action is initiated in Moonbeam, you'll need to hold its [XC-20](/builders/xcm/xc20/){target=_blank} representation. For this example, because you are sending an XCM message to the relay chain, you need `xcUNIT` tokens to pay for the execution fees, which is the Moonbase Alpha representation of the Alphanet relay chain token `UNIT`. You can acquire some by swapping for DEV tokens (Moonbase Alpha's native token) on [Moonbeam-Swap](https://moonbeam-swap.netlify.app){target=_blank}, a demo Uniswap-V2 clone on Moonbase Alpha

![Moonbeam Swap xcUNITs](/images/builders/xcm/xc20/xtokens/xtokens-1.png)

To check your `xcUNIT` balance, you can add the XC-20 to MetaMask with the following address:

```
0xFfFFfFff1FcaCBd218EDc0EbA20Fc2308C778080
```

If you're interested in how the precompile address is calculated, you can check out the following guides:

- [Calculate External XC-20 Precompile Addresses](/builders/xcm/xc20/xc20/#calculate-xc20-address){target=_blank}
- [Calculate Mintable XC-20 Precompile Addresses](/builders/xcm/xc20/mintable-xc20/#calculate-xc20-address){target=_blank}


### Building the XCM {: #xcm-transact-through-derivative}

If you've [checked the prerequisites](#xcmtransactor-derivative-check-prerequisites), head to the extrinsic page of [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/extrinsics){target=_blank} and set the following options:

1. Select the account from which you want to send the XCM. Make sure the account complies with all the [prerequisites](#xcmtransactor-derivative-check-prerequisites)
2. Choose the **xcmTransactor** pallet
3. Choose the **transactThroughDerivative** extrinsic
4. Set the destination to **Relay**, to target the relay chain
5. Enter the index of the derivative account you've been registered to. For this example, the index value is `42`. Remember that the derivate account depends on the index
6. Select the **fee** type. For this example, set it to **AsCurrencyId**
7. Select **ForeignAsset** as the currency ID. This is because you are not transferring DEV tokens (*SelfReserve*), but interacting with an XC-20
8. Enter the asset ID. For this example, `xcUNIT` has an asset id of `42259045809535163221576417993425387648`. You can check all available assets IDs in the [XC-20 address section](/builders/xcm/xc20/overview/#current-xc20-assets){target=_blank}
9. (Optional) Set the **feeAmount**. This is the units per second, of the selected fee token (XC-20), that will be burned to free up the corresponding balance in the sovereign account on the destination chain. For this example, it was set to `13764626000000` units per second. If you do not provide this value, the pallet will use the element in storage (if exists)
10. Enter the inner call that will be executed in the destination chain. This is the encoded call data of the pallet, method, and input values to be called. It can be constructed in Polkadot.js Apps (must be connected to the destination chain), or using the [Polkadot.js API](/builders/build/substrate-api/polkadot-js-api/){target=_blank}. For this example, the inner call is `0x04000030fcfb53304c429689c8f94ead291272333e16d77a2560717f3a7a410be9b208070010a5d4e8`, which is a simple balance transfer of 1 `UNIT` to Alice's account in the relay chain. You can decode the call in [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ffrag-moonbase-relay-rpc-ws.g.moonbase.moonbeam.network#/extrinsics/decode){target=_blank}
11. Set the **transactRequiredWeightAtMost** weight value of the **weightInfo** struct. The value must include the `asDerivative` extrinsic as well. However, this does not include the weight of the XCM instructions. For this example, `1000000000` weight units are enough
12. (Optional) Set the **overallWeight** weight value of the **weightInfo** struct. The value must be the total of **transactRequiredWeightAtMost** plus the weight needed to cover the XCM instructions execution costs in the destination chain. If you do not provide this value, the pallet will use the element in storage (if exists), and add it to **transactRequiredWeightAtMost**. For this example, `2000000000` weight units are enough
13. Click the **Submit Transaction** button and sign the transaction

!!! note
    The encoded call data for the extrinsic configured above is 
    `0x2102002a0000018080778c30c20fa2ebc0ed18d2cbca1f0180a8a4d3840c00000000000000000000a404000030fcfb53304c429689c8f94ead291272333e16d77a2560717f3a7a410be9b208070010a5d4e800ca9a3b00000000010094357700000000`.

![XCM-Transactor Transact Through Derivative Extrinsic](/images/builders/xcm/xcm-transactor/xcmtransactor-1.png)

Once the transaction is processed, you can check the relevant extrinsics and events in [Moonbase Alpha](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/explorer/query/0xa90b23a54f2bb691ba2f04ae3228b1de2d2e7231b98490bf6f94e491baf09185){target=_blank} and the [relay chain](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ffrag-moonbase-relay-rpc-ws.g.moonbase.moonbeam.network#/explorer/query/0xb5a0ecc0c2f7f1363ede2e3aebab2702dd2e7b9036a6ba23a694db2b4002cd7f){target=_blank}. Note that, in Moonbase Alpha, there is an event associated with the `transactThroughDerivative` method, but also some `xcUNIT` tokens are burned to repay the sovereign account for the transactions fees. In the relay chain, the `paraInherent.enter` extrinsic shows a `balance.Transfer` event, where 1 `UNIT` token is transferred to Alice's address. Still, the transaction fees are paid by the Moonbase Alpha sovereign account.

!!! note
    The `AssetsTrapped` event on the relay chain is because the XCM-transactor pallet does not handle refunds yet. Therefore, overestimating weight will result in assets being trapped when the XCM is executed in the destination chain.

### Retrieve Registered Derivative Indexes

To fetch a list of all registered addresses allowed to operate through the Moonbeam-based network sovereign account and their corresponding indexes, head to the **Chain State** section of [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/chainstate){target=_blank} (under the **Developer** tab). In there, take the following steps:

1. From the **selected state query** dropdown, choose **xcmTransactor**
2. Select the **indexToAccount** method
3. (Optional) Disable/enable the include options slider. This will allow you to query the address authorized for a given index or request all addresses for all registered indexes
4. If you've enabled the slider, enter the index to query
5. Send the query by clicking on the **+** button

![Check Registered Derivative Indexes](/images/builders/xcm/xcm-transactor/xcmtransactor-2.png)


## XCM-Transactor Transact Through Signed {: #xcmtransactor-transact-through-signed}

This section covers building an XCM message for remote executions using the XCM-transactor pallet, specifically with the `transactThroughSigned` function. However, you'll not be able to follow along as the destination parachain is not publicly available.

!!! note
    You need to ensure that the call you are going to execute remotely is allowed in the destination chain!

### Checking Prerequisites {: #xcmtransactor-signed-check-prerequisites}

To be able to send the extrinsics in Polkadot.js Apps, you need to have:

 - An [account](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/accounts){target=_blank} in the origin chain with [funds](/builders/get-started/networks/moonbase/#get-tokens){target=_blank}
 - Funds in the multilocation-derivative account on the target chain. You can calculate this address by using the [`calculateMultilocationDerivative.ts` script](https://github.com/albertov19/xcmTools/blob/main/calculateMultilocationDerivative.ts){target=_blank}

For this example, the following accounts will be used:

 - Alice in the origin parachain with address `0x44236223aB4291b93EEd10E4B511B37a398DEE55`
 - Its multilocation-derivative address in the target parachain is `0x5c27c4bb7047083420eddff9cddac4a0a120b45c`

### Building the XCM {: #xcm-transact-through-derivative}

If you've [checked the prerequisites](#xcmtransactor-signed-check-prerequisites), head to the extrinsic page of [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/extrinsics){target=_blank} and set the following options:

1. Select the account from which you want to send the XCM. Make sure the account complies with all the [prerequisites](#xcmtransactor-signed-check-prerequisites)
2. Choose the **xcmTransactor** pallet
3. Choose the **transactThroughSigned** extrinsic
4. Define the destination multilocation. For this particular example, it is as follows:

    | Parameter |    Value    |
    |:---------:|:-----------:|
    |  Version  |     V1      |
    |  Parents  |      1      |
    | Interior  |     X1      |
    |    X1     |  Parachain  |
    | Parachain | ParachainID |

5. Select the **fee** type. For this example, set it to **AsCurrencyId** 
6. Set the currency ID to **ForeignAsset**. Note that this is the token that will be withdrawn from the multilocation-derivative account to pay for XCM execution fees. In this case, the native reserve token of the destination chain is being used, but it can be any other token as long as it is accepted as XCM execution fee token
7. Enter the asset ID. For this example, the XC-20 token has an asset id of `35487752324713722007834302681851459189`. You can check all available assets IDs in the [assets section](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fwss.api.moonbase.moonbeam.network#/assets){target=_blank} of Polkadot.js Apps
8. (Optional) Set the **feeAmount**. This is the units per second, of the selected fee token (XC-20), that will be withdrawn from the multilocation-derivative account to pay for XCM execution fees. For this example, it was set to `50000000000000000` units per second. If you do not provide this value, the pallet will use the element in storage (if exists). Note that the element in storage might not be up to date
9. Enter the inner call that will be executed in the destination chain. This is the encoded call data of the pallet, method, and input values to be called. It can be constructed in Polkadot.js Apps (must be connected to the destination chain), or using the [Polkadot.js API](/builders/build/substrate-api/polkadot-js-api/){target=_blank}. For this example, the inner call is `0x030044236223ab4291b93eed10e4b511b37a398dee5513000064a7b3b6e00d`, which is a simple balance transfer of 1 token of the destination chain to Alice's account there
10. Set the **transactRequiredWeightAtMost** weight value of the **weightInfo** struct. This does not include the weight of the XCM instructions. For this example, `1000000000` is enough
11. (Optional) Set the **overallWeight** weight value of the **weightInfo** struct. The value must be the total of **transactRequiredWeightAtMost** plus the weight needed to cover the XCM instructions execution costs in the destination chain. If you do not provide this value, the pallet will use the element in storage (if exists), and add it to **transactRequiredWeightAtMost**. For this example, `2000000000` weight units are enough
12. Click the **Submit Transaction** button and sign the transaction

!!! note
    The encoded call data for the extrinsic configured above is `0x210601010100e10d00017576e5e612ff054915d426c546b1b21a010000c52ebca2b10000000000000000007c030044236223ab4291b93eed10e4b511b37a398dee5513000064a7b3b6e00d00ca9a3b00000000010094357700000000`.

![XCM-Transactor Transact Through Signed Extrinsic](/images/builders/xcm/xcm-transactor/xcmtransactor-3.png)

Once the transaction is processed, Alice should've received one token in her address but on the destination chain.

## XCM-Transactor Precompile {: #xcmtransactor-precompile}

The XCM-transactor precompile contract allows developers to access the XCM-transactor pallet features through the Ethereum API of Moonbeam-based networks. Similar to other [precompile contracts](/builders/pallets-precompiles/precompiles/){target=_blank}, the XCM-transactor precompile is located at the following addresses:

=== "Moonbase Alpha"
     ```
     {{networks.moonbase.precompiles.xcm_transactor}}
     ```

The XCM-transactor legacy precompile is still available in all Moonbeam-based networks. However, **the legacy version will be depecrated in the near future**, so all implementations must migrate to the newer interface. The XCM-transactor legacy precompile is located at the following addresses:

=== "Moonbeam"
     ```
     {{networks.moonbeam.precompiles.xcm_transactor_legacy}}
     ```

=== "Moonriver"
     ```
     {{networks.moonriver.precompiles.xcm_transactor_legacy}}
     ```

=== "Moonbase Alpha"
     ```
     {{networks.moonbase.precompiles.xcm_transactor_legacy}}
     ```

### The XCM-Transactor Solidity Interface {: #xcmtrasactor-solidity-interface } 

[XcmTransactor.sol](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-transactor/src/v2/XcmTransactorV2.sol){target=_blank} is an interface through which developers can interact with the XCM-transactor pallet using the Ethereum API. 

!!! note
    The [legacy version](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-transactor/src/v1/XcmTransactorV1.sol){target=_blank} of the XCM-transactor precompile will be depecrated in the near future, so all implementations must migrate to the newer interface.

The interface includes the following functions:

 - **indexToAccount**(*uint16* index) — read-only function that returns the registered address authorized to operate using a derivative account of the Moonbeam-based network sovereign account for the given index
  - **transactInfoWithSigned**(*Multilocation* *memory* multilocation) — read-only function that, for a given chain defined as a multilocation, returns the transact information considering the three XCM instructions associated with the external call execution (`transactExtraWeight`). It also returns extra weight information associated with the `DescendOrigin` XCM instruction for the transact through signed extrinsic (`transactExtraWeightSigned`)
 - **feePerSecond**(*Multilocation* *memory* multilocation) — read-only function that, for a given asset as a multilocation, returns units of token per second of the XCM execution that is charged as the XCM execution fee. This is useful when, for a given chain, there are multiple assets that can be used for fee payment
 - **transactThroughDerivativeMultilocation**(*uint8* transactor, *uint16* index, *Multilocation* *memory* feeAsset, *uint64* transactRequiredWeightAtMost, *bytes* *memory* innerCall, *uint256* feeAmount, *uint64* overallWeight) — function that represents the `transactThroughDerivative` method described in the [previous example](#xcmtransactor-transact-through-derivative), setting the **fee** type to **AsMultiLocation**. You need to provide the asset multilocation of the token that is used for fee payment instead of the XC-20 token `address`
 - **transactThroughDerivative**(*uint8* transactor, *uint16* index, *address* currencyId, *uint64* transactRequiredWeightAtMost, *bytes* *memory* innerCall, *uint256* feeAmount, *uint64* overallWeight) — function that represents the `transactThroughDerivative` method described in the [previous example](#xcmtransactor-transact-through-derivative), setting the **fee** type to **AsCurrencyId**. Instead of the asset ID, you'll need to provide the [asset XC-20 address](/builders/xcm/xc20/overview/#current-xc20-assets){target=_blank} of the token that is used for fee payment 
 - **transactThroughSignedMultilocation**(*Multilocation* *memory* dest, *Multilocation* *memory* feeLocation, *uint64* transactRequiredWeightAtMost, *bytes* *memory* call, *uint256* feeAmount, *uint64* overallWeight) — function that represents the `transactThroughSigned` method described in the [previous example](#xcmtransactor-transact-through-signed), setting the **fee** type to **AsMultiLocation**. You need to provide the asset multilocation of the token that is used for fee payment instead of the XC-20 token `address`
 - **transactThroughSigned**(*Multilocation* *memory* dest, *address* feeLocationAddress, *uint64* transactRequiredWeightAtMost, *bytes* *memory* call, *uint256* feeAmount, *uint64* overallWeight) — function that represents the `transactThroughSigned` method described in the [previous example](#xcmtransactor-transact-through-signed), setting the **fee** type to **AsCurrencyId**.  Instead of the asset ID, you'll need to provide the [asset XC-20 address](/builders/xcm/xc20/overview/#current-xc20-assets){target=_blank} of the token that is used for fee payment 

### Building the Precompile Multilocation {: #building-the-precompile-multilocation }

In the XCM-transactor precompile interface, the `Multilocation` structure is defined as follows:

--8<-- 'text/xcm/xcm-precompile-multilocation.md'

The following code snippet goes through some examples of `Multilocation` structures, as they would need to be fed into the XCM-transactor precompile functions:


```js
// Multilocation targeting the relay chain asset from a parachain
{
    1, // parents = 1
    [] // interior = here
}

// Multilocation targeting Moonbase Alpha DEV token from another parachain
{
    1, // parents = 1
    // Size of array is 2, meaning is an X2 interior
    [
        "0x00000003E8", // Selector Parachain, ID = 1000 (Moonbase Alpha)
        "0x0403" // Pallet Instance = 3
    ]
}

// Multilocation targeting aUSD asset on Acala
{
    1, // parents = 1
    // Size of array is 1, meaning is an X1 interior
    [
        "0x00000007D0", // Selector Parachain, ID = 2000 (Acala)
        "0x060001" // General Key Selector + Asset Key
    ]
}
```

## XCM-Utils Precompile {: #xcmutils-precompile}

The XCM-utils precompile contract gives developers polkadot related utility functions directly within the EVM. This allows for easier transactions and interactions with other XCM related precompiles. Similar to other [precompile contracts](/builders/pallets-precompiles/precompiles/){target=_blank}, the XCM-utils precompile is located at the following addresses:

<<<<<<< HEAD
=== "Moonbeam"
     ```
     {{networks.moonbeam.precompiles.xcm_utils}}
     ```
=== "Moonriver"
     ```
     {{networks.moonriver.precompiles.xcm_utils}}
     ```
=======
>>>>>>> b4f48362 (Added xcm-utils to xcm-transactor)
=== "Moonbase Alpha"
     ```
     {{networks.moonbase.precompiles.xcm_utils}}
     ```

### The XCM-Utils Solidity Interface {: #xcmutils-solidity-interface } 
[XcmUtils.sol](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-utils/XcmUtils.sol){target=_blank} is an interface to interact with the precompile.

!!! note
    While there is currently only one function, the precompile will be updated in the future to include additional features. Feel free to suggest additional utility functions in the [Discord](https://discord.gg/PfpUATX){target=_blank}.

The interface includes the following functions:

 - **multilocationToAddress**(*Multilocation memory* multilocation) — read-only function that returns the EVM-compatible address derived from a given Multilocation. For example, the Multilocation `[0, ["0x0344236223aB4291b93EEd10E4B511B37a398DEE5500"]]` will return `0x44236223aB4291b93EEd10E4B511B37a398DEE5500`

The `Multilocation` structure in the XCM-utils precompile is built the [same as the XCM-transactor](#building-the-precompile-multilocation) precompile's `Multilocation`.
