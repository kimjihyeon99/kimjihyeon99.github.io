---
layout: post
title: "[Solidity]Types"
date: "2021-05-13 12:00:00 +0200" 
image: 7.jpg
tags: [solidity, types]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/types.html

## Types

`Solidity` 는 복합 유형을 형성하기 위해 결합될 수 있는 몇 가지 유형을 제공한다.


연산자를 포함하는 식에서 유형이 상호작용 할 수 있다.


`undefiend` 또는 `null`값의 개념은 Solidity에 존재하지 않지만 새로 선언된 변수는 항상 해당

유형에 따라 기본값을 사용한다. 

### Value Types

항상 값으로 전달 되기 때문에, `함수인수` 또는 `할당`으로 사용될 때 항상 복사됨

#### Booleans

|구문|리턴값|
|---|---|
|bool x|true/false|

#### Integers

|구문|설명|default|
|---|---|---|
|int|양수, 음수 저장 가능|int==int256|
|uint|양수만 저장가능|uint=uint256|

#### Fixed Point Numbers

|구문|설명|Keyword|설명|
|---|---|---|---|
|fixed|다양한 크기의 양수, 음수의 고정소수점|fixedMxN|M은 유형별 비트 수, N은 사용가능한 소수점|
|ufixed|다양한 크기의 양수의 고정소수점|ufixedMxN|M은 8의 배수이고(8~256),N은 0~80|

#### Address

|구문|설명|
|---|---|
|address|20byte 값(이더리움 주소의 크기)을 보유함|
|address payable|adress와 동일하지만, 추가 멤버들과 함께 전송함|

차이점은 `address payable`가 Ether로 보낼 수 있는 주소이고,  `address`는 Ether로 보낼 수 없다.

##### Member of Address

- `balance` 와 `transfer`

`balance`를 사용해 `address`의 잔액을 조회, `transfer`를 사용해 Ether를 `address payable`로 보낼 수 있다.

````solidity
address payable x = address(0x123);
address myAddress = address(this);
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
````

현재 계약 잔액이 충분하지 않거나 Ether transfer가 수신 계정에의해 거부되는 경우  `transfer`함수 실패

`transfer`함수 실패시 되돌아감

- `send` 

`send`는 `transfer`의 하위단계이다.

실행 실패하면, 현재 contract는 예외로 중지되지 않지만 `send`는 `false`로 반환된다.

- `call`, `delegatecall` and `staticcall`

ABI를 준수하지 않는 contrat와 인터페이스하거나 인코딩에 대한 직접적인 제어를 얻기위해 사용함

단일 바이트 메모리 매개 변수를 사용하고 성공 조건과 반환된 데이터를 반환한다.

`abi.encode`, `abi.encodePacked`, `abi.encodeWithSelector` and `abi.encodeWithSignature` 함수가 구조화된 데이터를 encode하는데 사용될 수 있다.

````solidity
bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
(bool success, bytes memory returnData) = address(nameReg).call(payload);
require(success);
````

#### Contract Types



### Reference Types

### mapping Types

### Operators Involving LValues

### Conversions between Elementary Types


