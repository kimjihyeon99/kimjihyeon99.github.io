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

    // This is a type for a single proposal.
    struct Proposal {
        bytes32 name;   // short name (up to 32 bytes)
        uint voteCount; // number of accumulated votes
    }

    address public chairperson;

    // This declares a state variable that
    // stores a `Voter` struct for each possible address.
    mapping(address => Voter) public voters;

    // A dynamically-sized array of `Proposal` structs.
    Proposal[] public proposals;

    /// Create a new ballot to choose one of `proposalNames`.
    constructor(bytes32[] memory proposalNames) {
        chairperson = msg.sender;
        voters[chairperson].weight = 1;

        // For each of the provided proposal names,
        // create a new proposal object and add it
        // to the end of the array.
        for (uint i = 0; i < proposalNames.length; i++) {
            // `Proposal({...})` creates a temporary
            // Proposal object and `proposals.push(...)`
            // appends it to the end of `proposals`.
            proposals.push(Proposal({
                name: proposalNames[i],
                voteCount: 0
            }));
        }
    }

    // Give `voter` the right to vote on this ballot.
    // May only be called by `chairperson`.
    function giveRightToVote(address voter) public {
        // If the first argument of `require` evaluates
        // to `false`, execution terminates and all
        // changes to the state and to Ether balances
        // are reverted.
        // This used to consume all gas in old EVM versions, but
        // not anymore.
        // It is often a good idea to use `require` to check if
        // functions are called correctly.
        // As a second argument, you can also provide an
        // explanation about what went wrong.
        require(
            msg.sender == chairperson,
            "Only chairperson can give right to vote."
        );
        require(
            !voters[voter].voted,
            "The voter already voted."
        );
        require(voters[voter].weight == 0);
        voters[voter].weight = 1;
    }

    /// Delegate your vote to the voter `to`.
    function delegate(address to) public {
        // assigns reference
        Voter storage sender = voters[msg.sender];
        require(!sender.voted, "You already voted.");

        require(to != msg.sender, "Self-delegation is disallowed.");

        // Forward the delegation as long as
        // `to` also delegated.
        // In general, such loops are very dangerous,
        // because if they run too long, they might
        // need more gas than is available in a block.
        // In this case, the delegation will not be executed,
        // but in other situations, such loops might
        // cause a contract to get "stuck" completely.
        while (voters[to].delegate != address(0)) {
            to = voters[to].delegate;

            // We found a loop in the delegation, not allowed.
            require(to != msg.sender, "Found loop in delegation.");
        }

        // Since `sender` is a reference, this
        // modifies `voters[msg.sender].voted`
        sender.voted = true;
        sender.delegate = to;
        Voter storage delegate_ = voters[to];
        if (delegate_.voted) {
            // If the delegate already voted,
            // directly add to the number of votes
            proposals[delegate_.vote].voteCount += sender.weight;
        } else {
            // If the delegate did not vote yet,
            // add to her weight.
            delegate_.weight += sender.weight;
        }
    }

    /// Give your vote (including votes delegated to you)
    /// to proposal `proposals[proposal].name`.
    function vote(uint proposal) public {
        Voter storage sender = voters[msg.sender];
        require(sender.weight != 0, "Has no right to vote");
        require(!sender.voted, "Already voted.");
        sender.voted = true;
        sender.vote = proposal;

        // If `proposal` is out of the range of the array,
        // this will throw automatically and revert all
        // changes.
        proposals[proposal].voteCount += sender.weight;
    }

    /// @dev Computes the winning proposal taking all
    /// previous votes into account.
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

    // Calls winningProposal() function to get the index
    // of the winner contained in the proposals array and then
    // returns the name of the winner
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
    // Parameters of the auction. Times are either
    // absolute unix timestamps (seconds since 1970-01-01)
    // or time periods in seconds.
    address payable public beneficiary;
    uint public auctionEndTime;

    // Current state of the auction.
    address public highestBidder;
    uint public highestBid;

    // Allowed withdrawals of previous bids
    mapping(address => uint) pendingReturns;

    // Set to true at the end, disallows any change.
    // By default initialized to `false`.
    bool ended;

    // Events that will be emitted on changes.
    event HighestBidIncreased(address bidder, uint amount);
    event AuctionEnded(address winner, uint amount);

    // Errors that describe failures.

    // The triple-slash comments are so-called natspec
    // comments. They will be shown when the user
    // is asked to confirm a transaction or
    // when an error is displayed.

    /// The auction has already ended.
    error AuctionAlreadyEnded();
    /// There is already a higher or equal bid.
    error BidNotHighEnough(uint highestBid);
    /// The auction has not ended yet.
    error AuctionNotYetEnded();
    /// The function auctionEnd has already been called.
    error AuctionEndAlreadyCalled();

    /// Create a simple auction with `_biddingTime`
    /// seconds bidding time on behalf of the
    /// beneficiary address `_beneficiary`.
    constructor(
        uint _biddingTime,
        address payable _beneficiary
    ) {
        beneficiary = _beneficiary;
        auctionEndTime = block.timestamp + _biddingTime;
    }

    /// Bid on the auction with the value sent
    /// together with this transaction.
    /// The value will only be refunded if the
    /// auction is not won.
    function bid() public payable {
        // No arguments are necessary, all
        // information is already part of
        // the transaction. The keyword payable
        // is required for the function to
        // be able to receive Ether.

        // Revert the call if the bidding
        // period is over.
        if (block.timestamp > auctionEndTime)
            revert AuctionAlreadyEnded();

        // If the bid is not higher, send the
        // money back (the revert statement
        // will revert all changes in this
        // function execution including
        // it having received the money).
        if (msg.value <= highestBid)
            revert BidNotHighEnough(highestBid);

        if (highestBid != 0) {
            // Sending back the money by simply using
            // highestBidder.send(highestBid) is a security risk
            // because it could execute an untrusted contract.
            // It is always safer to let the recipients
            // withdraw their money themselves.
            pendingReturns[highestBidder] += highestBid;
        }
        highestBidder = msg.sender;
        highestBid = msg.value;
        emit HighestBidIncreased(msg.sender, msg.value);
    }

    /// Withdraw a bid that was overbid.
    function withdraw() public returns (bool) {
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {
            // It is important to set this to zero because the recipient
            // can call this function again as part of the receiving call
            // before `send` returns.
            pendingReturns[msg.sender] = 0;

            if (!payable(msg.sender).send(amount)) {
                // No need to call throw here, just reset the amount owing
                pendingReturns[msg.sender] = amount;
                return false;
            }
        }
        return true;
    }

    /// End the auction and send the highest bid
    /// to the beneficiary.
    function auctionEnd() public {
        // It is a good guideline to structure functions that interact
        // with other contracts (i.e. they call functions or send Ether)
        // into three phases:
        // 1. checking conditions
        // 2. performing actions (potentially changing conditions)
        // 3. interacting with other contracts
        // If these phases are mixed up, the other contract could call
        // back into the current contract and modify the state or cause
        // effects (ether payout) to be performed multiple times.
        // If functions called internally include interaction with external
        // contracts, they also have to be considered interaction with
        // external contracts.

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


- 다음 예는 최고 입찰액보다 큰 값을 수락함으로써 이문제를 해결한다. 
- 공개 단꼐에서만 확인할 수 있기 때문에 일부 입찰은 무효일 수 있으며, 이는 고의적인것이다. 
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

    /// The function has been called too early.
    /// Try again at `time`.
    error TooEarly(uint time);
    /// The function has been called too late.
    /// It cannot be called after `time`.
    error TooLate(uint time);
    /// The function auctionEnd has already been called.
    error AuctionEndAlreadyCalled();

    // Modifiers are a convenient way to validate inputs to
    // functions. `onlyBefore` is applied to `bid` below:
    // The new function body is the modifier's body where
    // `_` is replaced by the old function body.
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

    /// Place a blinded bid with `_blindedBid` =
    /// keccak256(abi.encodePacked(value, fake, secret)).
    /// The sent ether is only refunded if the bid is correctly
    /// revealed in the revealing phase. The bid is valid if the
    /// ether sent together with the bid is at least "value" and
    /// "fake" is not true. Setting "fake" to true and sending
    /// not the exact amount are ways to hide the real bid but
    /// still make the required deposit. The same address can
    /// place multiple bids.
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

    /// Reveal your blinded bids. You will get a refund for all
    /// correctly blinded invalid bids and for all bids except for
    /// the totally highest.
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
                // Bid was not actually revealed.
                // Do not refund deposit.
                continue;
            }
            refund += bidToCheck.deposit;
            if (!fake && bidToCheck.deposit >= value) {
                if (placeBid(msg.sender, value))
                    refund -= value;
            }
            // Make it impossible for the sender to re-claim
            // the same deposit.
            bidToCheck.blindedBid = bytes32(0);
        }
        payable(msg.sender).transfer(refund);
    }

    /// Withdraw a bid that was overbid.
    function withdraw() public {
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {
            // It is important to set this to zero because the recipient
            // can call this function again as part of the receiving call
            // before `transfer` returns (see the remark above about
            // conditions -> effects -> interaction).
            pendingReturns[msg.sender] = 0;

            payable(msg.sender).transfer(amount);
        }
    }

    /// End the auction and send the highest bid
    /// to the beneficiary.
    function auctionEnd()
        public
        onlyAfter(revealEnd)
    {
        if (ended) revert AuctionEndAlreadyCalled();
        emit AuctionEnded(highestBidder, highestBid);
        ended = true;
        beneficiary.transfer(highestBid);
    }

    // This is an "internal" function which means that it
    // can only be called from the contract itself (or from
    // derived contracts).
    function placeBid(address bidder, uint value) internal
            returns (bool success)
    {
        if (value <= highestBid) {
            return false;
        }
        if (highestBidder != address(0)) {
            // Refund the previously highest bidder.
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
    /// The function cannot be called at the current state.
    error InvalidState();
    /// The provided value has to be even.
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

    // Ensure that `msg.value` is an even number.
    // Division will truncate if it is an odd number.
    // Check via multiplication that it wasn't an odd number.
    constructor() payable {
        seller = payable(msg.sender);
        value = msg.value / 2;
        if ((2 * value) != msg.value)
            revert ValueNotEven();
    }

    /// Abort the purchase and reclaim the ether.
    /// Can only be called by the seller before
    /// the contract is locked.
    function abort()
        public
        onlySeller
        inState(State.Created)
    {
        emit Aborted();
        state = State.Inactive;
        // We use transfer here directly. It is
        // reentrancy-safe, because it is the
        // last call in this function and we
        // already changed the state.
        seller.transfer(address(this).balance);
    }

    /// Confirm the purchase as buyer.
    /// Transaction has to include `2 * value` ether.
    /// The ether will be locked until confirmReceived
    /// is called.
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

    /// Confirm that you (the buyer) received the item.
    /// This will release the locked ether.
    function confirmReceived()
        public
        onlyBuyer
        inState(State.Locked)
    {
        emit ItemReceived();
        // It is important to change the state first because
        // otherwise, the contracts called using `send` below
        // can call in again here.
        state = State.Release;

        buyer.transfer(value);
    }

    /// This function refunds the seller, i.e.
    /// pays back the locked funds of the seller.
    function refundSeller()
        public
        onlySeller
        inState(State.Release)
    {
        emit SellerRefunded();
        // It is important to change the state first because
        // otherwise, the contracts called using `send` below
        // can call in again here.
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
// recipient is the address that should be paid.
// amount, in wei, specifies how much ether should be sent.
// nonce can be any unique number to prevent replay attacks
// contractAddress is used to prevent cross-contract replay attacks
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

        // this recreates the message that was signed on the client
        bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

        require(recoverSigner(message, signature) == owner);

        payable(msg.sender).transfer(amount);
    }

    /// destroy the contract and reclaim the leftover funds.
    function shutdown() public {
        require(msg.sender == owner);
        selfdestruct(payable(msg.sender));
    }

    /// signature methods.
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

    /// builds a prefixed hash to mimic the behavior of eth_sign.
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

// contractAddress is used to prevent cross-contract replay attacks.
// amount, in wei, specifies how much Ether should be sent.

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

    /// the recipient can close the channel at any time by presenting a
    /// signed amount from the sender. the recipient will be sent that amount,
    /// and the remainder will go back to the sender
    function close(uint256 amount, bytes memory signature) public {
        require(msg.sender == recipient);
        require(isValidSignature(amount, signature));

        recipient.transfer(amount);
        selfdestruct(sender);
    }

    /// the sender can extend the expiration at any time
    function extend(uint256 newExpiration) public {
        require(msg.sender == sender);
        require(newExpiration > expiration);

        expiration = newExpiration;
    }

    /// if the timeout is reached without the recipient closing the channel,
    /// then the Ether is released back to the sender.
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

        // check that the signature is from the payment sender
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
