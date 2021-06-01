---
layout: post
title: "[Solidity] Solidity by Example"
date: "2021-06-01 12:00:00 +0200" 
image: 3.jpg
tags: [solidity, expressions, control]
categories: solidity
---

참고자료 : https://docs.soliditylang.org/en/v0.8.4/solidity-by-example.html

# Solidity by Example

## Voting

- 투표 contract는 복잡하지만 Solidity의 특징을 많이 보여준다.
- 전자 투표의 주요 문제는 올바른 사람에게 투표권을 어떻게 배분하느냐와 **조작을 막는 방법**이다.
- 모든문제를 해결하지는 않지만, 개표가 자동적이고 완전히 투명하게 동시에 이뤄질 수 있도록 위임투표가 어떻게 이뤄질 수 있는지 볼 수 있다.

- 투표용지마다 계약서를 하나씩 만들어 옵션별로 짧은 이름을 붙이자는 아이디어이다.
- 의장 역할을 하는 contract 작성자가 각각의 주소에 개별적으로 투표권을 주게된다.
- 사람들은 스스로 투표하거나 신뢰하는 사람에게 투표하도록 선택할 수 있다.

- 투표의 마지막에, `winningProposal()`는 최다 득표자가 반환된다.



### Possible Improvements

## Blind Auction

### Simple Open Auction

### Blind Auction

## Safe Remote Purchase

## Micropayment Channel

### Creating and verifying signatures

### Writing a Simple Payment Channel

## Modular Contracts
