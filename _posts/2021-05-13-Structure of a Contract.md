---
layout: post
title: "Structure of a Contract"
tags: [solidity, structure, contract]
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/structure-of-a-contract.html

## Structure of a Contract


Solidity의 Contract는 객체 지향 언어의 **클래스**와 유사하다.

각 Contract는은 상태 변수, 함수, 함수 Modifier, 이벤트, 오류, Struct, enum 이 있다.

Contract는 다른 Contract로부터 **상속**받을 수 있다.


### State Variables

 State Variables은 `contract` 저장소에 값이 영구적으로 저장되는 변수
 
 ````solidity
pragma solidity >=0.4.0 <0.9.0;

contract SimpleStorage {
    uint storedData; // State variable
    // ...
}
 ````

* State VariablesType, Visibillity and Getters 내용은 Type 섹션에서 볼 수 있음 


### Functions

코드의 실행가능 단위

일반적으로 Contract 내부에 정의되지만, 외부에 정의 될 수 있음

 ````solidity
pragma solidity >0.7.0 <0.9.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}

//contract의 외부에 정의 예시
function helper(uint x) pure returns (uint) {
    return x * 2;
}
 ````
 
 
### Function Modifier

선언적 방식으로 Function의 함수의 의미를 수정하는데 사용할 수 있다.

매개 변수가 같은 한정자 이름을 가진 overlonding 불가능하다.

Function처럼 Modifier도 overlonding 가능하다.

*Contracts 섹션에서 자세한 내용

 ````solidity
pragma solidity >=0.4.22 <0.9.0;

contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(
            msg.sender == seller,
            "Only seller can call this."
        );
        _;
    }

    function abort() public view onlySeller { // Modifier usage
        // ...
    }
}
 ````
 
 
### Events

EVM(이더리움 가상머신) logging 기능과의 편의 인터페이스이다.

* contracts 섹션 내의 events 에서 선언 방법을 자세히 설명

 ````solidity
pragma solidity >=0.4.21 <0.9.0;

contract SimpleAuction {
    event HighestBidIncreased(address bidder, uint amount); // Event

    function bid() public payable {
        // ...
        emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
    }
}
 ````
 
 
### Errors

고장상황에 대한 설명 이름과 데이터를 정의할 수 있다.

`revert statements` 사용할 수 있다.

문자열 설명과 비교하여, `Errors`는 훨씬 저렴하고 추가 데이터를 인코딩할 수 있다.

*Contracts 섹션의 Errors and the Revert Statemennt 에 자세한 내용

 ````solidity
pragma solidity ^0.8.4;

/// 송금할 자금이 부족할때 error
/// `requested`가 요청됐지만, `available`만큼만 사용가능함.
error NotEnoughFunds(uint requested, uint available);

contract Token {
    mapping(address => uint) balances;
    function transfer(address to, uint amount) public {
        uint balance = balances[msg.sender];
        if (balance < amount)
            revert NotEnoughFunds(amount, balance);
        balances[msg.sender] -= amount;
        balances[to] += amount;
        // ...
    }
}
 ````
 
 
### Struct Types

`Struct`는 여러 변수를 그룹화 할 수 있는 사용자 정의 유형이다.

````solidity
pragma solidity >=0.4.0 <0.9.0;

contract Ballot {
    struct Voter { // Struct
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
````


### Enum Types 

`Enum`은 유한 집합 '상수 값'으로 사용자 정의 유형 생성 가능하다.

````solidity
pragma solidity >=0.4.0 <0.9.0;

contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}
````
