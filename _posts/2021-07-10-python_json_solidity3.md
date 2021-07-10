---
layout: post
title: "python_json_solidity3"
tags: [학부생인턴, python,json,solidity]
---

## 이번주 과제 : 지난주 과제에서 reveal 함수 수정하기, example 폴더에서 zkay파일 reveal부분 찾아서 스톤 코드로 바꾸기

이번주는 지난주에 받은 피드백을 기반으로 코드를 수정하였다.

reveal 에 대한 키워드를 삽입할때 없는 상태에서 새로 reveal을 넣는 것이 아니라, @me>@don 과 같이 delegate 대상을 함수 헤더에 추가해달라고 하셨다.

이를 수정하기 위해서 visitParmeter 부분을 수정하였고, 이 부분에서 코드를 추가해 >@ 와 json파일에서 가져온 키워드를 추가하였다.

### input.sol

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.4.4;
contract MedStats {
    final address hosipital;
    unit count;
    mapping(address=>bool) risk;

    constructor(){
        hospital = me;
        count =0;
    }

    function record(address don, bool r){
        require(hospital == me);
        risk[don] = reveal(r, don);
        count = count + (r ? 1:0);
    }

    function check(bool r){
        require(reveal(r == risk[me], all));
    }
 }
````

### json파일

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
					"name": "don"
				},
				{
					"type": "bool",
					"owner": "me",
					"delegate_to": "don",
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


### 결과

<img src="/assets/img/solidity/solidity3.JPG">
