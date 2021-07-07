---
layout: post
title: "python_json_solidity2"
tags: [학부생인턴, python,json,solidity]
---

## 이번주 과제 : 지난주에 이어 mapping 함수, reveal 함수도 적용해보기 

reveal 함수는 "name"으로 위치를 찾아 해당하는 "delegate_to"의 값를 추가해 주었다.

그리고 check 함수에 대한 키워드도 json 파일에 추가하고, "delegate_to"값을 수정하였다. 

하지만 mapping 함수는 "key_type" 과 "value_type"으로 위치를 찾아 "owner"의 값을 나누어 추가하였다.

그렇게 할 수 밖에 없는 이유는 "name"으로 찾기위해서는 depth가 같지 않아 mapping 변수가 "name"의 값이랑 같은지 비교할 수 없었다. 


### input.sol파일

````Solidity
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
        risk[don] = r;
        count = count + (r ? 1:0);
    }

    function check(bool r){
        require(r == risk[me]);
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

<img src="/assets/img/solidity/solidity2.JPG">
