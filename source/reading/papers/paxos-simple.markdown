---
layout: page
title: "Paxos Made Simple"
sharing: true 
comments: true
footer: true
--- 

[Origin paper link](http://pdos.csail.mit.edu/6.824-2013/papers/paxos-simple.pdf)

### Problem 

safety requirements 

- Only a value that has been proposed may be chosen
- Only a single value is chosen, and 
- A process never learns that a value has been chosen unless it 
  acutally has been


three agents: *proposers*(P), *acceptors*(A), and *learners*(L).


use customary asynchronous, non-Byzantine model, in which:

- Agents operate at arbitrary speed, may fail by stopping, and may
  restart
- Messages can take arbitrarily long to be delivered, can be duplicated,
  and can be lost, but they are not corrupted.

### Choosing a Value

Single acceptor, simple but unsatisfacroty, suffer from failure
of this single acceptor.


**Majority of the agents?**
Literally understand as more than half of acceptors. 


To ensure that only a single value is chosen, we can let a large
enough set consist of **any majority of the agents**. Because any two 
majorities have at least one acceptor in common, this works if an 
acceptor can accept at most one value.


> P1. An acceptor must accept the first proposal that it receives. 

This ensure that the value got chosen if there is only one value 
proposed. But raises the problem when more than two values are 
proposed and each got same amount of acceptors(3 values, each 1/3 of 
all acceptors). 

**Acceptor must be allowed to accept more than one proposals**
Though there can be only one value that got chosen, but each acceptor indeed could accept more than one proposals.

**Proposal number is global? or for each acceptor? How to achieve global?**
It should be global.

> P2. If a proposal with value *v* is chosen, then every higher-numbered 
> proposal that is chosen has value *v*.

P2 guarantees the crucial safety property that only a single value is chosen.


> P2a. If a proposal with value *v* is chosen, then every higher-numbered 
> proposal accepted by any acceptor has value *v*.

P1 may conficts P2a in some situations.
Suppose a proposal was chosen with some particular acceptor *c* never 
having received any proposal. A new proposer "wakes up" then and 
issues a higher-numbered proposal with a different value. P1 requires
*c* to accept this proposal, violating P2a. 

Maintaining both P1 and P2a requires strengthening P2a to:

> P2b . If a proposal with value *v* is chosen, then every higher-numbered 
> proposal issued by any proposer has value *v*.

**Difference between concepts *chosen*, *accept* and *issue*?** *Chosen* is a global state that a value *v* has been accepted by majority of acceptors, the whole system can only choose one value. *Accept* is the behavior of a single acceptor, the acceptor can change its mind to accept another newer proposal at any time. *Issue* is the behavior of a single proposer, if a value *v* is *chosen* (globally accepted), then all proposers would make compromise to propose *v*.


Given that **any two sets of majaority acceptors must have at least one acceptor in common**. We want the following invariance meet:

>P2c. For any *v* and *n*, if a proposal with value *v* and number *n* is 
>issued, then there is a set *S* consisting of a majority of 
>acceptors such that
>either (a) no acceptor in *S* has accepted any proposal numbered less
>than *n*, or (b) *v* is the value of the highest-numbered proposal 
>among all proposals numbered less than *n* accepted by the acceptors 
>in *S*.

To maintain the invariance of P2c, a proposer that wants to issue a 
proposal numbered *n* must learn the highest-numbered proposal with 
number less than *n* that has been or will be accepted by each 
acceptor in some majority of acceptors.

It is hard to predict future acceptances, instead, the proposer controls 
it by extracting a **promise** that the acceptors won't accept any more 
proposals numbered less than *n*. 

Note that P2c guaranteed that **if a value *v* is chosen, then the highest-numbered proposal must have value *v***.

**Algorithm for a proposer to issue proposals**:

1. A proposer choses *n*, sends a request to each acceptors in some 
set, asking:
    1. Promise it won't accept a proposal numbered less than *n*
    2. The proposal with highest number less than *n* that it has accepted.
2. The proposer can issue a proposal with number *n* and *v* if it 
receives responses from a majority of the acceptors, where *v* is the 
value of the highest-numbered proposal among the responses, or is any 
value if responders reported no proposals. 


The request in step 1 is a *prepare* request, and that in step 2 is 
an *accept* request.

**How an acceptor responds to requests?**
It can always respond to a *prepare* request, and it can respond to an 
*accept* request iff it has not promised not to. 

>P1a. An acceptor can accept a proposal numbered *n* iff it has not 
>responded to a prepare request having a number greater than *n*.

An acceptor needs to remember only the highest-numbered proposal that 
it has ever accepted and the number of the highest-numbered prepare 
request to which it has responded. 

**Note that the proposer can always abandon a proposal and forget all about it—as long as it never tries to issue another proposal with the same number. What if a proposer got a promise but never issued?**
Just like a network package lose. Will be eventually replaced by 
other proposals.



**Phase 1.**
**(a)** A proposer selects a proposal number *n* and sends a *prepare*
request with number *n* to a majority of acceptors.
**(b)** If an acceptor receives a *prepare* request with number *n* greater
than that of any *prepare* request to which it has already responded,
then it responds to the request with a promise not to accept any more
proposals numbered less than *n* and with the highest-numbered proposal
(if any) that it has accepted.

**Phase 2.**
**(a)** If the proposer receives a response to its *prepare* requests
(numbered *n*) from a majority of acceptors, then it sends an *accept*
request to each of those acceptors for a proposal numbered *n* with a
value *v* , where *v* is the value of the highest-numbered proposal 
among the responses, or is any value if the responses reported no 
proposals.
**(b)** If an acceptor receives an *accept* request for a proposal numbered 
*n*, it accepts the proposal unless it has already responded to a 
*prepare* request having a number greater than *n*.


**Optimization**: abandon a proposal if some proposers has begun trying to 
issue a higher-numbered one.


### Learning a Chosen Value 
Acceptors return responds to *accept* requests to all the learners, 
the number of responds that required equals to the product of the number 
of acceptors and the number of learners. 

Acceptors could return only to a set of distinguished learners, these 
learners will inform other learners.

### Progress

It’s easy to construct a scenario in which two proposers each keep 
issuing a sequence of proposals with increasing numbers, none of which 
are ever chosen. 

To guarantee progress, a distinguished proposer must be selected as the
only one to try issuing proposals.

Result of [Fischer, Lynch, and Patterson](http://dl.acm.org/citation.cfm?id=214121) shows a reliable algorithm 
to electing a proposer must use either randomness or real time.


### Implementation

[The part-time parliament](http://dl.acm.org/citation.cfm?id=279229)

**No two proposals are ever issued with the same number?** 
Different proposers choose their numbers from disjoint sets of numbers,
each proposer remembers (in stable storage) the highest-numbered 
proposal it has tried to issue.


## My Summary
Behavior of *Proposer*, *Acceptor* and *Learner*

### Proposer
1. sends prepare request *n* to all acceptors
2. after receives responds from majority of acceptors, choose a value *v* according to responds
	1. *v* should be the value of the highest-numberd proposal in those responds
	2. if none of responds returns any proposal, use arbitrary value
	3. if any responds contains an error or a proposal whose number is bigger than *n*, go to next step
3. repick a bigger *n* and repeat step 1 & 2 until a value is chosen


### Acceptor
1. Acceptor should know the highest-numbered proposal it accepted *APa* and the highest-numbered proposal it responsed *APr*
2. Upon receiving a prepare request *P*, compare *P.n* with *APr.n*
	1. if *P.n* > *APr.n*, respond a promise and *APa*, then change *APr* to *P*
	2. if *P.n* <= *APr.n*, respond with some error or *APr*?
3. Upon receiving a accept request *P*
	1. if *P.n* >= *APr.n*, accept it by making *APa* equals to *P*
	2. if *P.n* < *APr.n*, abondon it


### Learner
TODO
