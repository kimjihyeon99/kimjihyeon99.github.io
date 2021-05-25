---
layout: post
title: "[Solidity] Contracts"
date: "2021-05-24 12:00:00 +0200" 
image: 5.jpg
tags: [solidity, expressions, control]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/contracts.html

## Contracts

개념 정리:

- 객체 지향 언어의 클래스와 유사함
- state variables에는 영구 데이터 및 이런 변수를 수정할 수 있는 함수가 포함됨
- 다른 contract에서 함수를 호출하는 것은 EVM함수를 호출을 수행하며, state variables에 접근할 수 없도록 컨텍스트를 전환한다. 
- 이더리움에는 특정 event에서 함수를 자동으로 호출하는 개념이 없어, 어떤 event가 발생하려면 contract와 function을 호출해야한다.

### Creating Contracts

- contract가 생성되면 해당 생성자가 한번 실행된다.
- 생성자는 선택사항이고, 하나만 사용할 수 있으므로 overloading이 지원되지 않는다.
- 생성자가 실행된 후, contract의 최종 코드가 블록체인에 저장됨
- 코드에는 public 및 external function, 함수 호출을 통해 도달할 수있는 모든 함수가 포함되고, 생성자코드와 생성자에서 호출된 internal function만 포함되지 않는다.
- contract에서 다른 contract를 작성하려면 작성된 contract의 소스코드를 creater에게 알려야한다. 
   -> 순환 생성 dependencies 가 불가능하다는 의미
   
 // OwnedToken 코드 remix 실행방법??

### Visibillity and Getter

Solidity는 두가지 종류의 함수 호출(internal, external)이 있다. 

실제 EVM(=메시지) 호출을 생성하지 않는 internal호출, 이를 실행하는 external 호출

따라서  functions 와 state variables에 대한 가시성이 있다.


`external`

- contract interface의 일부, 다른 contract와 거래를 통해 호출될 수 있다. 
- external 함수 f는 interal하게 호출할 수 없다. (f() 는 불가능, this.f()는 가능) 

`public`

- contract interface의 일부, internal하게 또는 메시지를 통해 호출할 수 있다.
- 자동 getter 함수가 생성된다. 


`internal`

- `this`를 사용하지 않고, functions 과 state variable 은 internal하게 액세스 할 수 있다. 
- state variables의 default 가시성이다. 
 
 
`private`

- `private`함수와 state variables은 오직 정의된 contract에 대해서만 표시된다.

Visibillity을 나타내는 키워드의 위치는 state variable 유형 뒤에, function 매개변수 목록과 반환 매개변수 목록 사이에 제공된다. 

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
````
//추가 예시

#### Getter Functions

컴파일러는 모든 `public`변수에 대한 getter함수를 자동 생성한다. 

어떠한 인자도 취하지 않고, unit를 반환하는 함수를 생성한다.

state variables는 선언될 때 초기화 할 수 있다.

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public view returns (uint) {
        return c.data(); //getter 호출
    }
}
````


### Function Modifiers

### Constant and Immutable State Variables

#### Constant

#### Immutable

### Functions

#### Function Parameters and Return Variables

##### Function Parameters

##### Return Variables

##### Returning Multiple Values

#### View Functions

#### Pure Functions

#### Receive Ether Function

#### Fallback Function

#### Function Overloading

##### Overload resolution and Argument matching

### Events

#### Additional Resources for Understanding Events

### Errors and the Revert Statement

### Inheritance

#### Function Overriding

#### Modifier Overriding

#### Constructors

#### Arguments for Base Constructors

#### Multiple Inheritance and Linearization

#### Inheriting Different Kinds of Members of the Same Name

### Abstract Contracts

### Interfaces

### Libraries

#### Function Signatures and Selectors in Libraries

#### Call Protection For Libraries

### Using For
