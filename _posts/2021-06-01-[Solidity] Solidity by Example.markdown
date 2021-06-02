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
    // This declares a new complex type which will
    // be used for variables later.
    // It will represent a single voter.
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


#### Packing arguments

#### Recovering ther Message Signer in Solidity

#### Extracting the Signature Parameters

#### Computing the Message Hash

#### The full contract

### Writing a Simple Payment Channel



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
