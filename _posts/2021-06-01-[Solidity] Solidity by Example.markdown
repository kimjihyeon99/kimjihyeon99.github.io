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

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;
/// @title Voting with delegation.
contract Ballot {
    //한명의 투표자를 나타냄
    struct Voter {
        uint weight; // weight is accumulated by delegation
        bool voted;  // if true, that person already voted
        address delegate; // person delegated to
        uint vote;   // index of the voted proposal
    }

    //하나의 Proposal(후보자)에 대한 type
    struct Proposal {
        bytes32 name;   // short name (up to 32 bytes)
        uint voteCount; // number of accumulated votes
    }

    address public chairperson;

    // 각각의 가능한 주소에  `Voter` struct를 저장하는  state variable을 선언한다.
    mapping(address => Voter) public voters;

    // `Proposal` structs의 동적 사이즈 배열 
    Proposal[] public proposals;

    /// 새 투표용지를 만들어 'proposalNames' 중 하나를 선택한다.
    constructor(bytes32[] memory proposalNames) {
        chairperson = msg.sender;
        voters[chairperson].weight = 1;

        // 각각의 proposal names에 대해서,
        // 새로운 'Proposal'를 만들고, 배열 끝에 추가한다.
        for (uint i = 0; i < proposalNames.length; i++) {
            proposals.push(Proposal({
                name: proposalNames[i],
                voteCount: 0
            }));
        }
    }

   
    // 'voter'에게 이 투표권을 준다.
    // `chairperson`만 이 함수를 호출 할 수 있다. 
    function giveRightToVote(address voter) public {
        // 'require'의 첫번째 인자로 'false'로 되면 실행이 종료되고, 상태 및 Ether 잔액에 대한 모든 변경사항은 revert 된다. 
        // 예전 EVM에서는 모든 가스를 소비했지만, 이제는 그렇지 않다. 
        // 'require'로 무엇이 잘못되었는지 설명을 추가할 수 있다. 
        
        require(
            msg.sender == chairperson, //sender가 chairperson이 아니면 revert!!
            "Only chairperson can give right to vote."
        );
        require(
            !voters[voter].voted, //이미 투표한 투표자라면 revert!!
            "The voter already voted."
        );
        require(voters[voter].weight == 0);
        voters[voter].weight = 1;
    }

    /// 투표권을 'to'에게 위임한다. 
    function delegate(address to) public {
        // assigns reference
        Voter storage sender = voters[msg.sender];
        require(!sender.voted, "You already voted."); //이미 투표한 투표자라면 revert!!

        require(to != msg.sender, "Self-delegation is disallowed.");

        // 'to'도 위임된 경우, 위임한다. 
        // 일반적으로 이 루프는 위험하다. ??
        while (voters[to].delegate != address(0)) {
            to = voters[to].delegate;

            // We found a loop in the delegation, not allowed.
            require(to != msg.sender, "Found loop in delegation.");
        }

        // 'sender'는 참조이므로, 'voters[msg.sender].voted' 를 수정한다. 
        sender.voted = true;
        sender.delegate = to;
        Voter storage delegate_ = voters[to];
        if (delegate_.voted) {
            // 위임자가 이미 투표한 경우 , 'voteCount'에 직접 추가
            proposals[delegate_.vote].voteCount += sender.weight;
        } else {
            // 위임자가 투표하지 않은 경우, 대표자의 weight 직접 추가 
            delegate_.weight += sender.weight;
        }
    }

    /// 위임 받은 투표를 포함한 투표를 후보자에게 투표하는 함수 
    /// 후보자 : 'proposals[proposal].name' 
    function vote(uint proposal) public {
        Voter storage sender = voters[msg.sender];
        require(sender.weight != 0, "Has no right to vote");
        require(!sender.voted, "Already voted.");
        sender.voted = true;
        sender.vote = proposal;

        // `proposal`이 배열  범위에 벗어나면, 자동으로 throw되고, 모든 변경사항이 revert 된다. 
        proposals[proposal].voteCount += sender.weight;
    }

    /// account에 이전의 모든 표를 고려하여 winning proposal를 계산하는 함수
    function winningProposal() public view
            returns (uint winningProposal_)
    {
        uint winningVoteCount = 0;
        for (uint p = 0; p < proposals.length; p++) {
            if (proposals[p].voteCount > winningVoteCount) {
                winningVoteCount = proposals[p].voteCount;
                winningProposal_ = p;
            }
        }
    }

    // proposals array 에 포함된 winner의 인덱스를 얻는 함수를 호출하고,
    // winner의 이름을 반환한다. 
    function winnerName() public view
            returns (bytes32 winnerName_)
    {
        winnerName_ = proposals[winningProposal()].name;
    }
}
````

### Possible Improvements

## Blind Auction

이더리움에서 완전 블라인드 경매 contract을 만드는 것은 쉽다. 

누구나 입찰을 볼 수 있는 **공개 경매**, 입찰기간이 끝날 때까지 실제 입찰을 볼 수 없는 **블라인드 경매**가 있다. 

### Simple Open Auction

일반적인 생각은 입찰기간 동안 누구나 입찰을 보낼 수 있다는 것이다.

입찰에는 입찰자들을 입찰에 묶기 위해 돈을 보내는 것이 이미 포함되어있다. 

만약 최고 입찰각가 인상되면, 이전의 최고 낙찰자는 그들의 돈을 돌려받는다.

입찰기간이 종료된 후 수익자가 돈을 받으려면 수동으로 계약을 호출해야 하므로 계약이 스스로 활성화 될 수 없다. 

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;
contract SimpleAuction {
    // 경매의 파라미터 
    // Time은 absolute unix timestamps 또는 시간간격(초)을 나타냄 
    address payable public beneficiary;
    uint public auctionEndTime;

    // 경매의 현재 상태
    address public highestBidder;
    uint public highestBid;

    // Allowed withdrawals of previous bids
    // 이전 입찰의 철회 허용
    mapping(address => uint) pendingReturns;

    // 마지막에 'true'로 설정되면, 변경이 허용되지 않음
    // default 값 false
    bool ended;

    // 변경시에 전송될 event이다. 
    event HighestBidIncreased(address bidder, uint amount);
    event AuctionEnded(address winner, uint amount);

    // 실패를 설명하는 Errors

    // 슬래시 세개로 표시되어있는 것들은  트랜잭션 확인을 요청하거나 오류가 표시될 때 표시된다.
    
    /// The auction has already ended.
    error AuctionAlreadyEnded();
    /// There is already a higher or equal bid.
    error BidNotHighEnough(uint highestBid);
    /// The auction has not ended yet.
    error AuctionNotYetEnded();
    /// The function auctionEnd has already been called.
    error AuctionEndAlreadyCalled();

    /// 낙찰자 주소 '_beneficiary'를 대신하여 `_biddingTime`초 입찰 시간을 가진 간단한 auction을 생성한다. 
    constructor(
        uint _biddingTime,
        address payable _beneficiary
    ) {
        beneficiary = _beneficiary;
        auctionEndTime = block.timestamp + _biddingTime;
    }
    
    // transaction과 같이 보낸 값과 경매에 입찰함
    // 경매에 낙찰되지 않은 경우, 그 값은 환불된다.  
    function bid() public payable {
        // No arguments are necessary, all
        // information is already part of
        // the transaction. The keyword payable
        // is required for the function to
        // be able to receive Ether.

        // 입찰 기간을 넘어서면 revert를 호출한다. 
        if (block.timestamp > auctionEndTime)
            revert AuctionAlreadyEnded();

        // 입찰가격이 낮아 낙찰되지 않은 경우 revert를 호출한다. 
        if (msg.value <= highestBid)
            revert BidNotHighEnough(highestBid);

        if (highestBid != 0) {
            // 단순히 'highestBidder.send(highestBid)'를 사용해 돈을 돌려 보내는 것은 보안상 위험이 있다. 
            // 수령인이 직접 돈을 인출하도록 하는 것이 항상 안전하다. 
            pendingReturns[highestBidder] += highestBid;
        }
        highestBidder = msg.sender;
        highestBid = msg.value;
        emit HighestBidIncreased(msg.sender, msg.value);
    }

    /// 너무 많이 입찰된 경우 철회 하는 함수
    function withdraw() public returns (bool) {
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {    
            // 수신자가 'send'를 반환하기 전에 receiving call의 일부로 이 함수를 다시호출 할 수 있기때문에 0으로 설정하는 것이 중요하다.
            pendingReturns[msg.sender] = 0;

            if (!payable(msg.sender).send(amount)) {
                // 여기서 throw할 필요없이 아래금액만 재설정하면 된다.
                pendingReturns[msg.sender] = amount;
                return false;
            }
        }
        return true;
    }

    // 경매를 끝내고 최고 낙찰가를을 낙찰자에게 보낸다. 
    function auctionEnd() public {
        // 다른 contract와 interact하는 function을 다음 3단계로 구조화하는 것은 좋은 가이드라인임
        // 1. 조건 체크하기 
        // 2. 경매 수행하기
        // 3. 다른 contract와 거래
        
        // 이 단계가 섞이면, 다른 contract에서는 현재 contract를 다시 체결하여 여러번 수행될 상태나 결과를 수정할 수 있다.  
        // 내부적으로 호출되는 함수가 외부 contract와 상호작용 하는경우 고려해보아야한다. 

        // 1. Conditions
        if (block.timestamp < auctionEndTime)
            revert AuctionNotYetEnded();
        if (ended)
            revert AuctionEndAlreadyCalled();

        // 2. Effects
        ended = true;
        emit AuctionEnded(highestBidder, highestBid);

        // 3. Interaction
        beneficiary.transfer(highestBid);
    }
}
````

### Blind Auction

공개 경매는 블라인드 경매로 확장된다.

블라인드 경매의 장점은 입찰기간이 끝날 때 까지 시간 압박이 없다는 점이다.

투명한 컴퓨팅 플랫폼에 블라인드 경매를 만드는 것은 모순처럼 들릴 수 있지만 암호화가 구조를 가져다 준다.


- 입찰 기간 동안, 입찰자는 실제로 입찰을 보내지 않고 단지 그것의 해시된 버전만 보낸다.
- 현재 해시 값이 동일한 두 개의 값을 찾는 것은 현실적으로 불가능 하다고 간주되기 때문에, 입찰자는 그것에 의해 입찰에 응한다.
- 입찰 기간이 끝나면 입찰자는 입찰 내용을 공개해야 한다.
- 암호화되지 않은 값을 전송하고 계약은 해시 값이 입찰 기간 동안 제공된 값과 동일한지 확인한다. 


- 경매의 구속력과 블라인드를 동시에 만드는 방법이 또 다른 과제이다.
- 경매에서 낙찰된 후 입찰자가 돈을 보내지 않는 것을 막을 수 있는 유일한 방법은 **입찰과 함께 돈을 보내도록 하는 것**이다. 
- 이더리움에서는 value transfers을 블라인드 할 수 없기 때문에 **누구나 가치를 볼 수 있다**. 


- 다음 예는 **최고 입찰액보다 큰 값을 수락함**으로써 이문제를 해결한다. 
- 공개 단계에서만 확인할 수 있기 때문에 일부 입찰은 무효일 수 있으며, 이는 고의적인것이다. 
- 입찰자들은 여러개의 높은 혹은 낮은 무효 입찰을 함으로써 경쟁을 혼란스럽게 할 수 있다.

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;
contract BlindAuction {
    struct Bid {
        bytes32 blindedBid;
        uint deposit;
    }

    address payable public beneficiary;
    uint public biddingEnd;
    uint public revealEnd;
    bool public ended;

    mapping(address => Bid[]) public bids;

    address public highestBidder;
    uint public highestBid;

    // Allowed withdrawals of previous bids
    mapping(address => uint) pendingReturns;

    event AuctionEnded(address winner, uint highestBid);

    // Errors that describe failures.

    /// 함수가 너무 일찍 호출된경우 'time'이후에 다시시도
    error TooEarly(uint time);
    ///  함수가 너무 늦게 호출된경우 'time'이후에 호출되지 못함
    error TooLate(uint time);
    /// 'auctionEnd' 함수가 이미 불린 경우
    error AuctionEndAlreadyCalled();

    // Modifiers는 입력을 검증하는 편리한 방법임!
    // 새로운 함수의 body는 old body에 의해 대체된 '_'가 있는 modifier의 body이다.
    modifier onlyBefore(uint _time) {
        if (block.timestamp >= _time) revert TooLate(_time);
        _;
    }
    modifier onlyAfter(uint _time) {
        if (block.timestamp <= _time) revert TooEarly(_time);
        _;
    }
    
    constructor(
        uint _biddingTime,
        uint _revealTime,
        address payable _beneficiary
    ) {
        beneficiary = _beneficiary;
        biddingEnd = block.timestamp + _biddingTime;
        revealEnd = biddingEnd + _revealTime;
    }

    /// `_blindedBid` = keccak256(abi.encodePacked(value, fake, secret)) 로 블라인드 입찰로 둔다.
    // 입찰이 the revealing phase에서 정확히 드러난 경우에만 send ether가 환불된다. 
    // 입찰과 함께 보낸 ether가 적어도 "value"이고 "false"가 참이 아닐 경우 입찰은 유효하다.
    // "fake"를 참으로 설정하고 정확한 금액을 보내지 않은 경우, 실제 입찰을 숨기면서도 필요한 예금을 하는 방식이다. 
    // 동일한 주소로 다수의 입찰로 둔다. 
    function bid(bytes32 _blindedBid)
        public
        payable
        onlyBefore(biddingEnd)
    {
        bids[msg.sender].push(Bid({
            blindedBid: _blindedBid,
            deposit: msg.value
        }));
    }

    // blind 된 나의 입찰을 드러내는 함수
    // 올바르게 blind 처리된 모든 무효 입찰과 가장 높은 입찰을 제외한 모든 입찰에 대해 환불된다. 
    function reveal(
        uint[] memory _values,
        bool[] memory _fake,
        bytes32[] memory _secret
    )
        public
        onlyAfter(biddingEnd)
        onlyBefore(revealEnd)
    {
        uint length = bids[msg.sender].length;
        require(_values.length == length);
        require(_fake.length == length);
        require(_secret.length == length);

        uint refund;
        for (uint i = 0; i < length; i++) {
            Bid storage bidToCheck = bids[msg.sender][i];
            (uint value, bool fake, bytes32 secret) =
                    (_values[i], _fake[i], _secret[i]);
            if (bidToCheck.blindedBid != keccak256(abi.encodePacked(value, fake, secret))) {
                // 입찰은 실제로 밝혀지지 않은 경우, 예금을 환불하지 않는다. 
                continue;
            }
            refund += bidToCheck.deposit;
            if (!fake && bidToCheck.deposit >= value) {
                if (placeBid(msg.sender, value))
                    refund -= value;
            }
            // sender가 동일한 보증금을 재청구 할 수 없도로 한다.
            bidToCheck.blindedBid = bytes32(0);
        }
        payable(msg.sender).transfer(refund);
    }

    /// Withdraw a bid that was overbid.
    function withdraw() public {
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {
            // 0으로 설정하는게 중요함(위 설명과 중복)
            pendingReturns[msg.sender] = 0;

            payable(msg.sender).transfer(amount);
        }
    }

    // 설명 중복
    function auctionEnd()
        public
        onlyAfter(revealEnd)
    {
        if (ended) revert AuctionEndAlreadyCalled();
        emit AuctionEnded(highestBidder, highestBid);
        ended = true;
        beneficiary.transfer(highestBid);
    }

    // "internal" 함수 
    function placeBid(address bidder, uint value) internal
            returns (bool success)
    {
        if (value <= highestBid) {
            return false;
        }
        if (highestBidder != address(0)) {
            // 이전에 가장 높은 입찰자에게 환불한다. 
            pendingReturns[highestBidder] += highestBid;
        }
        highestBid = value;
        highestBidder = bidder;
        return true;
    }
}
````

## Safe Remote Purchase

원격으로 상품을 구매하려면 서로를 신뢰해야 하는 여러 당사자가 필요하다.

가장 간단한 구성은 seller와 buyer를 포함한다. 

buyer는 seller로부터 아이템을 받고자 하며 seller는 그 대가로 돈을 받고자 한다.

여기서 문제가 되는 부분은 배송이다. 물건이 buyer에게 도착했는지 확인할 방법이 없다. 


이 문제를 해결할 수 있는방법

- 다음 예에서, buyer와 seller는 contract에서 조건부 날인 증서 항목 가치의 두배를 contract에 넣어야한다. 
- 그 후, buyer가 물건을 받았다는 것을 확인할 때까지 돈은 계약서 안에 lock 되어 있을 것이다. 
- 그 후, buyer는 예금의 절반(예금/2)을 돌려받게 되고, seller는 예금 절반의 3배(예금+가치)를 얻게 된다. 
- 이면의 idea는 buyer와 seller가 상황을 해결할 동기를 가지고 있거나 그렇지 않으면 그들의 돈이 영원히 lock되어있다는 것이다.

이 contract는 문제를 해결하지는 않지만, contract내에서  state machine과 같은 구조를 사용할 수 있는 방법에 대한 계요를 제공한다. 


````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;
contract Purchase {
    uint public value;
    address payable public seller;
    address payable public buyer;

    enum State { Created, Locked, Release, Inactive }
    // The state variable has a default value of the first member, `State.created`
    State public state;

    modifier condition(bool _condition) {
        require(_condition);
        _;
    }

    /// Only the buyer can call this function.
    error OnlyBuyer();
    /// Only the seller can call this function.
    error OnlySeller();
    /// 현재 state에서 함수가 호출될 수 없는 경우
    error InvalidState();
    /// 제공된 value가 짝수가 되야하는 경우
    error ValueNotEven();

    modifier onlyBuyer() {
        if (msg.sender != buyer)
            revert OnlyBuyer();
        _;
    }

    modifier onlySeller() {
        if (msg.sender != seller)
            revert OnlySeller();
        _;
    }

    modifier inState(State _state) {
        if (state != _state)
            revert InvalidState();
        _;
    }

    event Aborted();
    event PurchaseConfirmed();
    event ItemReceived();
    event SellerRefunded();

    // `msg.value`가 짝수인지 확인한다. 
    // 홀수 인경우 나머지를 자른다. 
    // 곱셈을 통해 홀수가 아니었는지 확인한다. 홀수인 경우, error
    constructor() payable {
        seller = payable(msg.sender);
        value = msg.value / 2;
        if ((2 * value) != msg.value)
            revert ValueNotEven();
    }


    /// 구매를 abort(중단)하고 ether를 회수한다. 
    /// contract가 lock되기 전에 seller에 의해서만 호출 가능하다.   
    function abort()
        public
        onlySeller
        inState(State.Created)
    {
        emit Aborted();
        state = State.Inactive;
        // transfer를 직접 사용한다. 
        // 이 함수에서 마지막 호출이고, 이미 state를 변경했기 때문에reentrancy-safe 하다.
        seller.transfer(address(this).balance);
    }

    /// buyer로서 구매를 확인한다. 
    /// transaction에는 '2 * value' ether가 포함되어있어야하고, ether는 'confirmReceived'가 호출될때가 지 lock되어 있을것이다.  
    function confirmPurchase()
        public
        inState(State.Created)
        condition(msg.value == (2 * value))
        payable
    {
        emit PurchaseConfirmed();
        buyer = payable(msg.sender);
        state = State.Locked; 
    }
    
    /// 아이템을 받았는지 buyer가 확인한다. 
    /// lock 되어있는 ether가 해제될 것이다. 
    function confirmReceived()
        public
        onlyBuyer
        inState(State.Locked)
    {
        emit ItemReceived();
        // state를 첫번째로 바꾸는 것은 중요하다. 
        // 그러지 않으면 contract는 아래의 'send'를 사용해서 호출한 contract가 여기서 다시 호출 될 수 있다. 
        state = State.Release;

        buyer.transfer(value);
    }

    /// seller에게 환불하는 함수이다. == 판매자의 lock된 자금을 환불함
    function refundSeller()
        public
        onlySeller
        inState(State.Release)
    {
        emit SellerRefunded();
        // state를 첫번째로 바꾸는 것이 중요하다. 
        // 그러지 않으면, contract는 아래의 'send'를 사용해서 호출한 contract가 여기서 다시 호출 될 
        state = State.Inactive;

        seller.transfer(3 * value);
    }
}
````

## Micropayment Channel

결제 채널의 예시 구현을 구축하는 방법에 대해 알아보자.

암호화 서명을 사용하여 동일한 당사자 갇에 보안적이고 즉각적인 트랜잭션 수수료 없이 Ether를 반복적으로 전송한다. 

예를들어, 서명 및 확인 방법을 이해하고 결제 채널을 설정해야한다. 


### Creating and verifying signatures

A는 일정양의 Ether를 B에게 보내려고 한다. **A : 보낸 사람 B: 받는 사람**

A는 암호로 서명된 메시지 off-chain(예: 이메일)을 B에게 보내고, 이는 수표 작성과 비슷하다. 

A와 B는 서명을 이용해 거래를 승인하는데, 이는 이더리움에서 smart contract로 가능하다.

A는 Ether를 전송할 수 있는 간단한 smart contract를 만들지만, 결제를 시작하기 위해 직접 함수를 호출하는 대신  B가 이를 수행하도록 하여 거래 수수료를 지불 한다. 

contract 작동 과정

1. A는 `ReceiverPays` contract를 배포하고, 지불을 처리할 수 있는 충분한 Ether를 첨부한다.
2. A는 개인 키로 메시지에 서명함으로 써 지불을 승인하낟. 
3. A는 암호로 서명된 메시지를 B에게 보낸다. 
4. 메시지는 비밀로 유지할 필요가 없으므로, 메시지를 보내는 메커니즘에는 문제가 없다. 
5. B는 서명된 메시지를 smart contract에 제시하여 지불을 청구하고, 메시지의 진위를 확인한 다음 자금을 방출한다? 해제한다?. 

// 그림 만들기?

#### Creating the signature

A는 거래를 서명하기 위해 이더리움 네트워크와 상호 작용 할 필요가 없고, 프로세스는 완전히 offline 상태이다. 

이 튜토리얼에서는 다른 많은 보안 이점을 제공하므로 **EIP-762**에 설명된 방법을 사용하여 web3.js 및 메타마스크를 사용하여 브라우저에 메시지를 서명한다.  

````Solidity
/// Hashing first makes things easier
var hash = web3.utils.sha3("message to sign");
web3.eth.personal.sign(hash, web3.eth.defaultAccount, function () { console.log("Signed"); });
````

> ` web3.eth.personal.sign`는 서명된 데이터에 메시지 길이를 더한다. 먼저 hash를 하기 때문에 메시지는 항상 32byte 길이이며, 따라서 이 길이 접두사는 항상 동일하다.


#### What to Sign

지불을 이행하는 contract의 경우, 서명된 메시지는 다음을 반드시 포함해야한다.
    
    1. 받는 사람의 주소
    2. 전송되는 양
    3. `replay attack(?)`에 대한 보호

`replay attack` 

- 서명된 메시지를 재사용하여 두번째 액션에 대한 인증을 청구하는 경우이다. 
- 이를 피하기 위해 우리는 이더리움 트랜잭션 자체에서와 동일한 기술을 사용한다. -> `nonce`
- `nonce`는 account가 전송하는 트랜잭션 수이다. smart contract에서는 `nonce`가 여러번 사용되는지 여부를 확인한다. 


- 소유자가 `ReceiverPays` smart contract를 배포하고, 일부 결제를 한 다음, contract를 파기할때 다른 유형의 `replay attack`이 발생할 수 있다. 
- 나중에 `RecipientPays` smart contract를 다시 배포하기로 결정하지만, 새 contract는 이전 배포에서 사용된 `nonce`를 알지 못하므로 공격자기 이전 메시지를 다시 사용할 수 있다. 


- A는 메시지에 contract’s address를 포함 시킴으로써 이 공격에 대해 보호할 수 있고, contract 자체의 주소를 포함 하는 메시지만 수락된다. 
- 마지막에 나오는 전체 contract의 `claimPayment()` 함수의 처음 두 줄에서 이런 예를 찾을 수 있다. 


#### Packing arguments

이제 서명된 메시지에 포함할 정보를 식별했으므로, 메시지를 함께 넣고 hash한 후 서명할 준비가 되었다.

**단순성**을 위해 **데이터을 연결**한다.  

- `ethereumjs-abi`library는 `soliditySHA3`라고 불리는 함수를 제공한다. 
- `soliditySHA3` 는 `abi.encodePacked`에 사용하는 인코딩된 인자에 적용된 Solidity’s `keccak256` 함수를 따라한다. 
- 다음 예는 `ReceiverPays`를 위해 적절한 서명을 만드는 js 함수 이다. 


````Solidity
// `recipient`는 지불되어야하는 주소이다. 
// `amount`(wei)는 얼마의 ether가 보내져야하는지 구체화한다.
// `nonce`는 replay attacks을 막기 위한 unique한 숫자가 될 수 있다. 
// `contractAddress`는 cross-contract replay attacks를 막기위해 사용된다. 
function signPayment(recipient, amount, nonce, contractAddress, callback) {
    var hash = "0x" + abi.soliditySHA3(
        ["address", "uint256", "uint256", "address"],
        [recipient, amount, nonce, contractAddress]
    ).toString("hex");

    web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
}
````

#### Recovering ther Message Signer in Solidity

일반적으로, ECDSA(?) signature는 두개의 인자 `r`, `s`로 구성되어있다. 

이더리움에서 Signature는  메시지 서명에 사용된 계정의 개인 키를 확인하는 데 사용할 수 있는 3번째 매개변수 `v`와  트랜잭션의 sender가 포함된다. 

Solidity 는 메시`r`,`s`인자와 `v`인자와 함께 메시지를 수신하고 메시지 서명에 사용된 주소를 반환하는  built-in function `ecrecover`를 제공한다. 


#### Extracting the Signature Parameters

web3.js에의해 생성된 서명은 `r`,`s`,`v`의 결합이므로 첫 번째 단계는 이러한 매개 변수를 분리하는 것이다.

클라이언트 측에서 할 수 있지만, smart contract 내에서 할 경우 서명 매개변수를 3개만 보내면 된다. 

바이트 배열을 그것의 구성 부분으로 나누는 것은 엉망이기 때문에, `splitSignature` 함수를 사용하기 위해  `inline assembly`를 사용한다. 

#### Computing the Message Hash

smart contract는 **어떤 매개 변수가 서명되었는지** 정확히 알아야 하므로 매개 변수에서 메시지를 재생성하여 서명 확인을 위해 사용해야한다. 

`claimPayment`함수에서  `prefixed`, `recoverSigner` 함수를 사용한다. 

#### The full contract

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;
contract ReceiverPays {
    address owner = msg.sender;

    mapping(uint256 => bool) usedNonces;

    constructor() payable {}

    function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) public {
        require(!usedNonces[nonce]);
        usedNonces[nonce] = true;

        // 클라이언트에서 서명된 메시지를 다시 작성한다. 
        bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

        require(recoverSigner(message, signature) == owner);

        payable(msg.sender).transfer(amount);
    }

    /// contract를 바기하고, 남은 자금을 회수한다. 
    function shutdown() public {
        require(msg.sender == owner);
        selfdestruct(payable(msg.sender));
    }

    /// signature 메소드.
    function splitSignature(bytes memory sig)
        internal
        pure
        returns (uint8 v, bytes32 r, bytes32 s)
    {
        require(sig.length == 65);

        assembly {
            // first 32 bytes, after the length prefix.
            r := mload(add(sig, 32))
            // second 32 bytes.
            s := mload(add(sig, 64))
            // final byte (first byte of the next 32 bytes).
            v := byte(0, mload(add(sig, 96)))
        }

        return (v, r, s);
    }

    function recoverSigner(bytes32 message, bytes memory sig)
        internal
        pure
        returns (address)
    {
        (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

        return ecrecover(message, v, r, s);
    }

    //// eth_sign의 동작을 모방하는 prefixed hash를 빌드한다.  
    function prefixed(bytes32 hash) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
    }
}
````

### Writing a Simple Payment Channel

A는 이제 간편하지만 완전한 결제 채널 구현을 만든다. 

결제 채널은 암호화 서명을 사용하여 거래 수수료 없이 즉각적으로 안전하게 Ether를 반복적으로 전송한다. 

#### What is a Payment Channel?

결제 채널의 참여자들이 트랜잭션을 사용하지 않고도 Ether를 반복적으로 전송할 수 있도록 한다. 

즉, 트랜잭션과 관련된 지연 및 수수료를 피할 수 있다. 

A와 B 사이의 간단한 단방향 결제 채널을 알아 볼 것 이다. 

세가지 단계가 포함된다

1. A는 Ether와 smart contract를 체결한다. 그럼 payment 채널이 "open"된다. 
2. A는 Ether가 받는 사람에게 얼마나 지불해야하는지 지정하는 메시지에 서명한다. 이 단계는 각 지불에 대해 반복된다.
3. B는 payment 채널을 "close"하고 자신의 이더리움 부분을 인출하고 나머지 부분을 보낸사람에게 보낸다. 


> Note
> 
> 1,3 단계는 이더리움 transaction을 요구하고, 2는 보낸 사람이 암호화 서명된 메시지를 오프체인 방법을 통해 수신자에게 전송하는 것을 의미한다. 
> 즉, 임의 수의 전송을 지원하기 위해 두 개의 트랜잭션만 필요하다.


B는 smart contract로 ether를 서명 날인 하고, 유효한 서명된 메시지를 존중하기 때문에 자금을 받을 수 있다. 

smart contract는 또한 시간 제한을 적용하므로 A는 수신자가 채널 폐쇄를 거부하더라도 결국 자금을 회수할 수 있다. 

얼마나 오래 열어 둘지 결정하는 것은 payment 채널의 참가자에게 달려있다. 

네트워크 액세스 1분마다 인터넷 카페에 지불하는 등 단기 거래의 경우 payment 채널이 제한된 기간동안 열려 있을 수 있다.

반면 직원에게 시급과 같은 반복적인 지급에 대해서는 수개월 또는 수년간 개방할 수 있다. 

#### Opening the Payment Channel

payment 채널을 열기 위해서, A는 서명날인된 Ether를 를 연결하고 의도된 수신자와 채널이 존재하는 최대 기간을 지정하는  smart contract를 배포한다. 

마지막에 나오는 contract의 간편결제 채널 기능이다. 

#### Making Payments

A는 **서명된 메시지**를 B에게 보냄으로써 payments를 만든다. 

이 단계는 완전히 이더리움 네트워크의 외부에서 수행된다.

**메시지**는 발신인에 의해 암호로 서명된 다음 수신인에게 직접 전송된다.

각 **메시지**는 다음 정보를 포함한다.

    - cross-contract replay attacks을 방지하는데 사용되는, smart contract의 주소
    - 지금까지 수신자에게 지불해야 하는 총 Ether 양

일련의 transfer가 끝나면, payment 채널은 한번만 닫힌다.

이 때문에, 보낸 메시지 중 오직 하나만 사용된다. 

이런 이유로, 각 메시지는 개별 소액결제의 금액이 아니라 Ether의 누적 총 금액을 지정한다. 

수신자는 당연히 가장 최근의 메시지가 총합계가 높은 메시지이기 때문에, **가장 최근의 메시지를 상환**하도록 선택할 것입니다.

smart contract는 단 하나의 메시지만을 받아들이기 때문에, 메시지 당 임시 값은 더 이상 필요하지 않다. 

smart contract의 주소는 하나의 payment 채널을 위한 메시지가 다른 채널에 사용되는 것을 방지하기 위해 사용된다.


이전 섹션의 메시지에 암호화 방식으로 서명하기 위해 수정된 js 코드

````Solidity
function constructPaymentMessage(contractAddress, amount) {
    return abi.soliditySHA3(
        ["address", "uint256"],
        [contractAddress, amount]
    );
}

function signMessage(message, callback) {
    web3.eth.personal.sign(
        "0x" + message.toString("hex"),
        web3.eth.defaultAccount,
        callback
    );
}

// `contractAddress`는  cross-contract replay attacks를 막는데 사용된다. 
// `amount` (in wei)는 얼마의 ether가 보내져야하는지 명시한다. 
function signPayment(contractAddress, amount, callback) {
    var message = constructPaymentMessage(contractAddress, amount);
    signMessage(message, callback);
}
````

#### Closing the Payment Channel

B가 그의 자금을 받을 준비가 되었을때, smart contract의 `close`를 호출해 payment를 close 할 시간다. 

채널을 닫으면, 수신자에게 ether를 지급하고, contract를 파기하여 남은 ether를 A에게 돌려 보낸다. 

채널을 닫으려면, B는 A가 서명한 메시지를 제공해야한다. 


smart contract에서는 **메시지에 보낸사람의 서명이 포함되어 있는지** 확인해야 한다. 

확인을 수행하는 프로세스는 수신인이 사용하는 프로세스와 동일하다. 

Solidity 함수는 `ValidSignature` 및 `recoverySigner` 작업이며 ,`ReceiverPays` contract에서 빌린 함수도 있다.

오직 payment 채널 수신자만 `close`함수를 호출할 수 있고,  이 기능은 **가장 최근의 결제 메시지**를 자연스럽게 전달하며, 그 메시지는 가장 높은 총액을 전달하기 때문이다. 

만약 sender가 이 함수 호출 할 수 있을 떄, 그들은 더 적은 액수의 메시지를 제공하고 **수신자를 속여 빚진 것을 빼앗을 수 있다**.


함수는 서명된 메시지가 지정된 매개 변수와 일치하는지 확인한다.

모든 것이 확인되면 수신인은 **이더넷의 일부를 전송받고**, 나머지 수신인은 자동 소멸을 통해 전송된다.

전체 계약에서 `close` 기능을 볼 수 있다.

#### Channel Expiration

B는 언제는지 payment 채널을 닫을 수 있지만, 그렇게 하지 못하면 A는 적립된 자금을 회수할 방법이 필요하다.

만료시간은 contract 배포시 설정되었다. 그 시간이 되면 A는 자금을 회수하기 위해 `claimTimeout` 를 호출할 수 있다.

 `claimTimeout` 함수는 **full contract** 에서 볼 수 있다. 
 
 
이 함수가 호출된 후에 B는 더 이상 Ether를 수신할 수 없으므로, Expiration에 도달하기 전에 채널을 닫는 것이 중요하다.


#### The full contract

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;
contract SimplePaymentChannel {
    address payable public sender;      // The account sending payments.
    address payable public recipient;   // The account receiving the payments.
    uint256 public expiration;  // Timeout in case the recipient never closes.

    constructor (address payable _recipient, uint256 duration)
        payable
    {
        sender = payable(msg.sender);
        recipient = _recipient;
        expiration = block.timestamp + duration;
    }

    /// 수신자는 송신자로부터 서명된 amount 를 제시하여 언제든지 채널을 닫을 수 있다.
    /// 수신자는 해당 amount를 보내고, 나머지는 송신자에게 간다.   
    function close(uint256 amount, bytes memory signature) public {
        require(msg.sender == recipient);
        require(isValidSignature(amount, signature));

        recipient.transfer(amount);
        selfdestruct(sender);
    }

    /// 송신자는 언제든지 expiration을 연장할 수 있다. 
    function extend(uint256 newExpiration) public {
        require(msg.sender == sender);
        require(newExpiration > expiration);

        expiration = newExpiration;
    }

    /// 만약 the recipient가 채널을 닫는 것 없이 timeout이 되었다면, ether는 송신자에게 보내진다. 
    function claimTimeout() public {
        require(block.timestamp >= expiration);
        selfdestruct(sender);
    }

    function isValidSignature(uint256 amount, bytes memory signature)
        internal
        view
        returns (bool)
    {
        bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));

        // payment sender의 서명을 확인한다. 
        return recoverSigner(message, signature) == sender;
    }

    /// All functions below this are just taken from the chapter
    /// 'creating and verifying signatures' chapter.

    function splitSignature(bytes memory sig)
        internal
        pure
        returns (uint8 v, bytes32 r, bytes32 s)
    {
        require(sig.length == 65);

        assembly {
            // first 32 bytes, after the length prefix
            r := mload(add(sig, 32))
            // second 32 bytes
            s := mload(add(sig, 64))
            // final byte (first byte of the next 32 bytes)
            v := byte(0, mload(add(sig, 96)))
        }

        return (v, r, s);
    }

    function recoverSigner(bytes32 message, bytes memory sig)
        internal
        pure
        returns (address)
    {
        (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

        return ecrecover(message, v, r, s);
    }

    /// builds a prefixed hash to mimic the behavior of eth_sign.
    function prefixed(bytes32 hash) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
    }
}
````

> Note
> 
> `splitSignature`함수는 모든 보안 체크에 사용하지 않는다. 

#### Verifying Payments

이전 섹션에서와 달리, payment 채널에서 메시지는 바로 전송되지 않는다.

수신자는 최신 메시지를 추적하여, payment 채널을 닫을 때 다시 작성한다.

즉, 수신인이 각 메시지에 대해 자체 검증을 수행하는 것이 중요하다.

그렇지 않으면 수신자가 돈을 받을 수 있다는 보장이 없다. 


수신자는 각 메시지를 다음과 같은 과정으로 검증해야한다.

1. 메시지에 contract address가 payment 채널과 같은지 검증
2. 새로운 총 금액이 예상한 금액과 같은지 검증
3. 새로운 총 금액이 조건부 날인 증서(escrowed) 된 Ether의 양을 초과하지 않는지 검증
4. 서명이 유효한지, payment 채널의 sender로부터 온것인지 검증


이 검증을 작성하기 위해 ethereumjs-util 라이브러리를 사용할 것이다.

마지막 단계는 여러가지 방법으로 수행할 수 있고, js를 사용한다.

다음 코드는 위의 서명 js코드에서 `constructMessage`함수를 빌린다.

````Solidity
// this mimics the prefixing behavior of the eth_sign JSON-RPC method.
function prefixed(hash) {
    return ethereumjs.ABI.soliditySHA3(
        ["string", "bytes32"],
        ["\x19Ethereum Signed Message:\n32", hash]
    );
}

function recoverSigner(message, signature) {
    var split = ethereumjs.Util.fromRpcSig(signature);
    var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
    var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
    return signer;
}

function isValidSignature(contractAddress, amount, signature, expectedSigner) {
    var message = prefixed(constructPaymentMessage(contractAddress, amount));
    var signer = recoverSigner(message, signature);
    return signer.toLowerCase() ==
        ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
}
````

## Modular Contracts

contract를 만들기 위한 **모듈식 접근 방법**은 복잡성을 줄이고 가독성을 향상시켜

개발 및 검토 중에 버그와 취약성을 식별하는데 도움이 된다.

동작이나 각 모듈을 개별적으로 지정하고 제어하는 경우, 고려해야 할 상호작용은 contract의 다른 모든 이동이 아닌 **모듈 사양 간의 상호작용** 이다.


아래 예시에서 contract는 `Balances` libarary의 `move` 메소드를 사용하여 주소간에 전송된 잔액이 예상한 값과 일치하는 지 확인한다. 

`Balances` libarary는 accounts의 balances를 적절하게 추적하는 독립된 component를 제공한다. 

`Balances` libarary는 절대 마이너스 잔액 또는 오버플로를 만들지 않고, 모든 잔액의 합계는 contract기간동안 변하지 않는다.

````Solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;

library Balances {
    function move(mapping(address => uint256) storage balances, address from, address to, uint amount) internal {
        require(balances[from] >= amount);
        require(balances[to] + amount >= balances[to]);
        balances[from] -= amount;
        balances[to] += amount;
    }
}

contract Token {
    mapping(address => uint256) balances;
    using Balances for *;
    mapping(address => mapping (address => uint256)) allowed;

    event Transfer(address from, address to, uint amount);
    event Approval(address owner, address spender, uint amount);

    function transfer(address to, uint amount) public returns (bool success) {
        balances.move(msg.sender, to, amount);
        emit Transfer(msg.sender, to, amount);
        return true;

    }

    function transferFrom(address from, address to, uint amount) public returns (bool success) {
        require(allowed[from][msg.sender] >= amount);
        allowed[from][msg.sender] -= amount;
        balances.move(from, to, amount);
        emit Transfer(from, to, amount);
        return true;
    }

    function approve(address spender, uint tokens) public returns (bool success) {
        require(allowed[msg.sender][spender] == 0, "");
        allowed[msg.sender][spender] = tokens;
        emit Approval(msg.sender, spender, tokens);
        return true;
    }

    function balanceOf(address tokenOwner) public view returns (uint balance) {
        return balances[tokenOwner];
    }
}
````
