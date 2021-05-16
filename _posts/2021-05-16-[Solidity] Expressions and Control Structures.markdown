---
layout: post
title: "[Solidity] Expressions and Control Structures"
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

선언된 변수는 byte 표현이 모두 0인 초기 기본값을 가진다. 

예)
- `bool` : false
- `uint/int` : 0
- `array/string` : empty array or string
- `enum` : default 값이 첫번째 member

예시코드)

````solidity
pragma solidity >=0.5.0 <0.9.0;
contract C {
    function minimalScoping() pure public {
        {
            uint same;
            same = 1;
        }

        {
            uint same;
            same = 3;
        }
    }
}
````

두 변수의 이름은 `same`으로 같지만, 범위가 분리되어있으므로 경고없이 컴파일 된다. 

-특별 규칙

x에 대한 첫번째 할당은 내부 변수가 아닌 외부변수를 할당함

````solidity
contract C {
    function f() pure public returns (uint) {
        uint x = 1;
        {
            x = 2; // 외부변수 x가 할당됨
            uint x;
        }
        return x; // x => 2
    }
}
````


### Checked or Unchecked Arithmetic

overflow 또는 underflow는 산술 연산의 결과 값이 제한되지 않은 정수로 실행될 때 결과 type의 범위를 벗어나는 상황

Solidity 0.8.0 이전, 산술 연산은 추가 검사를 도입하는 라이브러리의 광범위한 사용으로 이어지는 과소 또는 오버플로의 경우, 항상 wrapping 되었다.

Solidity 0.8.0 이후, 모든 산술 연산은 기본적으로 오버플로 및 언더플로우를 반복하기 때문에, 이러한 라이브러리를 사용할 필요가 없다.

이러한 동작을 얻기위해서, `Unchecked` 블록을 사용할 수 있다. 

````solidity
pragma solidity ^0.8.0;
contract C {
    function f(uint a, uint b) pure public returns (uint) {
        // underflow wrapping
        unchecked { return a - b; }
    }
    function g(uint a, uint b) pure public returns (uint) {
        // underflow에 revert 수행
        return a - b;
    }
}
````
`Unchecked` 블록은 block 내에서 어디서나 사용할 수 있지만, block 대체용도 사용은 불가능함

setting은 block 내에 statement에만 영향 준다.


### Error handling: Assert, Require, Revert and Exceptions


`assert`와 `require`함수는 조건을 검사하고, 조건이 충족되지 않으면 예외 처리하는데 사용된다.

#### Assert

`assert`함수는 painc type의 오류를 생성한다. 

아래와같은 특정 상황에서 컴파일러에 의해 동일한 오류가 생성된다. 

- `assert`함수는 internal오류를 테스트하고, 불변을 확인하는데만 사용해야한다. 

- 코드가 올바르게 작동하면 잘못된 외부입력에도 `painc`이 발생하지 않아야한다.

- 두 조건이 만족되면, contract에 bug가 있는 곳을 고쳐야한다. 

- `언어 분석 도구`는 contract를 평가하여 `painc`을 일으킬 수 있는 조건과 함수호출을 식별할 수 있다


*`panic`이 생기는 경우는 문서 참고


#### Require

`require`함수는 데이터 없이 Error를 생성하거나 Error(string) type을 생성한다. 

`require`함수는 실행시간까지 감지할 수 없는 유효한 조건을 보장하기위해 사용해야한다. 

여기에는 external contract에 대한 call로부터의 input 또는 return 값에 대한 condition이 포함된다.

아래와 같은 경우 `Error(String)` 예외가 컴파일러에 의해 생성된다.

- `x`가 `false`일 때 `require(x)`함수를 호출하는 경우
- `revert()` 또는 `revert("description")`을 사용하는 경우
- 코드가 없는 contract를 대상으로 external function call를 수행하는 경우
- contract가 `payable` 수정자 없이 public 함수에 의해 ether를 받는 경우 
- contract가 public getter 함수에 의해 Ether를 받는 경우

아래와 같은 경우 external call로 부터의 error data가 전송된다

- `.transfer`가 실패하는 경우
- message call을 통해 함수를 호출했지만, 제대로 완료되지 않은 경우(가스부족,해당 기능 없음)
- `new`키워드를 사용해 contract를 만들었지만, contract 생성이 제대로 완료되지 않은 경우

`require`는 필요에따라 문자열을 제공할 수 있지만, `assert`는 불가능하다. 


#### Rrevert

direct revert는 `revert` statement와 `revert` function으로 트리거 할 수 있다.

1. `revert`statement는 괄호가 없는 direct 인자로서 사용자 지정 오류를 가져온다. 

예시 코드 : `revert CustomError(arg1, arg2);`

2. 백워드 호환성의 이유로, `revert`function는 괄호를 사용하고 문자열을 받아들인다. 

예시 코드 : `revert(); revert(“description”);`


#### try/catch

external call의 실패는 `try/catch`로 찾아낼 수 있다.

1. `try`문은 external function call 또는 contract creation 을 나타내는 expression 뒤에 나와야한다.
2. expression 내부의 에러는 찾아지지 않고,  external call 자체 내에서만 revert가 일어난다.
3. return 부분은 external call에서 반환된 type과 일치하는 return 변수를 선언한다. 
4. 오류가 없는 경우 이런 변수가 할당되고, 첫번째 성공 block 내에 contract의 실행이 계속된다.
5. 성공 block이 끝에 도달하면, catch block 이후에 실행이 계속된다. 

````solidity
pragma solidity >0.8.0;

interface DataFeed { function getData(address token) external returns (uint value); }

contract FeedConsumer {
    DataFeed feed;
    uint errorCount;
    function rate(address token) public returns (uint value, bool success) {
        require(errorCount < 10); //에러가 10번이상이면 영구적으로 수행 불가능
        try feed.getData(token) returns (uint v) {// external call
            return (v, true);
        } catch Error(string memory /*reason*/) {
            errorCount++;
            return (0, false);
        } catch Panic(uint /*errorCode*/) {
            errorCount++;
            return (0, false);
        } catch (bytes memory /*lowLevelData*/) {
            //revert()가 수행되면 해당 내용이 수행
            errorCount++;
            return (0, false);
        }
    }
}
````
