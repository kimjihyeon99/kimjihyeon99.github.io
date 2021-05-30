---
layout: post
title: "[Solidity] Contracts"
date: "2021-05-24 12:00:00 +0200" 
image: 5.jpg
tags: [solidity, expressions, control]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/contracts.html

## Contracts

개념 정리:

- 객체 지향 언어의 클래스와 유사함
- state variables에는 영구 데이터 및 이런 변수를 수정할 수 있는 함수가 포함됨
- 다른 contract에서 함수를 호출하는 것은 EVM함수를 호출을 수행하며, state variables에 접근할 수 없도록 컨텍스트를 전환한다. 
- 이더리움에는 특정 event에서 함수를 자동으로 호출하는 개념이 없어, 어떤 event가 발생하려면 contract와 function을 호출해야한다.

### Creating Contracts

- contract가 생성되면 해당 생성자가 한번 실행된다.
- 생성자는 선택사항이고, 하나만 사용할 수 있으므로 overloading이 지원되지 않는다.
- 생성자가 실행된 후, contract의 최종 코드가 블록체인에 저장됨
- 코드에는 public 및 external function, 함수 호출을 통해 도달할 수있는 모든 함수가 포함되고, 생성자코드와 생성자에서 호출된 internal function만 포함되지 않는다.
- contract에서 다른 contract를 작성하려면 작성된 contract의 소스코드를 creater에게 알려야한다. 
   -> 순환 생성 dependencies 가 불가능하다는 의미
   
 // OwnedToken 코드 remix 실행방법??

### Visibillity and Getter

Solidity는 두가지 종류의 함수 호출(internal, external)이 있다. 

실제 EVM(=메시지) 호출을 생성하지 않는 internal호출, 이를 실행하는 external 호출

따라서  functions 와 state variables에 대한 가시성이 있다.


`external`

- contract interface의 일부, 다른 contract와 거래를 통해 호출될 수 있다. 
- external 함수 f는 interal하게 호출할 수 없다. (f() 는 불가능, this.f()는 가능) 

`public`

- contract interface의 일부, internal하게 또는 메시지를 통해 호출할 수 있다.
- 자동 getter 함수가 생성된다. 


`internal`

- `this`를 사용하지 않고, functions 과 state variable 은 internal하게 액세스 할 수 있다. 
- state variables의 default 가시성이다. 
 
 
`private`

- `private`함수와 state variables은 오직 정의된 contract에 대해서만 표시된다.

Visibillity을 나타내는 키워드의 위치는 state variable 유형 뒤에, function 매개변수 목록과 반환 매개변수 목록 사이에 제공된다. 

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
````

#### Getter Functions

컴파일러는 모든 `public`변수에 대한 getter함수를 자동 생성한다. 

어떠한 인자도 취하지 않고, unit를 반환하는 함수를 생성한다.

state variables는 선언될 때 초기화 할 수 있다.

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public view returns (uint) {
        return c.data(); //getter 호출
    }
}
````

단일 element만 검색할 수 있는 getter는 전체 array를 반환할때 높은 gas 비용을 방지하기 위해 존재함

전체 array를 반환하기 위해서는 함수를 작성해야함

````solidity
contract arrayExample {
    // public state variable
    uint[] public myArray;

    // 컴파일러에 의해 만들어진 getter
    /*
    function myArray(uint i) public view returns (uint) {
        return myArray[i];
    }
    */

    // 전체 array를 반환하는 function
    function getArray() public view returns (uint[] memory) {
        return myArray;
    }
}
````

### Function Modifiers

함수동작을 선언적으로 변경할 수 있다.

예를들어, 함수를 실행하기 전에 자동적으로 조건을 확인하는 modifier를 사용할 수 있다. 

상속가능한 속성이고, derived contract에 의해 재정의 될 수 있지만, `virtual`로 표시된 경우만 해당된다. 

*자세한 내용은 Modifer Overriding 섹션에서

예시코드)

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >0.7.0 <0.9.0;

contract owned {
    constructor() { owner = payable(msg.sender); }
    address payable owner;

    // derived된 contract에서 사용될 modifier임
    // body에 `_;`선언 의미 : 
    // owner가 onlyOwner를 부르면, 이 함수가실행되거나 그러지 않으면 예외처리가 된다. 
    modifier onlyOwner {
        require(
            msg.sender == owner,
            "Only owner can call this function."
        );
        _;
    }
}

contract destructible is owned {//owned contract 상속
    // destroy함수 적용, ??
    function destroy() public onlyOwner {
        selfdestruct(owner);
    }
}

contract priced {
    // modifier는 인자를 받을 수 있다.
    modifier costs(uint price) {
        if (msg.value >= price) {
            _;
        }
    }
}

contract Register is priced, destructible {//priced, destructible 상속
    mapping (address => bool) registeredAddresses;
    uint price;

    constructor(uint initialPrice) { price = initialPrice; }

    //`payable`키워드를 붙이는게 중요함.
    //그렇지 않으면, ehter를 전송할 수 없음!
    function register() public payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }

    function changePrice(uint _price) public onlyOwner {
        price = _price;
    }
}

contract Mutex {
    bool locked;
    modifier noReentrancy() {
        require(
            !locked,
            "Reentrant call."
        );
        locked = true;
        _;
        locked = false;
    }

    /// This function is protected by a mutex, which means that
    /// reentrant calls from within `msg.sender.call` cannot call `f` again.
    /// The `return 7` statement assigns 7 to the return value but still
    /// executes the statement `locked = false` in the modifier.
    function f() public noReentrancy returns (uint) {
        (bool success,) = msg.sender.call("");
        require(success);
        return 7;
    }
}
````

- contract `C`에 정의된 modifier에 액세스하려면 `C.m`을 사용해 virtual 조회 없이 참조할 수 있다.
- 현재 contract에 정의된 modifier만 사용할 수 있다. 
- 라이브러리에도 정의될 수 있지만, 동일 라이브러리 함수로 제한된다.

- 공백으로 구분된 리스트에서 여러개의 한정자를 지정하여 함수에 적용되며, 제시된 순서대로 평가된다.

- modifier는 수정되는 함수의 인수와 반환값을 암시적으로 액세스하거나 변경할 수 없다. 

- modifier 또는 function body로부터 명시적인 반환은 현재 modifier 또는 function body만 남는다.
- 반환 변수가 할당 되고 앞의 modifier에서 _ 뒤의 제어흐름이 계속된다. 

- 반환이 있는 modifier에서 명시적으로 반환된다. 

- `_` 기호는 fucntion 본문으로 대체 된다.  

### Constant and Immutable State Variables

State variables은 `constant` 또는 `immutable`로 선언될 수 있다. 

- 두 경우 모두 contract 체결 후에는 변수를 수정할 수 없다.
- `constant`의 경우, compile-time에 값을 고정해야하지만
- `immutable`의 경우, 계약 체결 시간에 값을 할당할 수 있다.   

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.4;

uint constant X = 32**22 + 8;

contract C {
    string constant TEXT = "abc";
    bytes32 constant MY_HASH = keccak256("abc");
    uint immutable decimals;
    uint immutable maxBalance;
    address immutable owner = msg.sender;

    constructor(uint _decimals, address _reference) {
        decimals = _decimals;
        // immutable에 대한 할당은 환경에 액세스 가능함.
        maxBalance = _reference.balance;
    }

    function isBalanceTooHigh(address _other) public view returns (bool) {
        return _other.balance > maxBalance;
    }
}
````

#### Constant

- 컴파일시 상수값이어야 하고, 변수가 선언된 위치에 할당되어야 한다. 
-  storage, blockchain data, execution data 에 액세스하거나 external contract를 호출하는 어떤 expression도 허용되지 않는다.
- 메모리 할당에  side-effect가 있을 수 있는 Expression들은 허용되지만, 
- 다른 메모리 object에 side-effect가 있을 수 있는 Expression들은 허용되지 않는다.
- built-in functions은 허용된다.(keccak256는 예외- external contract를 호출하기 때문에)

* 메모리 할당에 대한 side-effect가 허용되는 이유는 복잡한 object를 구성할 수 있어야하기 때문이다.

** remind

storage : block.timestamp, address(this).balance, block.number
blockchain data : msg.value, gasleft()
built-in function : keccak256, sha256, ripemd160, ecrecover, addmod, mulmod

#### Immutable

- `constant`로 선언된 것보다 덜 제한적임
- contract의 생성자 또는 선언 시점에서 임의 값을 할당할 수 있다. 
- 계약 체결동안 읽을 수 없고, 오직 한번만 할당할 수 있다. 

- 컴파일러에 의해 생성된 contract 생성 코드는 `immutable`에 대한 모든 참조를 할당된 값으로 대체함으로써 반환되기전에 contract의 runtime 코드를 수정할 것이다. 
- **컴파일러에서 생성된 런타임 코드**를 블록체인에 실제로 **저장된 런타임 코드**와 비교하는 것이 중요!

### Functions

"free function"이라고 불리는 contract의 외부 함수는 항상 `internal`가시성을 포함하고 있다.

다음 코드는 "free functions"을 호출하는 모든 contract를 포함한다.

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >0.7.0 <0.9.0;

function sum(uint[] memory _arr) pure returns (uint s) {
    for (uint i = 0; i < _arr.length; i++)
        s += _arr[i];
}

contract ArrayExample {
    bool found;
    function f(uint[] memory _arr) public {
        // internal하게 "free function"을 호출
        // 컴파일러는 contract에 code를 추가함
        uint s = sum(_arr);
        require(s >= 10);
        found = true;
    }
}
````

#### Function Parameters and Return Variables

function은 입력된 매개변수를 입력으로 사용하고, **임의 개수의 값을 출력으로 반환**할 수 있다.

##### Function Parameters

Function Parameters는 변수와 동일한 방식으로 선언되면, 사용되지 않은 매개변수의 이름은 생략 가능함 

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract Simple {
    uint sum;
    function taker(uint _a, uint _b) public {
        sum = _a + _b;
    }
}
`````

Function Parameters는 다른 local 변수로 사용할 수 있고, 할당될 수도 있다. 

##### Return Variables

`returns`키워드 뒤에 동일한 구분으로 선언된다. 

예시)  function parameters로 전달된 두개의 정수의 합과 곱을 반환

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract Simple {
    function arithmetic(uint _a, uint _b)
        public
        pure
        returns (uint o_sum, uint o_product)
    {
        o_sum = _a + _b;
        o_product = _a * _b;
    }
}
````

return variables의 이름은 생략가능하다.

다른 local 변수로 사용될 수 있으며 기본값으로 초기화되며 재할당될 때까지 해당 값을 가진다. 

명시적으로 할당한 다음 위와 같이 함수를 종료하거나 return문을 사용하여 반환 값을 직접 제공가능하다. 

##### Returning Multiple Values

함수의 return 타입이 여러개인경우 `return(v0, v1, ..., vn)`을 사용하여 여러값을 반환할 수 있다. 

단, return variable의 유형이 동일 해야하며, 암묵적 변환 후 해당 유형이 일치해야한다. 

#### View Functions

function은 상태를 수정하지 않겠다고 약속하는 경우 `view`로 선언될 수 있다. 

다음 문장은 상태를 modify한다고 간주된다.

- state variable에 쓰는 것
- event를 보내는 것
- 다른 contract를 생성하는 것
- selfdestruct를 사용 하는 것
- 호출을 통해 ether를 전송하는 것
- view 또는 pure 로 mark되지 않은 함수를 호출하는 것
- low-level 호출를 사용하는 것
- 특정 opcode를 포함한 inline assembly 사용하는 것

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;

contract C {
    function f(uint a, uint b) public view returns (uint) {
        return a * (b + 42) + block.timestamp;
    }
}
````

#### Pure Functions

function는 상태를 읽거나 수정하지 않겠다고 약속하는 경우 `pure` 로 선언될 수 있다.

다음 문장은 상태로 부터 읽는 것으로 간주된다. 

- state variable에서 읽는 것
- block, tx, msg의 member 중 하나에 접근하는 것
- pure 로 mark되지 않는 함수를 호출하는 것
- 특정 opcode를 포함한 inline assembly 사용하는 것
- `address(this).balance` or `<address>.balance.` 에 접근하는 것


````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;

contract C {
    function f(uint a, uint b) public pure returns (uint) {
        return a * (b + 42);
    }
}
````   

`pure`함수는 `revert()`와 `require()` 기능을 사용할 수 있다. 

`STATICCALL opcode` 와 동일한 동작

#### Receive Ether Function

contract는 하나의 `receive` function을 가질 수 있다.

선언식 : `receive() external payable { ... }`

- `function`키워드 없음
- arguments를 가지고 있지 않음
- 어떤 것도 return 할 수 없음
- `external` 가시성을 가져야하고, `payable` 상태를 가져야한다. 
- `virtual`될 수 있고, override할 수 있고, modifier를 가질 수 있다. 

동작 과정
- 비어있는 calldata를 가진 contract 호출로 실행된다.
- plain Ether transfers(`.send()` or `.transfer()`)로 수행된다.
- 이러한 function이 없고, payable fallback function만 존재하면, 해당 함수가 plain Ether transfers를 호출한다. 
- 두 함수 모두 없으면, contract는 정상적인 거래를 통해 ether를 receive할 수 없다. 

receive function 은 이용가능한 2300 gas에만 의존할 수 있으므로, 기본 logging을 제외한 다른 작업을 수행할 공간이 없다. 

다음작업은 2300 gas보다 더 많은 gas를 사용한다. 

- storage 작성하는 것
- contract 생성하는 것
- 큰 양의 gas를 소비하는 external 함수 호출하는 것
- ether를 전성하는 것

`receive` 함수를 사용하는 예 

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.0 <0.9.0;

//해당 contract는 ether가 보낸 모든 ether를 되돌릴 수 없는 상태로 유지함
contract Sink {
    event Received(address, uint);
    receive() external payable {
        emit Received(msg.sender, msg.value);
    }
}
````

#### Fallback Function

#### Function Overloading

contract에는 이름이 같지만 매개변수 유형이 다른 여러 function이 포함될 수있다. => overloaging

예시 코드) 

contract A의 범위에서 함수 f의 overloading을 보여줌

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract A {
    function f(uint _in) public pure returns (uint out) {
        out = _in;
    }

    function f(uint _in, bool _really) public pure returns (uint out) {
        if (_really)
            out = _in;
    }
}
Ove
````


##### Overload resolution and Argument matching

Overloaded functions 는 function 호출에 제공된 인자와 현재 scope에 function 선언을 일치시켜 선택한다. 

만약 모든 인자가 implicit 하게 예상 types으로 변환할 수 있으면, function은 overload 후보자로서 선택된다. 

만약 후보가 정확히 하나가 아니면, resolution 은 실패함

> return parameter는 overload resolution을 고려하지 않음

````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

contract A {
    function f(uint8 _in) public pure returns (uint8 out) {
        out = _in;
    }

    function f(uint256 _in) public pure returns (uint256 out) {
        out = _in;
    }
}
//f(50)을 호출하면, unit8 및 uint256 타입으로 암시적으로 변환할 수 있으므로, type error 발생함
//하지만 uint256은 암시적으로 uint8로 변환할 수 없으므로, f(256)은 f(uint256)으로 해결할 수 있음
````

### Events

- EVM의 logging 기능 위에 추상화를 제공함
- contract의 상속가능한 member 임
- event를 호출하면 인자가 트랜잭션 log에 저장됨
- log는 contract의 주소와 연결되어있고, 블록체인에 통합되어있고, block에 엑세스할 수 있는한 존재한다.
- log 및 event 데이터는 contract 내에서 엑세스할 수 없다. (log를 생성한 contract에서도 액세스 불가능)


````solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.21 <0.9.0;

contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // event는 'emit'을 통해 전송됨, event 이름과 인자를 따른다.
        // 이런 호출은 'deposit'에 대한 필터링을 통해 JavaScript API에서 탐지할 수 있다. 
        emit Deposit(msg.sender, _id, msg.value);
    }
}
````

 JavaScript API에서 사용
 
 ````Solidity
var abi = /* abi as generated by the compiler */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

var depositEvent = clientReceipt.Deposit();

// watch for changes
depositEvent.watch(function(error, result){
    // result contains non-indexed arguments and topics
    // given to the `Deposit` call.
    if (!error)
        console.log(result);
});


// Or pass a callback to start watching immediately
var depositEvent = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
 ````
 
 결과
 
 ````Solidity
 {
   "returnValues": {
       "_from": "0x1111…FFFFCCCC",
       "_id": "0x50…sd5adb20",
       "_value": "0x420042"
   },
   "raw": {
       "data": "0x7f…91385",
       "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
   }
}
 ````

### Errors and the Revert Statement

- Error는 사용자에게 작업에 실패한 이유를 설명하는 편리하고 효율적으로 gas를 사용할 수 있는 방법을 제공한다.
- contract의 내부와 외부에 정의 될 수 있다
- 현재 호출의 모든 변경사항을 되돌리고, 오류데이터를 호출자에게 전달하는 revert와 함께 사용해야한다.

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

/// transfer를 위한 InsufficientBalance
/// @param available : 이용가능한 balance (필수)
/// @param required : transfer 양을 요청 (선택)
error InsufficientBalance(uint256 available, uint256 required);

contract TestToken {
    mapping(address => uint) balance;
    function transfer(address to, uint256 amount) public {
        if (amount > balance[msg.sender])
            revert InsufficientBalance({ //revert와 error 같이 사용함
                available: balance[msg.sender],
                required: amount
            });
        balance[msg.sender] -= amount;
        balance[to] += amount;
    }
    // ...
}
````

- Error는  overloaded 또는 overridden은 불가능하지만, 상속은 가능함
- scope가 구분되는한 동일한 error를 여러곳에서 정의 가능함
- 데이터를 생성한 다음, revert 작업을 통해 호출자에게 전달되어 off-chain 구성요소로 돌아가거나 try/catch명령 문으로 데이터를 catch한다.

- 매개변수를 제공하지 않는 경우, 4바이트만 필요하고 NATSpec을 사용하여 chain에 저장되지 않은 오류의 원인을 설명할 수 있다. 
- 구체적으로 errer instance는 동일한 이름 및 type의 함수에 대한 호출이 `revert`에서 반환 데이터로 사용되는 것과 동일한 방식으로 ABI로 인코딩된다. 
- 결국, 데이터는 4바이트 selector와 abi 인코딩 데이터로 구성된다. 

`require(condition, "description");` == `if (!condition) revert Error("description")`


### Inheritance

다형성을 포함한 다중 상속 지원

- `virtual`이나 `override`키워드를 사용해서 명시적으로 사용하도록 설정해야함
- contract가 다른 contract로부터 상속했을때, 하나의 contract만 blockchain에서 생성되고, 기본 contract 코드는 만들어진 contract로 컴파일 됨
- state variable shadowing은 error로 간주됨.
- derived contract은 동일한 이름을 가진 state variable가 어디에도 없는 경우에만 선언할 수 있다.

* shadowing : internal 변수와 external 변수가 동일한 이름으로 정의될때 발생,

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;


contract Owned {
    constructor() { owner = payable(msg.sender); }
    address payable owner;
}


// Use `is` to derive from another contract. Derived
// contracts can access all non-private members including
// internal functions and state variables. These cannot be
// accessed externally via `this`, though.
contract Destructible is Owned {
    // `virtual`는 함수를 바꿀 수 있다. 
    function destroy() virtual public {
        if (msg.sender == owner) selfdestruct(owner);
    }
}

abstract contract Config {
    function lookup(uint id) public virtual returns (address adr);
}


abstract contract NameReg {
    function register(bytes32 name) public virtual;
    function unregister() public virtual;
}


// Multiple inheritance is possible. Note that `owned` is
// also a base class of `Destructible`, yet there is only a single
// instance of `owned` (as for virtual inheritance in C++).
contract Named is Owned, Destructible {
    constructor(bytes32 name) {
        Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
        NameReg(config.lookup(1)).register(name);
    }

    // Functions can be overridden by another function with the same name and
    // the same number/types of inputs.  If the overriding function has different
    // types of output parameters, that causes an error.
    // Both local and message-based function calls take these overrides
    // into account.
    // If you want the function to override, you need to use the
    // `override` keyword. You need to specify the `virtual` keyword again
    // if you want this function to be overridden again.
    function destroy() public virtual override {
        if (msg.sender == owner) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).unregister();
            // It is still possible to call a specific
            // overridden function.
            Destructible.destroy();
        }
    }
}


// If a constructor takes an argument, it needs to be
// provided in the header or modifier-invocation-style at
// the constructor of the derived contract (see below).
contract PriceFeed is Owned, Destructible, Named("GoldFeed") {
    function updateInfo(uint newInfo) public {
        if (msg.sender == owner) info = newInfo;
    }

    // Here, we only specify `override` and not `virtual`.
    // This means that contracts deriving from `PriceFeed`
    // cannot change the behaviour of `destroy` anymore.
    function destroy() public override(Destructible, Named) { Named.destroy(); }
    function get() public view returns(uint r) { return info; }

    uint info;
}
````

#### Function Overriding

- 만약 기본 함수가 `virtual`로 표시된경우, 해당 동작을 변경하기 위해 contract를 상속하여 재정의 할 수 있다.
- 그 후, 재정의 함수는 function header에서 `override`키워드를 사용해야 한다.
- `external`가시성을 `public`으로만 변경가능
-  `mutability`은 순서에 따라 더 strict한 것으로 변경될 수 있다. 

: `non payable`은 `view`와 `pure`로 override될 수 있고,

   `view`는 `pure`로 override될 수 있고,
   
   `payable`은 예외적으로 다른 mutability로 변경될 수 없다. 
   
````Solidity
pragma solidity >=0.7.0 <0.9.0;

contract Base
{
    function foo() virtual external view {}
}

contract Middle is Base {}

contract Inherited is Middle
{
   //view -> pure
    function foo() override public pure {}
}
`````

동일한 함수를 정의하고, 아직 다른 기본 contract에 의해 override되지 않은 모든 기본 contract를 지정해야한다. 

만약 contract가 여러 base contract 에서 동일한 기능을 상속받는 경우, 명시적으로 해야함

예시코드)

````Solidity
contract Base1
{
    function foo() virtual public {}
}

contract Base2
{
    function foo() virtual public {}
}

contract Inherited is Base1, Base2
{
    // Derives from multiple bases defining foo(), so we must explicitly
    // override it
    function foo() public override(Base1, Base2) {}
}
````

함수가 공통 base contract에 정의되어 있거나 이미 다른 모든 함수를 재정의하는 공통 base contract에 고유 function이 있는경우 

명시적 지정자는 필요하지 않다.

예시 코드)

````Solidity
contract A { function f() public pure{} }
contract B is A {}
contract C is A {}
// No explicit override required
contract D is B, C {}
````

> `private`가시성을 가진 function 은 `virtual`이 될 수 없다. 

function의 매개변수와 반환 형식이 변수의 getter function과 일치할 경우

public state variable가 external함수를 override 할 수 있다. (단, 자기자신을 override 할 수 없음)

예시 코드)

````Solidity
contract A
{
    function f() external view virtual returns(uint) { return 5; }
}

contract B is A
{
    uint public override f;
}
````

#### Modifier Overriding

Function modifier는 서로 override할 수 있다. 

이는 function oveeriding이랑 동일한 방식으로 동작한다. 

- `virtual` 키워드는 override된 modifier에서만 사용되야함
- `override` 키워드는 override 한 modifier에서만 사용되야함

예시코드) 

````Solidity
pragma solidity >=0.6.0 <0.9.0;

contract Base
{
    modifier foo() virtual {_;}
}

contract Inherited is Base
{
    modifier foo() override {_;}
}
````

다중의 상속의 경우도, 지정자를 명시해주어야함

````Solidity
contract Base1
{
    modifier foo() virtual {_;}
}

contract Base2
{
    modifier foo() virtual {_;}
}

contract Inherited is Base1, Base2
{
    modifier foo() override(Base1, Base2) {_;}
}
````

#### Constructors

`Constructors`는 contract 작성시 실행되는 `생성자` 키워드로 선언된 선택적 함수, 초기화 코드 실행 가능함

`생성자`가 실행된 후 contract의 최종 코드가 블록체인에 배포됨

코드 배포에 따라 **코드** 길이의 에 선형으로 **추가 가스**가 발생함 

코드는 모든 function과 function 호출을 통해 도달할 수 있는 모든 기능을 포함함

단, `생성자` 코드 또는 `생성자`로부터 호출되는 internal 함수는 포함되지 않는다

예시코드) 

````Solidity
pragma solidity >=0.7.0 <0.9.0;

abstract contract A {
    uint public a;
     //생성자가 없으면 `constructor() {}`로 default값이 지정 
    constructor(uint _a) {
        a = _a;
    }
}

contract B is A(1) {
    constructor() {}
}
````

`생성자`내부 매개변수를 사용할 수 있고, 이 경우 매개변수는 외부로부터 유효한 값을 할당할 수 없고

derived된 contract의 `생성자`를 통해서만 할당되기 때문에 `abstract`를 표시해야한다. 

> 주의 : 0.7.0 이전에 constructors의 가시성(internal or public)을 지정해야 했다

#### Arguments for Base Constructors

아래 설명된 linearization 규칙에 따라 모든 base contract의 `생성자`를 호출한다.

인자가 있는경우, derived된 계약에서 인자가 모두 지정되어야한다. 두가지 방법으로 수행할 수 있다.

예시코드)

````Solidity
pragma solidity >=0.7.0 <0.9.0;

contract Base {
    uint x;
    constructor(uint _x) { x = _x; }
}

//1. directly specify in the inheritance list...
contract Derived1 is Base(7) { 
    constructor() {}
}

// 2. through a "modifier" of the derived constructor.
contract Derived2 is Base { 
    constructor(uint _y) Base(_y * _y) {}
}
````

1. 상속목록에 직접 명시
   인자가 상수이고, contract의 behaviour을 정의하거나 설명하는 경우 편리하다 

2. 상속목록 또는 derived된 `생성자`의 modifier를 통해 상속
   단, `생성자`인자가 derived된 contract의 인자에 의존할때 사용해야한다.

두 방법을 동시에 사용하는 경우 error

만약 derived contract가 base contract의 `생성자`에 대한 인자를 명시하지 않는다면, abstract contract일 것이다. 

#### Multiple Inheritance and Linearization

`다중상속`을 허용하는 언어는 여러문제를 처리해야한다.

문제1. 다이아몬드

 “C3 Linearization”를 사용하여 base class의 directed acyclic graph(DAG)에서 특정 순서를 강제한다는 점에서 **파이썬**과 유사하다.
 
이는 바람직한  monotonicity의 속성을 가져오지만, 일부 상속 graph는 허용하지 않는다.

특히, base class가 제공되는 순서는 중요하다.

: "most base-like"에서 "most derived"순서로 나열해야한다.

쉽게 설명하면, 서로 다른 contract에서 여러번 정의된 함수를 호출할 때, 주어진 base들을 오른쪽에서 왼쪽

으로 깊이우선방식으로 검색하여 첫번째 일치에서 멈춘다. base contract를 이미 검색한 경우 skip한다.

다음 코드는 "Linearization of inheritance graph impossible" error 발생한다.

````Solidity
pragma solidity >=0.4.0 <0.9.0;

contract X {}
contract A is X {}
//error
contract C is A, X {}
````

C가 X에게 A를 override하도록 요청하기 때문이다.

하지만 A는 X를 override하도록요청하기 때문에 해결할 수 없는 모순이다.

고유한 override 없이 여러 base에서 상속되는 함수를 명시적으로 override 해야하기 때문에 "C3 linearization" 는 그렇게 중요하지 않다.

`생성자`는 인자가 상속 contract의 `생성자`에 제공되는 순서에 관계없이 항상 linearize된 순서로 실행된다.

예시코드)

````Solidity
//  1 - Base2
//  2 - Base1
//  3 - Derived2
contract Derived2 is Base2, Base1 {
    constructor() Base2() Base1() {}
}
````


#### Inheriting Different Kinds of Members of the Same Name

상속으로 인해 contract의 다음 쌍 중 하나가 동일한 이름을 가진 경우 error

- function 과 modifier
- function 과 event
- event 와 modifier

> 예외적으로 state variable getter는 external 함수를 override할 수 있다.

### Abstract Contracts

fuction들 중 최소 하나가 구현되지 않았을 때 `abstract`를 표시할 필요가 있다.

모든 function들이 구현되더라도 `abstract`로 표시될 수 있다.

Note)

function `utterance()`가 정의되었지만 구현이 제공되지 않았기 때문에 Contract는 `abstract`로 정의되어야함

예시 코드)

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.0 <0.9.0;

abstract contract Feline {
    function utterance() public virtual returns (bytes32);
}
// 일반 클래스에서 abstract class 사용
contract Cat is Feline {
    function utterance() public pure override returns (bytes32) { return "miaow"; }
}
````

활용

- contract의 정의를 구현과 분리하여 더 나은 확장성과 자체 문서화를 제공하고 템플릿 방법과 같은 패턴을 촉진하고 코드 복제를 제거함.

### Interfaces

abstract contract와 비슷하지만, 어떤 기능을 구현할 수는 없다.

추가 제한 사항

- 다른 contract에서는 상속할 수 없지만, 다른 interface에서는 상속할 수 있다.
- 선언된 모든 함수는 external이어야 한다. 
- constructor(생성자)를 선언할 수 없다.
- state variable을 선언할 수 없다.

이런 제한 사항들 중 일부는 미래에 해제될 수 있다. 

인터페이스는 기본적으로 contract ABI가 나타낼 수 있는 것으로 제한, ABI와 인터페이스 간의 변환은 정보 손실 없이 가능해야 한다.

````Solidity
interface Token {
    enum TokenType { Fungible, NonFungible }
    struct Coin { string obverse; string reverse; }
    function transfer(address recipient, uint amount) external;
}
````

특성

- contract는 인터페이스를 상속할 수 있다. 
- interfaces로 선언된 모든 함수는 implicit하게 `virtual`이므로 재정의 할 수 있다.
- 단, 재정의 함수가 `virtual`로 표시된 경우에만 가능하다.
- interface는 다른 interface를 상속할 수 있다. 

````Solidity
interface ParentA {
    function test() external returns (uint256);
}

interface ParentB {
    function test() external returns (uint256);
}

interface SubInterface is ParentA, ParentB {
    // Must redefine test in order to assert that the parent
    // meanings are compatible.
    function test() external override(ParentA, ParentB) returns (uint256);
}
````

### Libraries

contract와 유사하지만, 특정 주소에 한번만 배포되고, 코드는 EVM의 DELEGATECALL을 사용하여 재사용됨

라이브러리 함수가 호출될 경우 **호출 contract의 context에서 해당 코드가 실행**된다는 의미임

(== 호출 contract, 특히 호출 contract의 저장소에 액세스 가능함)

- 라이브러리는 소스코드의 분리된 부분이므로 명시적으로 제공된 경우에만 호출 contract의 state variable에 access 할 수 있다.
- 라이브러리는 state를 수정하지 않는 경우(view or pure) 직접호출 할 수 있다. 
- 라이브러리를 destory하는 것은 **불가능**함

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.0 <0.9.0;


// We define a new struct datatype that will be used to
// hold its data in the calling contract.
struct Data {
    mapping(uint => bool) flags;
}

library Set {
    // Note that the first parameter is of type "storage
    // reference" and thus only its storage address and not
    // its contents is passed as part of the call.  This is a
    // special feature of library functions.  It is idiomatic
    // to call the first parameter `self`, if the function can
    // be seen as a method of that object.
    function insert(Data storage self, uint value)
        public
        returns (bool)
    {
        if (self.flags[value])
            return false; // already there
        self.flags[value] = true;
        return true;
    }

    function remove(Data storage self, uint value)
        public
        returns (bool)
    {
        if (!self.flags[value])
            return false; // not there
        self.flags[value] = false;
        return true;
    }

    function contains(Data storage self, uint value)
        public
        view
        returns (bool)
    {
        return self.flags[value];
    }
}


contract C {
    Data knownValues;

    function register(uint value) public {
        // The library functions can be called without a
        // specific instance of the library, since the
        // "instance" will be the current contract.
        require(Set.insert(knownValues, value));
    }
    // In this contract, we can also directly access knownValues.flags, if we want.
}
````

#### Function Signatures and Selectors in Libraries

#### Call Protection For Libraries

### Using For

` using A for B;`

: 라이브러리 A의 함수를 contract의 context에서 모든 타입 (B)에 연결하는데 사용

 이러한 function은 첫번째 매개변수로 호출되는 객체를 받는다. (python의 self 역할과 동일)

 현재 contract 내에서만 활성화되며, contract 외부에서는 효과가 없다.

`using A for *;`

: 라이브러리 A 어느 타입이든 연결하는데 사용

첫번째 매개변수 type이 개체의 type과 일치하지 않는 함수도 연결 가능


````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.0 <0.9.0;

struct Data { mapping(uint => bool) flags; }

library Set {
    function insert(Data storage self, uint value)
        public
        returns (bool)
    {
        if (self.flags[value])
            return false; // already there
        self.flags[value] = true;
        return true;
    }

    function remove(Data storage self, uint value)
        public
        returns (bool)
    {
        if (!self.flags[value])
            return false; // not there
        self.flags[value] = false;
        return true;
    }

    function contains(Data storage self, uint value)
        public
        view
        returns (bool)
    {
        return self.flags[value];
    }
}


contract C {
    using Set for Data; // this is the crucial change
    Data knownValues;

    function register(uint value) public {
        // Here, all variables of type Data have
        // corresponding member functions.
        // The following function call is identical to
        // `Set.insert(knownValues, value)`
        require(knownValues.insert(value));
    }
}
````


