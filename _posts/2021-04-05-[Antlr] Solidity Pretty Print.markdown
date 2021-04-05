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

````sol
pragma solidity 0.4.4;
````

####  -테스트 1의 결과 

![image](https://user-images.githubusercontent.com/44187194/113542546-7cfc6380-961f-11eb-8e2c-7bee1db99b99.png)


## 2. contract와 function 선언테스트 
````sol
pragma solidity 0.4.4;
contract foo {
  function fun() {
  }
}
````
####  -테스트 2의 결과 

![image](https://user-images.githubusercontent.com/44187194/113543162-c26d6080-9620-11eb-82d8-0cce9fb0fb34.png)

## 3. contract와 enum 선언테스트 
````sol
pragma solidity 0.4.4;
contract c {
    enum foo { }
}
````
####  -테스트 3의 결과

![image](https://user-images.githubusercontent.com/44187194/113543928-53910700-9622-11eb-892b-de13e3e6619f.png)

