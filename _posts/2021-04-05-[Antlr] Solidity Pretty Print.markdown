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

![image](https://user-images.githubusercontent.com/44187194/113667157-5572cd00-96eb-11eb-8dc8-b44bf36a02d6.png)

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

## 

