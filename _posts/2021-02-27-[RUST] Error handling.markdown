---
layout: post
title: "[RUST] Error handling"
date: "2021-02-27 12:00:00 +0200" 
image: 12.jpg
tags: [rust, error]
categories: rust
---

Error Handling은 실패의 가능성을 처리하는 방법임
에러들을 주의하고 명확하게 관리해야 다양한 에러상황에서 프로그램의 나머지를 보호할 수 있음

### 회복 불가능한 오류
1. painc 
회복 불가능한 오류를 만났을때 어떤 오류가 발생했는지 띄어주고 종료함
따로 `panic` 매크로를 설정하지 않으면, 그동안 쌓인 스택을 풀어주고 종료

*스택풀기 : 스택 메모리에 쌓여있는 함수들의 자료를 처리하는 작업

`panic!` 매크로는 인자로 문자열을 받음
`panic!` 매크로가 호출되면 인자로 받은 문자열이 출력되고, 어디에서 호출되었는지 알려줌

라이브러리가 내부적으로 `painc!`을 호출하는 경우도 있음
 예)배열의 범위를 벗어난 원소를 접근하고자 하면, 프로그램 종료 후 오류 메시지 출력함

```rust
fn give_princess(gift: &str) {
    if gift == "snake" { panic!("AAAaaaaa!!!!"); }

    println!("I love {}s!!!!!", gift);
}

fn main() {
    give_princess("teddy bear");
    give_princess("snake");
}
```

*backtrace (역추적)
`panic!`함수가 호출되면 프로그램 종료되고 오류메시지와 오류 지점이 출력된 후 다음과 같은 문장이 출력됨
![image](https://user-images.githubusercontent.com/44187194/109315962-734c4700-788e-11eb-8230-478e7e7b0150.png)
`cargo run RUST_BACKTRACE=full`을 작성하면 `panic!`이 발생했을 때 호출과정을 역추적 할 수 있음
기본적으로 디버그 모드로 수행되기 때문에 해당 동작이 가능함

<출력화면>
![image](https://user-images.githubusercontent.com/44187194/109375800-e42d4680-7902-11eb-8ac0-6c01e3859892.png)

2. Option & unwrap
이전 코드에서 아무런 선물(문자열)을 받지 못할 경우도 나쁜 경우가 되도록 처리할 필요가 있음
`std`라이브러리에 있는 `Option<T>`로 불리는 enum이 부재가 가능할 때 사용됨

- Some(T) : T타입의 요소가 발견됐을 댸
- None : 요소가 없을때

이 경우는 `match`를 통해 명시적으로 처리되거나 `unwrap`으로 암시적으로 처리됨
암시적 처리는 내부요소를 반환하거나 `panic`을 반환함

```rust
fn give_commoner(gift: Option<&str>) {
    // 각 경우에 따라 처리 과정을 지시
    match gift {
        Some("snake") => println!("snake"),
        Some(inner)   => println!("{}", inner),
        None          => println!("No gift"),
    }
}

fn give_princess(gift: Option<&str>) {
    // unwrap은 'None'을 받으면 'panic'을 반환
    let input = gift.unwrap();
    if input == "snake" { panic!("AAAaaaaa!!!!"); }

    println!("I love {}s!!!!!", input);
}

fn main() {
    let bird= Some("robin");
    let nothing = None;

    give_commoner(bird);
    give_commoner(nothing);

    give_princess(bird);
    give_princess(nothing);

}
```

![image](https://user-images.githubusercontent.com/44187194/109378256-7a1d9d00-7914-11eb-9b14-8b6204636416.png)

### 회복 가능한 오류
회복 가능한 오류를 만나면 굳이 프로그램 종료 시킬 필요 없음
오류에 대한 적절한 처리를 해주는 것으로 충분함
예) 어떤 파일을 열고자 할때 해당 파일이 없다면, 해당 이름을 가진 파일을 새로 생성하는 것이 합리적임
  
우리는 `Result<T, E>` 열거형을 통해 회복 가능한 오류를 처리할 수 있음
3. Result
변수형 종류 
- `Ok(value)` 가 나타내는 것은 동작이 성공했다는 것, 연산이 반환하는 value를 포장  
- `Err(why)` 가 나타내는 것은 동작이 실패했다는 것, 실패의 원인을 설명하는 why를 포장

> `match`표현식과 같은 방법으로 처리

1) Result 를 사용해 파일 열기 
![image](https://user-images.githubusercontent.com/44187194/109378593-3aa48000-7917-11eb-9634-99595cf8fb37.png)


결과 : hello.txt 파일 생성

2) 중첩보다 간단한 방법으로 파일 생성
![image](https://user-images.githubusercontent.com/44187194/109378645-aa1a6f80-7917-11eb-99fd-5204b23b574b.png)

> `unwrap()`을 사용하면 `Ok` 열거값일 경우 저장된 값 리턴 `Err`열거값인 경우 `panic!` 매크로 호출함
> `except("에러메시지")`을 사용하면 오류 메시지 지정 가능 
     => 개발자의 의도를 더 명확하게 표현하는 동시에 `panic`이 발생한 원인을 더 쉽게 추적 가능함

4. Propagation(전파)
오류가 발생했을 때 그 자리에서 처리해주지 않고 
함수를 호출한 쪽에서 처리하도록 넘겨줄 필요가 있음

이런 방법으로 라이브러리 함수를 만들었을 때
발생할 수 있는 오류에 대한 처리를 우리가 미리 구현 하는 것이아니라
더 많은 정보를 가지고 있는 함수를 호출한 쪽에서 더 효과적으로 처리할 수 있도록 함

구현 방법
- Result 열거값을 반환
- 작업의 성공 여부에 따라 `Ok` 또는 `Err`를 직접반환

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn main() {
    match read_username_from_file() {
        Ok(name) => println!("username: {}", name),
        Err(err) => println!("ERROR: {:?}", err),
    }
}

//result 열거값 반환
//T는 String type, E는 Error 타입
fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        //file open을 실패해서 Error가 발생했을 때, 해당 함수를 중단하고, 
        //리턴한 에러값을 함수의 에러값으로 호출자에게 리턴
        Err(e) => return Err(e),
    };

    let mut s = String::new();
    //read_to_string : file 전체 내용을 string으로 반환하는 역할
    //읽는 과정에서 에러가 발생할 수 있음 -> result  사용
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<결과 화면>
파일이 존재할 경우 
![image](https://user-images.githubusercontent.com/44187194/109380924-b653fc00-791a-11eb-87f3-8d9b46e8cf8b.png)

오류가 발생하는 지점마다 매번 match 표현을 사용하면 가독성이 떨어짐
러스트에서는 위와 같이 에러를 전파하는 것이 일반적이므로 `?` 연산자를 제공

오류가 발생할 수 있는 코드 뒤에 `?`를 붙이면
오류가 발생할 경우 해당 오류가 전파되고, 발생하지 않을 경우 계속 진행함

> `?`은 반환타입이 `Result` 일때만 사용가능함

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn main() {
    match read_username_from_file() {
        Ok(name) => println!("username: {}", name),
        Err(err) => println!("ERROR: {:?}", err),
    }
}

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();

    f.read_to_string(&mut s)?;

    Ok(s)
}
```

`?` 이후에 호출을 이어갈 수 있는데 
작업을 성공한 경우만 다음 메서드가 호출, 그렇지 않은 경우 오류가 전파됨

```rust
// 중복 부분 생략

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    //한 줄로 이어서 코드 작성 가능!
    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}

```

간결하고 가독성 높으면서 오류에 대한 처리가 된 코드가 됨
우리가 작성한 코드를 `fs`의 메서드를 사용하면 더 쉽게 작성할 수 있음

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

- main함수가 리턴할 수 있는 타입중 하나는 ()이며, Result<T,E> 타입을 리턴할 수 있음
- Box<dyn Error> 타입은 trait 객체라고 부르는 타입, '모든 종류의 에러'를 의미함


### 정리
언제 `panic!`을 호출할지 `Result<T,E>`를 반환할지는 개발자의 판단임

`panic!`은 복구할수 없고, 유효하지 않거나 잘못된 값으로 시도하는 실행을 멈추게끔 해줌
`Result<T,E>`는 코드를 호출하는 코드에게 옵션을 제공함, 결과를 반환할 경우 필요에 따라 `panic!`할 수 있음

무엇을 사용할지 판단이 잘 안선다면 `Result<T,E>`를 반환하는 함수를 사용하는 것이 좋음

프로토타입이나 테스트 코드를 작성할때는 즉각적인 `panic!`을 사용하는 것이 좋음
