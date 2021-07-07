---
layout: post
title: "python_codingtest"
tags: [코딩테스트, python]
---


### 한번에 여러 입력 받기

````python
a,b = map(int, input.split())
````

### 문자열 입력받기

````python
a = str(input())

print(a+"??!") # +로 문자열연결: 공백 없이 연결
print(a,"??!") # ,로 문자열연결: 자동 공백 생김
````

### 아스키코드 변환

````python
a= str(input())

print(ord(a)) # A->65

b= int(input())

print(chr(a)) # 65->A
````

### 날짜/시간 모듈 이용하기

````python
time.strftime('%Y-%m-%d', time.localtime(time.time())) #2021-07-06
time.strftime('%c, time.localtime(time.time())) #Tue Jul  6 21:03:44 2021
````
