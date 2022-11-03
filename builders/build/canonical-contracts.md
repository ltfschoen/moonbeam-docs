---
title: Canonical Contract Addresses on Moonbeam
description: Overview of the canonical contracts available on Moonbeam, Moonriver, & Moonbase Alpha, including common-goods contracts and precompiles.
keywords: canonical, ethereum, moonbeam, precompiled, contracts
---

# Canonical Contracts

![Canonical contracts banner](/images/builders/build/canonical-contracts/canonical-contracts-banner.png)

## Common-goods Contracts {: #common-goods-contracts }

The following contracts addresses have been established:

=== "Moonbeam"
    |                                                        Contract                                                         |                  Address                   |
    |:-----------------------------------------------------------------------------------------------------------------------:|:------------------------------------------:|
    |      [WGLMR](https://moonbeam.moonscan.io/address/0xAcc15dC74880C9944775448304B263D191c6077F#code){target=_blank}       | 0xAcc15dC74880C9944775448304B263D191c6077F |
    |    [Multicall](https://moonbeam.moonscan.io/address/0x83e3b61886770de2F64AAcaD2724ED4f08F7f36B#code){target=_blank}     | 0x83e3b61886770de2F64AAcaD2724ED4f08F7f36B |
    |    [Multicall2](https://moonbeam.moonscan.io/address/0x6477204E12A7236b9619385ea453F370aD897bb2#code){target=_blank}    | 0x6477204E12A7236b9619385ea453F370aD897bb2 |
    |    [Multicall3](https://moonbeam.moonscan.io/address/0xca11bde05977b3631167028862be2a173976ca11#code){target=_blank}    | 0xcA11bde05977b3631167028862bE2a173976CA11 |
    | [Multisig Factory](https://moonbeam.moonscan.io/address/0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2#code){target=_blank} | 0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2 |
    |                           [EIP-1820](https://eips.ethereum.org/EIPS/eip-1820){target=_blank}                            | 0x1820a4b7618bde71dce8cdc73aab6c95905fad24 |

=== "Moonriver"
    |                                                         Contract                                                         |                  Address                   |
    |:------------------------------------------------------------------------------------------------------------------------:|:------------------------------------------:|
    |      [WMOVR](https://moonriver.moonscan.io/address/0xf50225a84382c74CbdeA10b0c176f71fc3DE0C4d#code){target=_blank}       | 0xf50225a84382c74CbdeA10b0c176f71fc3DE0C4d |
    |    [Multicall](https://moonriver.moonscan.io/address/0x30f283Cc0284482e9c29dFB143bd483B5C19954b#code){target=_blank}*    | 0x30f283Cc0284482e9c29dFB143bd483B5C19954b |
    |    [Multicall2](https://moonriver.moonscan.io/address/0xaef00a0cf402d9dedd54092d9ca179be6f9e5ce3#code){target=_blank}    | 0xaef00a0cf402d9dedd54092d9ca179be6f9e5ce3 |
    |   [Multicall3](https://moonriver.moonscan.io/address/0xca11bde05977b3631167028862be2a173976ca11#code/){target=_blank}    | 0xcA11bde05977b3631167028862bE2a173976CA11 |
    | [Multisig Factory](https://moonriver.moonscan.io/address/0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2#code){target=_blank} | 0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2 |
    |                            [EIP-1820](https://eips.ethereum.org/EIPS/eip-1820){target=_blank}                            | 0x1820a4b7618bde71dce8cdc73aab6c95905fad24 |

    _*Deployed by SushiSwap_

=== "Moonbase Alpha"
    |                                                        Contract                                                         |                  Address                   |
    |:-----------------------------------------------------------------------------------------------------------------------:|:------------------------------------------:|
    |       [WDEV](https://moonbase.moonscan.io/address/0xD909178CC99d318e4D46e7E66a972955859670E1#code){target=_blank}       | 0xD909178CC99d318e4D46e7E66a972955859670E1 |
    |    [Multicall](https://moonbase.moonscan.io/address/0x4E2cfca20580747AdBA58cd677A998f8B261Fc21#code){target=_blank}*    | 0x4E2cfca20580747AdBA58cd677A998f8B261Fc21 |
    |    [Multicall2](https://moonbase.moonscan.io/address/0x37084d0158C68128d6Bc3E5db537Be996f7B6979#code){target=_blank}    | 0x37084d0158C68128d6Bc3E5db537Be996f7B6979 |
    |   [Multicall3](https://moonbase.moonscan.io/address/0xca11bde05977b3631167028862be2a173976ca11#code/){target=_blank}    | 0xcA11bde05977b3631167028862bE2a173976CA11 |
    | [Multisig Factory](https://moonbase.moonscan.io/address/0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2#code){target=_blank} | 0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2 |
    |                           [EIP-1820](https://eips.ethereum.org/EIPS/eip-1820){target=_blank}                            | 0x1820a4b7618bde71dce8cdc73aab6c95905fad24 |

    _*Deployed in the [UniswapV2 Demo Repo](https://github.com/PureStake/moonbeam-uniswap/tree/main/uniswap-contracts-moonbeam){target=_blank}_

## Precompiled Contracts {: #precompiled-contracts }

There are a set of precompiled contracts included on Moonbeam, Moonriver, and Moonbase Alpha that are categorized by address and based on the origin network. If you were to convert the precompiled addresses to decimal format, and break them into categories by numeric value, the categories are as follows:

- **0-1023** - [Ethereum MainNet precompiles](#ethereum-mainnet-precompiles)
- **1024-2047** - precompiles that are [not in Ethereum and not Moonbeam specific](#non-moonbeam-specific-nor-ethereum-precomiles)
- **2048-4095** - [Moonbeam specific precompiles](#moonbeam-specific-precompiles)

### Ethereum MainNet Precompiles {: #ethereum-mainnet-precompiles }

|                                                         Contract                                                         |                  Address                   |
|:------------------------------------------------------------------------------------------------------------------------:|:------------------------------------------:|
|   [ECRECOVER](/builders/pallets-precompiles/precompiles/eth-mainnet/#verify-signatures-with-ecrecover){target=_blank}    | 0x0000000000000000000000000000000000000001 |
|           [SHA256](/builders/pallets-precompiles/precompiles/eth-mainnet/#hashing-with-sha256){target=_blank}            | 0x0000000000000000000000000000000000000002 |
|        [RIPEMD160](/builders/pallets-precompiles/precompiles/eth-mainnet/#hashing-with-ripemd-160){target=_blank}        | 0x0000000000000000000000000000000000000003 |
|         [Identity](/builders/pallets-precompiles/precompiles/eth-mainnet/#the-identity-function){target=_blank}          | 0x0000000000000000000000000000000000000004 |
|  [Modular Exponentiation](/builders/pallets-precompiles/precompiles/eth-mainnet/#modular-exponentiation){target=_blank}  | 0x0000000000000000000000000000000000000005 |
|                [BN128Add](/builders/pallets-precompiles/precompiles/eth-mainnet/#bn128add){target=_blank}                | 0x0000000000000000000000000000000000000006 |
|                [BN128Mul](/builders/pallets-precompiles/precompiles/eth-mainnet/#bn128mul){target=_blank}                | 0x0000000000000000000000000000000000000007 |
|            [BN128Pairing](/builders/pallets-precompiles/precompiles/eth-mainnet/#bn128pairing){target=_blank}            | 0x0000000000000000000000000000000000000008 |
| [Blake2](https://paritytech.github.io/frontier/rustdocs/pallet_evm_precompile_blake2/struct.Blake2F.html){target=_blank} | 0x0000000000000000000000000000000000000009 |


### Non-Moonbeam Specific nor Ethereum Precompiles {: #non-moonbeam-specific-nor-ethereum-precompiles }

|                                                                    Contract                                                                     |                  Address                   |
|:-----------------------------------------------------------------------------------------------------------------------------------------------:|:------------------------------------------:|
|                  [SHA3FIPS256](/builders/pallets-precompiles/precompiles/eth-mainnet/#hashing-with-sha3fips256){target=_blank}                  | 0x0000000000000000000000000000000000000400 |
|          [Dispatch](https://paritytech.github.io/frontier/rustdocs/pallet_evm_precompile_dispatch/struct.Dispatch.html){target=_blank}          | 0x0000000000000000000000000000000000000401 |
| [ECRecoverPublicKey](https://paritytech.github.io/frontier/rustdocs/pallet_evm_precompile_simple/struct.ECRecoverPublicKey.html){target=_blank} | 0x0000000000000000000000000000000000000402 |

### Moonbeam-Specific Precompiles {: #moonbeam-specific-precompiles }

=== "Moonbeam"
    |                                                                  Contract                                                                  |                         Address                         |
    |:------------------------------------------------------------------------------------------------------------------------------------------:|:-------------------------------------------------------:|
    |  [Parachain Staking](https://github.com/PureStake/moonbeam/blob/master/precompiles/parachain-staking/StakingInterface.sol){target=_blank}  |        {{networks.moonbeam.precompiles.staking}}        |
    | [Crowdloan Rewards](https://github.com/PureStake/moonbeam/blob/master/precompiles/crowdloan-rewards/CrowdloanInterface.sol){target=_blank} |      {{networks.moonbeam.precompiles.crowdloan }}       |
    |         [ERC-20 Interface](https://github.com/PureStake/moonbeam/blob/master/precompiles/balances-erc20/ERC20.sol){target=_blank}          |        {{networks.moonbeam.precompiles.erc20 }}         |
    |     [Democracy](https://github.com/PureStake/moonbeam/blob/master/precompiles/pallet-democracy/DemocracyInterface.sol){target=_blank}      |       {{networks.moonbeam.precompiles.democracy}}       |
    |                [Xtokens](https://github.com/PureStake/moonbeam/blob/master/precompiles/xtokens/Xtokens.sol){target=_blank}                 |        {{networks.moonbeam.precompiles.xtokens}}        |
    |        [Relay Encoder](https://github.com/PureStake/moonbeam/blob/master/precompiles/relay-encoder/RelayEncoder.sol){target=_blank}        |     {{networks.moonbeam.precompiles.relay_encoder}}     |
    |  [XCM Transactor](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-transactor/src/v1/XcmTransactorV1.sol){target=_blank}  | {{networks.moonbeam.precompiles.xcm_transactor_legacy}} |
    |  [Author Mapping](https://github.com/PureStake/moonbeam/blob/master/precompiles/author-mapping/AuthorMappingInterface.sol){target=_blank}  |    {{networks.moonbeam.precompiles.author_mapping}}     |
    |                   [Batch](https://github.com/PureStake/moonbeam/blob/master/precompiles/batch/Batch.sol){target=_blank}                    |         {{networks.moonbeam.precompiles.batch}}         |
    |            [Randomness](https://github.com/PureStake/moonbeam/blob/master/precompiles/randomness/Randomness.sol){target=_blank}            |      {{networks.moonbeam.precompiles.randomness}}       |
    |           [Call Permit](https://github.com/PureStake/moonbeam/blob/master/precompiles/call-permit/CallPermit.sol){target=_blank}           |     {{networks.moonbeam.precompiles.call_permit }}      |

=== "Moonriver"
    |                                                                  Contract                                                                  |                         Address                          |
    |:------------------------------------------------------------------------------------------------------------------------------------------:|:--------------------------------------------------------:|
    |  [Parachain Staking](https://github.com/PureStake/moonbeam/blob/master/precompiles/parachain-staking/StakingInterface.sol){target=_blank}  |        {{networks.moonriver.precompiles.staking}}        |
    | [Crowdloan Rewards](https://github.com/PureStake/moonbeam/blob/master/precompiles/crowdloan-rewards/CrowdloanInterface.sol){target=_blank} |      {{networks.moonriver.precompiles.crowdloan }}       |
    |         [ERC-20 Interface](https://github.com/PureStake/moonbeam/blob/master/precompiles/balances-erc20/ERC20.sol){target=_blank}          |        {{networks.moonriver.precompiles.erc20 }}         |
    |     [Democracy](https://github.com/PureStake/moonbeam/blob/master/precompiles/pallet-democracy/DemocracyInterface.sol){target=_blank}      |       {{networks.moonriver.precompiles.democracy}}       |
    |                [Xtokens](https://github.com/PureStake/moonbeam/blob/master/precompiles/xtokens/Xtokens.sol){target=_blank}                 |        {{networks.moonriver.precompiles.xtokens}}        |
    |        [Relay Encoder](https://github.com/PureStake/moonbeam/blob/master/precompiles/relay-encoder/RelayEncoder.sol){target=_blank}        |     {{networks.moonriver.precompiles.relay_encoder}}     |
    |  [XCM Transactor](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-transactor/src/v1/XcmTransactorV1.sol){target=_blank}  | {{networks.moonriver.precompiles.xcm_transactor_legacy}} |
    |  [Author Mapping](https://github.com/PureStake/moonbeam/blob/master/precompiles/author-mapping/AuthorMappingInterface.sol){target=_blank}  |    {{networks.moonriver.precompiles.author_mapping}}     |
    |                   [Batch](https://github.com/PureStake/moonbeam/blob/master/precompiles/batch/Batch.sol){target=_blank}                    |         {{networks.moonriver.precompiles.batch}}         |
    |            [Randomness](https://github.com/PureStake/moonbeam/blob/master/precompiles/randomness/Randomness.sol){target=_blank}            |      {{networks.moonriver.precompiles.randomness}}       |
    |           [Call Permit](https://github.com/PureStake/moonbeam/blob/master/precompiles/call-permit/CallPermit.sol){target=_blank}           |     {{networks.moonriver.precompiles.call_permit }}      |

=== "Moonbase Alpha"
    |                                                                    Contract                                                                     |                         Address                         |
    |:-----------------------------------------------------------------------------------------------------------------------------------------------:|:-------------------------------------------------------:|
    |    [Parachain Staking](https://github.com/PureStake/moonbeam/blob/master/precompiles/parachain-staking/StakingInterface.sol){target=_blank}     |        {{networks.moonbase.precompiles.staking}}        |
    |   [Crowdloan Rewards](https://github.com/PureStake/moonbeam/blob/master/precompiles/crowdloan-rewards/CrowdloanInterface.sol){target=_blank}    |      {{networks.moonbase.precompiles.crowdloan }}       |
    |            [ERC-20 Interface](https://github.com/PureStake/moonbeam/blob/master/precompiles/balances-erc20/ERC20.sol){target=_blank}            |        {{networks.moonbase.precompiles.erc20 }}         |
    |        [Democracy](https://github.com/PureStake/moonbeam/blob/master/precompiles/pallet-democracy/DemocracyInterface.sol){target=_blank}        |       {{networks.moonbase.precompiles.democracy}}       |
    |                   [Xtokens](https://github.com/PureStake/moonbeam/blob/master/precompiles/xtokens/Xtokens.sol){target=_blank}                   |        {{networks.moonbase.precompiles.xtokens}}        |
    |          [Relay Encoder](https://github.com/PureStake/moonbeam/blob/master/precompiles/relay-encoder/RelayEncoder.sol){target=_blank}           |     {{networks.moonbase.precompiles.relay_encoder}}     |
    | [XCM Transactor Legacy](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-transactor/src/v1/XcmTransactorV1.sol){target=_blank} | {{networks.moonbase.precompiles.xcm_transactor_legacy}} |
    |    [Author Mapping](https://github.com/PureStake/moonbeam/blob/master/precompiles/author-mapping/AuthorMappingInterface.sol){target=_blank}     |    {{networks.moonbase.precompiles.author_mapping}}     |
    |                      [Batch](https://github.com/PureStake/moonbeam/blob/master/precompiles/batch/Batch.sol){target=_blank}                      |         {{networks.moonbase.precompiles.batch}}         |
    |              [Randomness](https://github.com/PureStake/moonbeam/blob/master/precompiles/randomness/Randomness.sol){target=_blank}               |      {{networks.moonbase.precompiles.randomness}}       |
    |             [Call Permit](https://github.com/PureStake/moonbeam/blob/master/precompiles/call-permit/CallPermit.sol){target=_blank}              |     {{networks.moonbase.precompiles.call_permit }}      |
    |                [XCM Utils](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-utils/XcmUtils.sol){target=_blank}                 |       {{networks.moonbase.precompiles.xcm_utils}}       |
    |    [XCM Transactor](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-transactor/src/v2/XcmTransactorV2.sol){target=_blank}     |    {{networks.moonbase.precompiles.xcm_transactor}}     |
