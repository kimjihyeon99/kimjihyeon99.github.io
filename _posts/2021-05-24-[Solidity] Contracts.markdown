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

단일 element만 검색할 수 있는 getter는 전체 array를 반환할때 높은 gas 비용을 방지하기 위해 존재함

전체 array를 반환하기 위해서는 함수를 작성해야함

````solidity
contract arrayExample {
    // public state variable
    uint[] public myArray;

    // 컴파일러에 의해 만들어진 getter
    /*
    function myArray(uint i) public view returns (uint) {
        return myArray[i];
    }
    */

    // 전체 array를 반환하는 function
    function getArray() public view returns (uint[] memory) {
        return myArray;
    }
}
````

### Function Modifiers

함수동작을 선언적으로 변경할 수 있다.

예를들어, 함수를 실행하기 전에 자동적으로 조건을 확인하는 modifier를 사용할 수 있다. 

상속가능한 속성이고, derived contract에 의해 재정의 될 수 있지만, `virtual`로 표시된 경우만 해당된다. 

*자세한 내용은 Modifer Overriding 섹션에서

예시코드)

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >0.7.0 <0.9.0;

contract owned {
    constructor() { owner = payable(msg.sender); }
    address payable owner;

    // derived된 contract에서 사용될 modifier임
    // body에 `_;`선언 의미 : 
    // owner가 onlyOwner를 부르면, 이 함수가실행되거나 그러지 않으면 예외처리가 된다. 
    modifier onlyOwner {
        require(
            msg.sender == owner,
            "Only owner can call this function."
        );
        _;
    }
}

contract destructible is owned {//owned contract 상속
    // destroy함수 적용, ??
    function destroy() public onlyOwner {
        selfdestruct(owner);
    }
}

contract priced {
    // modifier는 인자를 받을 수 있다.
    modifier costs(uint price) {
        if (msg.value >= price) {
            _;
        }
    }
}

contract Register is priced, destructible {//priced, destructible 상속
    mapping (address => bool) registeredAddresses;
    uint price;

    constructor(uint initialPrice) { price = initialPrice; }

    //`payable`키워드를 붙이는게 중요함.
    //그렇지 않으면, ehter를 전송할 수 없음!
    function register() public payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }

    function changePrice(uint _price) public onlyOwner {
        price = _price;
    }
}

contract Mutex {
    bool locked;
    modifier noReentrancy() {
        require(
            !locked,
            "Reentrant call."
        );
        locked = true;
        _;
        locked = false;
    }

    /// This function is protected by a mutex, which means that
    /// reentrant calls from within `msg.sender.call` cannot call `f` again.
    /// The `return 7` statement assigns 7 to the return value but still
    /// executes the statement `locked = false` in the modifier.
    function f() public noReentrancy returns (uint) {
        (bool success,) = msg.sender.call("");
        require(success);
        return 7;
    }
}
````

- contract `C`에 정의된 modifier에 액세스하려면 `C.m`을 사용해 virtual 조회 없이 참조할 수 있다.
- 현재 contract에 정의된 modifier만 사용할 수 있다. 
- 라이브러리에도 정의될 수 있지만, 동일 라이브러리 함수로 제한된다.

- 공백으로 구분된 리스트에서 여러개의 한정자를 지정하여 함수에 적용되며, 제시된 순서대로 평가된다.

- modifier는 수정되는 함수의 인수와 반환값을 암시적으로 액세스하거나 변경할 수 없다. 

- modifier 또는 function body로부터 명시적인 반환은 현재 modifier 또는 function body만 남는다.
- 반환 변수가 할당 되고 앞의 modifier에서 _ 뒤의 제어흐름이 계속된다. 

- 반환이 있는 modifier에서 명시적으로 반환된다. 

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
