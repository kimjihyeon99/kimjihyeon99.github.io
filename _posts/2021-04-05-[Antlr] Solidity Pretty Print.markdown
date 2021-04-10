---
layout: post
title: "[Antlr] Solidity Pretty Print 구현"
date: "2021-04-05 12:00:00 +0200" 
image: 11.jpg
tags: [antlr, prettyPrint, solidity]
categories: antlr
---

### Visitor로 Solidity Pretty Print 구현하기


구현 언어 : Python 

구현 IDE : Intelij

테스트 문법 : https://github.com/kimjihyeon99/zkay/blob/master/zkay/solidity_parser/Solidity.g4

테스트 Input : grammars-v4/test.sol

## 1. pragma 선언테스트

(1) Operator 없는 pragma 선언 

````sol
pragma solidity 0.4.4;
````

  - Operator 없는 pragma 선언 결과

![image](https://user-images.githubusercontent.com/44187194/113542546-7cfc6380-961f-11eb-8e2c-7bee1db99b99.png)


## 2. contract와 function 선언테스트 

(1) 빈 func

````sol
pragma solidity 0.4.4;
contract foo {
  function fun() {
  }
}
````

  - 빈 func 결과

![image](https://user-images.githubusercontent.com/44187194/113543162-c26d6080-9620-11eb-82d8-0cce9fb0fb34.png)

(2) identifier 선언 및 연산 func

````sol
pragma solidity 0.4.4;
contract test {
    function fun(uint256 a) {
        uint256 x = 3 ** a;
    }
}
````

  - identifier 선언 및 연산 func 결과

![image](https://user-images.githubusercontent.com/44187194/113970330-a7455f80-9871-11eb-8587-c09e69aa13b6.png)

(3) mapping 선언 및 연산 func

````sol
pragma solidity 0.4.4;
contract test {
    function fun(uint256 a) {
        var b = 5;
        uint256 c;
        mapping(address=>bytes32) d;
    }
}
````

  -  mapping 선언 및 연산 func 결과

![image](https://user-images.githubusercontent.com/44187194/113669912-632a5180-96ef-11eb-9a9c-ab00f77c82c1.png)

(4) if else 문 선언 func
 
```` sol
pragma solidity 0.4.4;
contract test {
   function fun(uint256 a) {
       if (a >= 8) {
           return;
       } else {
            var b = 7;
       }
   }
}
````
 
  -  if else 문 선언 func 결과

![image](https://user-images.githubusercontent.com/44187194/113883857-7e818380-97f9-11eb-8607-35d44526339e.png)

(5) while 문 선언 func 

````sol
pragma solidity 0.4.4;
contract test {
    function fun(uint256 a) {
        while (true) {
            uint256 x = 1;
            break;
            continue;
        }
        x = 9;
    }
}
````

  - while 문 선언 func 결과
  
![image](https://user-images.githubusercontent.com/44187194/113884847-49c1fc00-97fa-11eb-94a4-e43318a14ace.png)

(6) for 문 선언 func

조건문 없는 for 문

````sol
pragma solidity 0.4.4;
contract test {
    function fun(uint256 a) {
        uint256 i = 0;
        for (;;) {
            uint256 x = i;
            break;
            continue;
        }
    }
}
````

  - 조건문 없는 for 문 선언 func 결과 

![image](https://user-images.githubusercontent.com/44187194/113885403-c6ed7100-97fa-11eb-8e51-90e964f570e0.png)

조건문 있는 for 문

````sol
pragma solidity 0.4.4;
contract test {
    function fun(uint256 a) {
        uint256 i = 0;
        for (i = 0; i < 10; i++) {
            uint256 x = i;
            break;
            continue;
        }
    }
}
````
  - 조건문 있는 for 문 선언 func 결과 

![image](https://user-images.githubusercontent.com/44187194/113970168-5df51000-9871-11eb-8c38-ab2175c2babe.png)

(7) return 선언 func 

````sol
pragma solidity 0.4.4;
contract test {
    function f() returns(bool succeeded) {
        return false;
    }
}
````

  - return 선언 func 결과 

![image](https://user-images.githubusercontent.com/44187194/113971002-dd371380-9872-11eb-8ff6-e69307b5a087.png)


## 3. contract와 enum 선언테스트 

(1) 빈 enum 선언

````sol
pragma solidity 0.4.4;
contract c {
    enum foo { }
}
````

  - 빈 enum 선언 결과 

![image](https://user-images.githubusercontent.com/44187194/113667609-011c1d00-96ec-11eb-99ab-e077afcaa650.png)

(2) value 여러개인 enum 선언

````sol
pragma solidity 0.4.4;
contract c {
    enum validEnum { Value1, Value2, Value3, Value4 }
    function c () {
        a = validEnum.Value3;
    }
    validEnum a;
}
````

  - value 여러개인 enum 선언 결과

![image](https://user-images.githubusercontent.com/44187194/114257150-f5d13600-99f8-11eb-86b6-ea6c8ccee820.png)


## 4. expression 선언테스트

(1) SignExpr, PostCrementExpr, BitwiseNotExpr, NotExpr 사용

SignExpr : op=('+' | '-') expr=expression

PostCrementExpr : expr=expression op=('++' | '--')

BitwiseNotExpr : '~' expr=expression

NotExpr : '!' expr=expression

````sol
pragma solidity 0.4.4;
contract test {
    function f() {
        uint a = +10;
        a--;
        a = ~a;
        bool b = !true;
    }
}
````

  - SignExpr, PostCrementExpr, BitwiseNotExpr, NotExpr 사용 결과
 
![image](https://user-images.githubusercontent.com/44187194/114016291-a7f7e900-98a5-11eb-8a78-cad72b829c46.png)

(2) Tuple 사용

TupleExpr : expr=tupleExpression 

tupleExpression : '(' ( expression? ( ',' expression? )* ) ')'

````sol
pragma solidity 0.4.4;
contract test {
    function f() {
        uint256 a;
        (a,) = g();
        (,) = g();
        () = ();
    }
}
````

  - Tuple 사용 결과

![image](https://user-images.githubusercontent.com/44187194/114182794-009ab500-997e-11eb-95c9-98f973034374.png)

(3) IterExpr 사용

IterExpr : cond=expression '?' then_expr=expression ':' else_expr=expression

예1) 일반적인 사용

````sol
pragma solidity 0.4.4;
contract A {
    function f() {
        uint y = 1;
        uint x = 3 < 0 ? x = 3 : 6;
        true ? x = 3 : 4;
    }
}
````

예2) codition에 괄호 있는 형식

````sol
pragma solidity 0.4.4;
contract A {
    function f() {
        uint x = 3 > 0 ? 3 : 0;
        uint y = (3 > 0) ? 3 : 0;
    }
}
````

  - IterExpr 사용 결과

예1)

![image](https://user-images.githubusercontent.com/44187194/114183683-e6ada200-997e-11eb-9546-6a62cf8aa58e.png)

예2)

![image](https://user-images.githubusercontent.com/44187194/114256763-95d99000-99f6-11eb-82fd-bb53d5a02823.png)


(3) numberLiteral, Strimg Literal 사용

````sol
pragma solidity 0.4.4;
contract test {
    function fun(uint256 a) {
        var b = 2;
        uint256 c = 0x87;
        mapping(address=>bytes32) d;
        bytes32 name = "Solidity";
    }
}
````

  - numberLiteral, Strimg Literal 사용 결과

![image](https://user-images.githubusercontent.com/44187194/114185480-bcf57a80-9980-11eb-8ed2-c7d3430a5bc8.png)

(4) PrimitiveCastExpr 사용

PrimitiveCastExpr : elem_type=elementaryTypeName '(' expr=expression ')'

````sol
pragma solidity 0.4.4;
contract base {
    function fun() {
        uint64(2);
    }
}
````

  - PrimitiveCastExpr 사용 결과 

![image](https://user-images.githubusercontent.com/44187194/114256871-4f386580-99f7-11eb-984b-197c5b9dfb5c.png)


## 5. elementaryTypeName 선언테스트

elementaryTypeName : 

name=('address' | 'address payable' | 'bool' | Int | Uint | // Supported types

      'var' | 'string' | 'bytes' | 'byte' | Byte | Fixed | Ufixed )  ; // Unsupported types

(1) fixed 와 ufixed 사용

````sol
pragma solidity 0.4.4;
contract A {
    fixed40x40 storeMe;
    function f(ufixed x, fixed32x32 y) {
        ufixed8x8 a;
        fixed b;
    }
}
````

  - fixed 와 ufixed 사용 결과

![image](https://user-images.githubusercontent.com/44187194/114189045-b8cb5c00-9984-11eb-9bd2-0782a93799ce.png)


## 구현 미완성
(1) visitTupleExpression 구현 

![image](https://user-images.githubusercontent.com/44187194/114027251-235f9780-98b2-11eb-9422-86b0b2bf7d1a.png)

해당 조건을 어떻게 케이스를 나누어야할지 모르겠다.

![image](https://user-images.githubusercontent.com/44187194/114182959-393a8e80-997e-11eb-8eb2-a3707c81f4fd.png)

다음과 같이 구현을 했지만 리팩토링이 아직 필요하다. 

## 테스트 input과 g4파일과 다른 점

g4에서 

Library 타입이 사라짐

[] 안에 expression이 들어가야만 선언 가능

var ( tupleExpression ) 선언 불가능

 hex'00AA0000'와 같은 표현식 대신 0x[알파벳,숫자] 와 같은 형식으로 선언 가능
 
function f(uint a, uint b); 와 같은 형태의 함수 선언 불가능


