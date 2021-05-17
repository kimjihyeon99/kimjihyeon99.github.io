---
layout: post
title: "[Solidity] Types"
date: "2021-05-13 12:00:00 +0200" 
image: 7.jpg
tags: [solidity, types]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/types.html

## Types


`Solidity` 는 복합 유형을 형성하기 위해 결합될 수 있는 몇 가지 유형을 제공한다.


연산자를 포함하는 식에서 유형이 상호작용 할 수 있다.


`undefiend` 또는 `null`값의 개념은 Solidity에 존재하지 않지만 새로 선언된 변수는 

항상 해당 유형에 따라 기본값을 사용한다. 


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
|ufixed|다양한 크기의 양수의 고정소수점|ufixedMxN|M은 8의 배수이고(8-256), N은 0-80|


#### Address

|구문|설명|
|---|---|
|address|20byte 값(이더리움 주소의 크기)을 보유함|
|address payable|adress와 동일하지만, 추가 멤버들과 함께 전송함|

*차이점은 `address payable`가 Ether로 보낼 수 있는 주소이고,  `address`는 Ether로 보낼 수 없다.

##### Member of Address

- `balance` 와 `transfer`

`balance`를 사용해 `address`의 잔액을 조회, `transfer`를 사용해 Ether를 `address payable`로 보낼 수 있다.

````solidity
address payable x = address(0x123);
address myAddress = address(this);
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
````

현재 계약 잔액이 충분하지 않거나 Ether transfer가 수신 계정에의해 거부되는 경우  `transfer`함수 실패

`transfer`함수 실패시 `revert`

- `send` 

`send`는 `transfer`의 하위단계이다.

실행 실패하면, 현재 contract는 예외로 중지되지 않지만 `send`는 `false`로 반환된다.

- `call`, `delegatecall` and `staticcall`

*ABI : Application Binary Interface

ABI를 준수하지 않는 contrat와 인터페이스하거나 인코딩에 대한 직접적인 제어를 얻기위해 사용함

단일 바이트 메모리 매개 변수를 사용하고 성공 조건과 반환된 데이터를 반환한다.

`abi.encode`, `abi.encodePacked`, `abi.encodeWithSelector` and `abi.encodeWithSignature` 함수가 구조화된 데이터를 encode하는데 사용될 수 있다.

````solidity
bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
(bool success, bytes memory returnData) = address(nameReg).call(payload);
require(success);
````


#### Contract Types

모든 contract는 자신의 Type을 정의한다.

contract는 `address` 타입으로 명시적으로 변환 가능함

`address payable`으로 명시적 변환은 contract 타입이 receive 또는 payable fallback 함수가 있는 경우 가능하다.

-변환은 `address(x)`을 사용한다. 

-만약 receive 또는 payable fallback 함수가 없는 경우, `address payable`으로 변환은  `payable(address(x))`을 사용한다. 

*자세한 내용은 address type에관한 섹션에서 찾아볼 수 있다. 


#### Function Types

`function`타입의 변수는 함수에서 할당할 수 있고, 매개변수는 함수 호출에 함수를 전달하거나 반환하는데 사용할 수 있다.

기능유형

- 내부 : 현재 contract 내에서만 호출 가능, entry label로 점프함으로써 내부`function`가 구현됨
- 외부 : `address`와 `function signature`로 구성, 외부 function 호출을 통해 전달 및 반환 가능

  +외부 함수 member

  - `.address` : function의 contract의 address를 반환
  - `.selector` : ABI function selector를 반환 (추가내용은 뒤에)

````solidity
function (<parameter types>) {internal|external} [pure|view|payable] [returns (<return types>)]
````

`pure` :  `view` 및 `non-payable` 함수로 변환될 수 있음

`view` : `non- payable`함수로 변환될 수 있음

`payable`: `non- payable`함수로 변환될 수 있음

`payable`과 `non-payable` 차이

만약 function이 `payable`이라면, zero Ether 결제도 허용한다는 의미이므로, `non-payable`도 된다.

하지만  `non-payable`함수는 Ether가 보내지는 것을 거부하므로, `payable`함수로 변환할 수 없다.

````solidity
//internal
contract Pyramid {
    using ArrayUtils for *;

    function pyramid(uint l) public pure returns (uint) {
        return ArrayUtils.range(l).map(square).reduce(sum);
    }

    function square(uint x) internal pure returns (uint) {
        return x * x;
    }

    function sum(uint x, uint y) internal pure returns (uint) {
        return x + y;
    }
}

//external
contract Example {
    function f() public payable returns (bytes4) {
        assert(this.f.address == address(this));
        return this.f.selector;
    }

    function g() public {
        this.f{gas: 10, value: 800}();
    }
}
````


### Reference Types

`Reference Type`의 값은 여러 다른 이름을 통해 수정 가능하다.

구성 : struct, arrays, mappings

`Reference Type`을 사용하는 경우, 항상 유형이 저장되는 데이터 영역을 명시적으로 제공해야한다.:

`memory`(수명이 외부 함수호출로 제한), 

`storage`(state variable이 저장되는 위치, contract의 수명으로 제한) 

또는 `calldata`(function arg가 포함된 특수한 값 위치, 수정할 수 없음) 를 명시적으로 제공해야함.


#### Data location

모든 reference type에는 추가적인 annotation이 있다.

data location는 어디에 저장되어있는지에 대해 지정한다. 

종류 : `memory`, `storage`, `call data`


- 할당, 변환
 
데이터 위치를 변경하는 경우, 항상 자동 복사 작업이 수행되지만,

동일한 데이터 위치 내의 할당은 `storage` types에 대해서만 복사된다.


#### Arrays

배열은 compile-time에 고정된 크기를 가질 수 있고, 또는 dynamic 크기를 가질 수 있다. 

##### `bytes.concat` function

`bytes` 변수를 연결할 수 있다.

패딩 없이 인자(arg)의 내용을 포함하는 단일 바이트 memory 배열을 반환한다.

````solidity
pragma solidity ^0.8.4;

contract C {
    bytes s = "Storage";
    function f(bytes calldata c, string memory m, bytes16 b) public view {
        bytes memory a = bytes.concat(s, c, c[:2], "Literal", bytes(m), b);
        assert((s.length + c.length + 2 + 7 + bytes(m).length + 16) == a.length);
    }
}
````


##### Array Literals

선언 예시 

````solidity
pragma solidity >=0.4.16 <0.9.0;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
    function g(uint[3] memory) public pure {
        // ...
    }
}
````

정적 크기의 메모리 배열이고, 배열 타입 변환시 요소 중 하나는 바꾸려는 type이어야 한다

위의 예제에서 [1,2,3]의 type은 uint8[3] memory 이다. 결과 타입을 uint[3]으로 만들기 위해 첫번째 요소를 uint로 변환 하였다.


##### Array Members

`length` : elements의 수를 포함. memory array 의 `length`는 고정되어있다.

`push()` : dynamic array 및 바이트 일 경우, 초기화 element를 array 끝에 추가할 수 있다.

`push(x)` : dynamic array 및 바이트 일 경우, array 끝에 element를 추가할 수 있다.

`pop` : dynamic array 및 바이트 일 경우, array 끝에 element를 제거할 수 있다.


### mapping Types

선언 예시)

````solidity
contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}
````

선언 형식 :  mapping(_Keytype => _valueType) [public/private] _VarableName;

`_Keytype` : mapping, struct 또는 array와 같은 복잡한 유형 허용안함

`_valueType` : mapping, array 및 struct를 포함한 모든 유형 가능함


#### Iterable Mappings

mapping을 iterate 할 수 없다.

하지만, 데이터 구조를 mapping 위에 구현하고, 그 위에 iterate하는 것은 가능하다. 


예시코드)

````solidity
library IterableMapping {

    function iterate_start(itmap storage self) internal view returns (uint keyIndex) {
        return iterate_next(self, type(uint).max);
    }

    function iterate_valid(itmap storage self, uint keyIndex) internal view returns (bool) {
        return keyIndex < self.keys.length;
    }

    function iterate_next(itmap storage self, uint keyIndex) internal view returns (uint r_keyIndex) {
        keyIndex++;
        while (keyIndex < self.keys.length && self.keys[keyIndex].deleted)
            keyIndex++;
        return keyIndex;
    }

    function iterate_get(itmap storage self, uint keyIndex) internal view returns (uint key, uint value) {
        key = self.keys[keyIndex].key;
        value = self.data[key].value;
    }
}

// How to use it
contract User {
    itmap data; //mapping 선언
    // data type에 interate library 적용
    using IterableMapping for itmap;

    // 저장된 데이터의 합 구하기
    function sum() public view returns (uint s) {
        for (
            uint i = data.iterate_start();
            data.iterate_valid(i);
            i = data.iterate_next(i)
        ) {
            (, uint value) = data.iterate_get(i);
            s += value;
        }
    }
}
````


### Operators Involving LValues

만약 `a`가 Lvalue인경우 다음과 같은 연산자를 사용할 수 있다. 

`+=` `-=` `*=` `/=` `%=` `|=` `&=` `^=` `a++` `--a`


#### delete

`delete a`는 type에 대한 초기 값을 a에 할당한다. 

1. 정수, 동적 배열, 정적배열도 사용 가능하다. 

2. struct의 경우, 모든 member가 reset된 struct를 할당하고, 해당 member로 재귀된다.

3. mapping의 경우, `delete`는 영향을 미치지 않는다. delete a[x] 는 x에 저장된 값이 삭제된다.


### Conversions between Elementary Types

#### Implicit Conversions

컴파일러가 할당 중, 

1. 함수들의 arg를 전달할 때
2. 연산자를 적용할 때
3. 의미가 있고, 정보가 손실되지 않는 경우 암묵적 변환이 가능함

예시)

uint8 -> uint16 가능

uint128 -> uint256 가능

uint8 -> uint256 불가능 (uint256은 -1과 같은 값을 저장 할 수 없음)

예시 코드)

````solidity
uint8 y;
uint16 z;
uint32 x = y + z;
````


#### Explicit Conversions

컴파일러가 Implicit Conversion을 허용하지 않지만, 변환이 작동할 것이라 확신할 경우 explicit 변환이 가능하다. 

예시코드)

````solidity
uint16 a = 0x1234;
uint32 b = uint32(a); // b => 0x00001234 
assert(a == b);
````


### Conversions between Literals and Elementary Types

#### Integer Types

10진수 또는 16진수가 잘리지 않고 잘 나타날 수 있게 변환 가능하다.

````solidity
uint8 a = 12;
uint32 b = 1234; 
uint16 c = 0x123456; // 실패, 0x3456
````


#### Fixed-Size Byte Arrays 

10진수는 고정 크기 byte 배열로 implicit 변환 불가능하다.

16진수는 바이트 types의 크기에 정확히 맞는 경우만 implicit 변환이 가능하다.

예시 코드)

````solidity
bytes2 a = 54321; // 10진수, 허용안함
bytes2 b = 0x12; // 16진수, 허용안함
bytes2 e = 0x0012; // 16진수, 크기 맞음
bytes4 f = 0; // 예외적으로 0은 고정크기 바이트 type으로 implicit 변환 가능함
````


#### Addresses

1. 다른 literal들은 implicit하게 address type 변환이 불가능하고 `Address Literals` 만 가능하다.

2. `byte20` 또는 `integer` type에서 `address` type으로 explicit 변환은 가능하다.

3. `address`는 `payable(a)`을 사용해 `address payable`로 변환 가능하다.


