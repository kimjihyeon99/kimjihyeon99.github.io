---
layout: post
title: "python_json_solidity"
tags: [학부생인턴, python, json,solidity]
---

## 이번주 과제 : 연결된 json을 지난번에 [solidity언어를 pretty print한 코드](https://github.com/kimjihyeon99/Antlr/tree/main/prettyPrintSolidity)에 적용

지난번에 구현한 solidity 언어를 pretty print한 코드에 json 연결 코드를 추가하였다. 

가정은 json에 컨트랙트에서 변수들과 매개변수들에 대한 정보가 선언되어 있는 상태에서 json에서 읽어서 하는 것이다. 

이 가정에서 파란색 키워드를 생략한 solidity 코드를 input으로 넣으면 output으로 json에서 가져온 정보를 파란색 키워드 자리에 채워 넣었다. 

### input.sol 파일

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.4.4;
contract MedStats {
    final address hosipital;
    unit count;
    mapping(address!x =>bool@x) risk;

    constructor(){
        hospital = me;
        count =0;
    }

    function record(address don, bool r){
        require(hospital == me);
        risk[don] = reveal(r,don);
        count = count + (r ? 1:0);
    }

    function check(bool r){
        require(reveal(r == risk[me],all));
    }
}
````

### json 파일

````json
{
	"g_field" :[
		{
			"type": "uint",
			"owner" : "hospital",
			"name": "count"
		},
		{
			"type": "mapping",
			"k_type": "address",
			"v_type": "bool",
			"owner" : "x",
			"name" : "risk"
		}
	],
	"functions":[
		{
			"signature" : "6f0d85bb",
			"args" : [
				{
					"type": "address",
					"name": "don"
				},
				{
					"type": "bool",
					"owner": "me",
					"name": "r"
				}
			]
		}
	]
} 
````

### 출력 


<img src="/assets/img/solidity/solidity1.JPG">

### [구현코드](https://github.com/kimjihyeon99/Antlr/tree/main/prettyPrint_json_link)
