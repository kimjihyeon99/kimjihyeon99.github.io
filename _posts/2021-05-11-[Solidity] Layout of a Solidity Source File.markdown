---
layout: post
title: "[Solidity] Layout of a Solidity Source File"
date: "2021-05-11 12:00:00 +0200" 
image: 9.jpg
tags: [solidity, Source File]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/layout-of-source-files.html

## Layout of a Solidity Source File

소스파일은 임의 수의 contract definition, directives, pragma directives 

and struct, enum, function, error and constant variable definitions 를 포함 할 수 있다.


### SPDX License Identifier

smart contract 에대한 신뢰는 소스 코드를 사용할 수 있다면 더 잘 확립될 수 있다.

Solidity 컴파일러는 기계가 읽을 수 있는 `SPDX 라이센스 식별자`의 사용을 권장한다.


### Progmas

1. 키워드는 특정 컴파일러 기능 또는 검사를 활성화하는 데 사용됨
2. 지시문은 항상 본 파일의 로컬에 있으므로, 활성화하려면 pragma를 모두 추가해야함
3. 다른 파일을 가져오는 경우 해당 파일의 pragma가 가져오기 파일에 자동으로 적용안됨 

변경 내용이 포함된 릴리스에 대해서는 변경 로그를 항상 읽어보는 것이 좋다.

버전 사용 예시 `pragma solidity ^0.5.2;`

0.5.2 이전 버전의 컴파일러로 컴파일하지 않으며, 0.6.0 버전부터 컴파일러에서 동작하지 않는다.

* pragma 버전을 사용해도 컴파일러의 버전은 변경되지 않는다. 

### Importing other Source Files 

global 수준에서 파일 가져오기

````solidity
import "filename";
````

모든 global 기호를 멤버로하는 새 글로벌 기호 이름을 만든다.

````solidity
import * as symbolName from "filename";

import "filename" as symbolName;

````


### Comments

````solidity
// This is a single-line comment.

/*
This is a
multi-line comment.
*/
````
