---
layout: post
title: "[Solidity]Expressions and Control Structures"
date: "2021-05-16 12:00:00 +0200" 
image: 6.jpg
tags: [solidity, expressions, control]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/control-structures.html

## Expressions and Control Structures

### Control Structures

`if`, `else`, `while`, `do`, `for`, `break`, `continue`, `return`, `try/catch`

*if(1) {...} 불가능함. int -> bool 변환 안됨

### Function Calls

#### Internal Function calls

````solidity
pragma solidity >=0.4.22 <0.9.0;

contract C { //동일 contract 내 함수는 서로 internal 함수 호출 가능
    function g(uint a) public pure returns (uint ret) { return a + f(); }
    function f() internal pure returns (uint ret) { return g(7) + f(); }
}
````

#### External Function calls

외부 function calls의 경우 모든 함수 인자를 메모리에 복사해야함

*한 contract에서 다른 contract로의 함수 호출은, 자체 transaction을 생성하는 것이 아니라 전체 transaction의 부분으로서 메시지 호출이다. 

다른 contract의 함수를 호출할 때, 옵션과 함께 wei의 양이나 gas를 구체화 할 수 있다. 

````solidity
pragma solidity >=0.6.2 <0.9.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; } //payable type을 사용 해야함
}

contract Consumer {
    InfoFeed feed; //other contract
    function setFeed(InfoFeed addr) public { feed = addr; }
    function callFeed() public { feed.info{value: 10, gas: 800}(); } //option
}
````

#### Named Calls and Anonymous Function Parameters

함수 호출 인자는 {}로 둘러싸여 있을 경우 임의의 순서로 이름을 지정할 수 있다.

인자목록은 함수 선언의 매개 변수 목록과 이름이 일치해야지만 임의 순서대로 정렬 할 수 있다.

````solidity
contract C {
    mapping(uint => uint) data;

    function f() public {
        set({value: 2, key: 3}); // 인자 : value, key 
    }

    function set(uint key, uint value) public { //매개 변수 : key, value
        data[key] = value;
    }

}
````

#### Omitted Function Parameter Names

사용되지 않는 매개 변수의 이름은 생략할 수 있다.

스택에 존재하지만 액세스 할 수 없게 된다.

````solidity
contract C {
    // 두번째 매개변수 uint의 이름을 생략함
    function func(uint k, uint) public pure returns(uint) {
        return k;
    }
}
````

### Creating Contracts via new

contract는 `new`키워드를 사용하여 다른 contract를 만들 수 있다.

생성 중인 contract의 전체 코드는 생성한 contract가 컴파일 될 때 알아야하므로 재귀 생성-종속성이 불가능하다.

````solidity
contract C {
    D d = new D(4); // C's constructor의 부분으로서 실행됨

    function createD(uint arg) public {
        D newD = new D(arg);
        newD.x();
    }

    function createAndEndowD(uint arg, uint amount) public payable {
        // 생성과 함께, ether 보냄
        D newD = new D{value: amount}(arg);
        newD.x();
    }
}
````

### Order of Evaluation of Expressions

expression의 evaluation의 순서가 지정되어있지 않다.

구문이 순서대로 실행되고, boolean 표현식에 대한 단락이 수행된다는 것만 보장된다.

### Assignment

#### Destructuring Assignments and Returning Multiple Values

Solidity는 내부적으로 튜플 type을 사용하여 여러 값을 동시에 반환 할 수 있다.

이러한 변수는 새로 선언된 변수 또는 기존 변수에 할당 할 수 있다.

*튜플은 Solidity에서 올바른 형식이 아니고, expression의 구문 그룹화를 구성하는 데만 사용가능하다.

````solidity
contract C {
    function f() public pure returns (uint, bool, uint) {
        return (7, true, 2);
    }

    function g() public {
        //새로운 변수에  할당
        (uint x, , uint y) = f();     
    }
}
````

#### Complications for Arrays and Structs

*자세한 내용은 Data location and assignment behaviour 에서 보기

예시 코드)

````solidity
pragma solidity >=0.4.22 <0.9.0;

contract C {
    uint[20] x;

    function f() public {
        g(x);
        h(x);
    }

    function g(uint[20] memory y) internal pure {
        y[2] = 3; //1
    }

    function h(uint[20] storage y) internal {
        y[3] = 4; //2
    }
}
````

1. g(x) 호출은 메모리에 저장 값의 독립 복사본을 만들기 때문에 x에 영향을 미치지 않는다.

2. 복사본이 전달 되지 않고 참조만 전달 되므로 h(x)는 x를 수정할 수 있다.

### Scoping and Declarations



### Checked or Unchecked Arithmetic

### Error handling: Assert, Require, Revert and Exceptions
