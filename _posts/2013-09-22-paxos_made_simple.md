---
layout: post
title: An Outline of Paxos Made Simple
---

<span class="meta">September 22, 2013</span>

<style>
p {
    margin: 0;
}
h2.sectionTitle {
    margin-bottom: 10px;
}
#post ul {
    margin-left: 20px;
}
</style>

<h2 class="sectionTitle">Introduction</h2>

<h2 class="sectionTitle">The Consensus Algorithm</h2>

* The Problem
   * Collection of procs can propose values, consensus ensures single value chosen
   * Safety Requirements
      * Only a value proposed may be chosen
      * Only a single val is chosen
      * Proc never learns val has been chosen unless actually chosen
   * Liveness goals

      * Eventually some proposed val chosen, proc can eventually learn the value
   * Three types of agents: proposers, acceptors, learners
      * Agents send messages to each other
      * Agents operate at arbitrary speed, may fail by stopping, may restart.  Sol'n impossible unless info can be remembered by agent.
      * Msgs take arbitrarily long to be delivered, can be duplicated, can be lost, not corrupted
* Choosing a Value
   * Use majority of acceptors.  Val chosen when majority accept.
   * P1. Acceptor must accept first proposal that it receives.
      * Problem: can result in plurality.
      * Even if only 2 proposals, if 1 acceptor fails, could become impossible to agree on val
   * So..acceptor must be allowed to accept more than 1 proposal.

      * Assign natural number to each proposal.  Diff proposals have diff numbers.

   * P2. If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v.
   * P2b. If a proposal with value v is chosen, then every higher-numbered proposal issued by any proposer has value v.
   * P2c. For any v and n, if a proposal with value v and number n is issued, then there is a set S consisting of a majority of acceptors such that either (a) no acceptor in S has accepted any proposal numbered less than n, or (b) v is the value of the highest-numbered proposal among all proposals numbered less than n accepted by the acceptors in S.
   * Algorithm for issuing proposals:

      * A proposer chooses a new proposal number n and sends a request to each member of some set of acceptors, asking it to respond with:

         * A promise never again to accept a proposal numbered less than n, and
         * The proposal with the highest number less than n that it has accepted, if any.
         * This is a prepare request with number n

      * If proposer receives request response from majority of acceptors it can issue proposal w/ number n and value v.  Proposer issues proposal with accept request.

   * P1a. An acceptor can accept a proposal numbered n iff it has not responded to a prepare request having number greater than n.

   * Two phases

      * Phase 1

         * A proposer selects a proposal number n and sends a prepare request with number n to a majority of acceptors.
         * If an acceptor receives a prepare request w/ number n greater than that of any prepare request to which already responded, then it responds to the request w/ promise not to accept any more proposals numbered less than n and with highest-numbered proposal it has accepted.

      * Phase 2

         * If proposer receives a response to prepare requests from majority of acceptors, it sends accept request to each acceptor for proposal numbered n w/ value v.
         * If acceptor receives accept request for proposal numbered n it accepts proposal unless already responded to prepare request having number greater than n.

* Learning a Chosen Value
   * Learner must find out about proposal that has been accepted by majority of acceptors.
   * Can use distinguished learners to reduce computation cost, but will incur complexity cost.
* Progress
   * Dueling proposers can halt progress.
   * Distinguished proposer must be selected to issue proposals.
* The Implementation
   * Acceptors need stable storage to recover from failure.
   * Different proposers choose their proposal numbers from disjoint sets.

<h2 class="sectionTitle">Implementing a State Machine</h2>

* Set of servers all executing a state machine.
* Elect single server to be leader and distinguished proposer.
* Clients send commands to leader who decides on sequence.
* Run an instance of paxos for every command to decide on sequence.
* Cost of algorithm dominated by phase 2 which is essentially optimal.
