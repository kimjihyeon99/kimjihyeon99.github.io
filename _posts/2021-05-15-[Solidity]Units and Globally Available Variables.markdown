---
layout: post
title: "[Solidity]Units and Globally Available Variables"
date: "2021-05-15 12:00:00 +0200" 
image: 6.jpg
tags: [solidity, units, globally ]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/units-and-global-variables.html

## Units and Globally Available Variables

### Ether Units

`Ether`의 하위 명칭을 지정하기 위해 literal number의 접미사로 `wei`, `gwei`, `ether` 를 사용할 수 있다.

접미사가 없는 경우 셋 중 `wei`로 지정한다.

````solidity
assert(1 wei == 1);
assert(1 gwei == 1e9);
assert(1 ether == 1e18);
````

- 0.9:gwei (가스, 네트워크 거래 수수료)
- 1:wei
- 10^12:szabo(0.7.0 버전에서 사라짐)
- 10^15:finney(0.7.0 버전에서 사라짐)
- 10^18:ether

### Time Units

`seconds`, `minutes`, `hours`, `days`, `weeks`

*`years`는 0.5.0버전에서 사라짐

### Special Variables and Functions

global namespaces에 항상 존재, 블록체인에 대한 정보를 제공하는데 주로 이용되거나, general-use utility 함수가 있다.

#### Block and Transaction Properties

|Block|Transaction Properties|
|---|---|
|blockhash(uint blockNumber) returns (bytes32)|지정된 블록의 해시를 제공(최신 블록 256개에서만 작동)|
|block.chainid(uint)|현재 chain id|
|block.coinbase(address payable)|현재 block miner's address|
|block.chainid(uint)|현재 chain id|
|block.difficulty|현재 block difficulty|
|block.gaslimit|현재 block gaslimit|
|block.number|현재 block number|
|block.timestamp(uint)|현재 block timestamp(unix epoch 이후 초)|
|gasleft() returns (uint256)|남아있는 gas|
|msg.data(bytes calldata)|전체 calldata|
|msg.sender(address)|message의 sender(current call)|
|msg.value(uint)|message와 함께 보낸 wei의 수|
|tx.gasprice(uint)|transaction의 gas 가격|
|tx.origin(address)|transaction의 sender(full call chain)|

#### ABI Encoding and Decoding Functions

|ABI Encoding|Decoding Functions|
|---|---|
|blockhash(uint blockNumber) returns (bytes32)|지정된 블록의 해시를 제공(최신 블록 256개에서만 작동)|
|block.chainid(uint)|현재 chain id|
|block.coinbase(address payable)|현재 block miner's address|
|block.chainid(uint)|현재 chain id|
|block.difficulty|현재 block difficulty|
|block.gaslimit|현재 block gaslimit|
|block.number|현재 block number|
|block.timestamp(uint)|현재 block timestamp(unix epoch 이후 초)|
|gasleft() returns (uint256)|남아있는 gas|
|msg.data(bytes calldata)|전체 calldata|
|msg.sender(address)|message의 sender(current call)|
|msg.value(uint)|message와 함께 보낸 wei의 수|
|tx.gasprice(uint)|transaction의 gas 가격|
|tx.origin(address)|transaction의 sender(full call chain)|

#### Error handling

#### Mathematical and Cryptographic Functions

#### Members of Address Types

#### Contract Related

#### Type Information

