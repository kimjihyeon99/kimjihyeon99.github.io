---
layout: post
title: "[Solidity] Units and Globally Available Variables"
date: "2021-05-15 12:00:00 +0200" 
image: 5.jpg
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
|abi.decode(bytes memory encodedData, (...)) returns (...)|주어진 데이터를 ABI-decoding하고, types은 괄호 안에 두번째 인자로 표시됨|
|abi.encode(...) returns (bytes memory)|주어진 인자를 ABI로 인코딩|
|abi.encodePacked(...) returns (bytes memory)|주어진 인자의 `압축인코딩`을 수행|
|abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)|두번째부터 시작하여 주어진 인자를 ABI로 인코딩하고 주어진 4바이트 선택자를 앞에 추가함|
|abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)|==abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)|

`keccak256(abi.encodePacked(a, b))` : 구조화된 데이터의 해시를 계산하는 방법


##### Members of bytes

`bytes.concat(...) returns (bytes memory)` : 가변 바이트 수 와 bytes1,..,bytes32 인자를 하나의 바이트 배열에 연결함


#### Error handling

*오류처리 및 사용시기에 대한 자세한 내용은 'assert and require' 섹션

|Error handling|description|
|---|---|
|assert(bool condition)|condition을 평가한 후 boolean값을 반환, 반환 값에 따라 프로그램을 실행 or 예외 발생시킴/ contract 실행 전 현재 상태와 함수 조건을 check하는데 사용(내부 에러) |
|require(bool condition)|condition이 충족되지 않으면 revert, 입력이나 외부 컴포넌트 오류에 사용됨|
|require(bool condition, string memory message)|주어진 인자의 `압축인코딩`을 수행|
|revert()|어떠한 조건을 검사하지 않고, 실행중단 및 함수 호출 되돌림|
|revert(string memory reason)|revert + 설명 문자열 제공|


#### Members of Address Types

|Function|description|
|---|---|
|<address>.balance (uint256)|`address의 잔고 (wei)|
|<address>.code (bytes memory)|address에서 code|
|<address>.codehash (bytes32)|address의 codehash|
|<address payable>.transfer(uint256 amount)|address로 주어진 amount (wei)를 보냄, 실패에 대해 revert|
|<address payable>.send(uint256 amount) returns (bool)|address로 주어진 amount (wei)를 보냄, 실패에 대해 false 반환|
|<address>.call(bytes memory) returns (bool, bytes memory)|주어진 payload로 low-level call를 발행하고, success condition과 return data를 반환 |
|<address>.delegatecall(bytes memory) returns (bool, bytes memory)|주어진 payload로 low-level delegatecall 발행하고, success condition과 return data를 반환 |
|<address>.staticcall(bytes memory) returns (bool, bytes memory)|주어진 payload로 low-level staticcall 발행하고, success condition과 return data를 반환 |
aass

#### Contract Related

`this`(현재 contract's type)

: 현재 contract, explicit하게 address로 변환가능함

`selfdestruct(address payable recipient)`

: 현재 contract를 파기하고, 해당 자금을 지정된 address로 보내고 실행종료함
  
   EVM으로부터 상속된 특성
   - receiving contract's receive 함수는 실행되지 않음
   - contract는 transaction이 끝날때만 파기되고,  "undo"로 해당 파기를 `revert`할 수 있음
 
+현재 contract의 모든 function은 현재 function을 포함하여 직접 호출 가능
 
 
#### Type Information

`type(X)`는 type X에 대한 정보를 검색하는데 사용할 수 있다.

-Contract type C

|Property|description|
|---|---|
|type(C).name|contract 이름|
|type(C).creationCode|contract의 creation byteCode가 있는 메모리 byte array|
|type(C).runtimeCode|contract의 runtime byteCode가 있는 메모리 byte array|

-Contract type I

|Property|description|
|---|---|
|type(I).interfaceId|상속된 모든 함수를 제외하고 인터페이스 자체 내에서 정의된 모든 함수 선택기의 XOR로 정의|

-Contract type T

|Property|description|
|---|---|
|type(T).min|Type T로 나타낼 수 있는 가장 작은 값|
|type(T).max|Type T로 나타낼 수 있는 가장 큰 값|
