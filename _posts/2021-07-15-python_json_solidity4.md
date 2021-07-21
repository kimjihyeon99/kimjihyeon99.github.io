---
layout: post
title: "python_json_solidity4"
tags: [학부생인턴, python,json,solidity]
---

## 이번주 과제 : example 폴더에서 zkay파일 json과 같이 visitor에 돌려보기, 이제까지 한 내용 정리해오기(2~3p)

reveal 이 들어있는 파일중, reveal 인자 중 파라미터와 로컬 변수만 있다면 `>@`를 추가한다. 

example 폴더 내의 zkay 파일을 가져와 json과 같이 돌렸을때 결과가 stone 파일로 잘 나오는지 확인한다.

다양한 코드에 돌려보는 것은 처음이라 모든 zkay 코드에 visitor가 돌아갈 수 있도록 코드를 수정하였다. 

에러사항이 많았지만, for문으로 json을 읽어오면서 해결되었다. 


### 테스트 결과 

### input.sol 

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma zkay >=0.2.0;

contract MedStats {
    final address hospital;
    uint@hospital count;
    mapping(address!x => bool@x) risk;

    constructor() public {
        hospital = me;
        count = 0;
    }

    function record(address pat, bool@me r) public {
        require(hospital == me);
        risk[pat] = reveal(r, pat);
        count = count + (r ? 1 : 0);
    }

    function check(bool@me r) public {
        require(reveal(r == risk[me], all));
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
					"owner" : "!@",
					"name" : "risk"
				}
			],
	"functions":[
		{
			"signature" : "6f0d85bb",
			"args" : [
				{
					"type": "address",
					"name": "pat"
				},
				{
					"type": "bool",
					"owner": "me",
					"delegate_to": "pat",
					"name": "r"
				}
			]
		},
		{
			"signature" : "6f0d85bb",
			"args" : [
				{
					"type": "bool",
					"owner": "me",
					"delegate_to": "all",
					"name": "r"
				}
			]
		}
	]
}
````

### 출력 결과

<img src="/assets/img/solidity/solidity4.JPG">


### BoolOperation.zkay

````solidity
pragma zkay >=0.2.0;

contract BoolOperations {
    function f(uint@me a, uint@me b) public {
        bool c;
        c = reveal((a > b) || (a == b), all);
        c = reveal((a > b) && (a == b), all);
        c = reveal(!(a > b), all);
    }
}
````

### BoolOperation_json

````json
{
	"functions":[
		{
			"signature" : "6f0d85bb",
			"args" : [
				{
					"type": "uint",
					"owner": "me",
					"delegate_to":"all",
					"name": "a"

				},
				{
					"type": "uint",
					"owner": "me",
					"delegate_to": "all",
					"name": "b"
				}
			]
		}
	]
}
````

### 출력결과 

````solidity
pragma zkay >=0.2.0;
contract BoolOperations {
    function f(uint@me>@all a, uint@me>@all b) public {
        bool c;
        c = (a > b) || (a == b);
        c = (a > b) && (a == b);
        c = !(a > b);
    }

}
````

### PLAS project 내용 정리

[ppt 링크](https://docs.google.com/presentation/d/1RBMQEPCjv1NK33vDz9qa3mwFQfxfeQKHPUezL2KbkGE/edit)
