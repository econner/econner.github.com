---
layout: post
title: An Outline of Time Clocks and the Ordering of Events
---

# {{page.title}}

<span class="meta">April 6 2013</span>

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

* Time fundamental to ordering of events.
* Distributed system communicates via message passing.
* System distributed if message trans delay not negligible comp to time between events in single proc.
* Sometimes impossible to say one of two events occurred first.  "happened before" only partial ordering.
* How to extend this to a total ordering of all events?

<h2 class="sectionTitle">The Partial Ordering</h2>

* a happened before b is a happened at earlier time than b
* Definition of system
   * Composed of collection of processes
   * Event => execution of subprogram, or execution of single machine instruction
   * Events form a sequence and process has a priori total ordering.
   * Sending or receiving msg is an event.
* Define relation "-->"
   * If a and b events in same proc and a before b then a --> b
   * If a is sending of msg by one proc and b is receipt of msg by another proc then a --> b
   * If a --> b and b --> c then a --> c
   * Concurrent if a -/-> b and b -/-> a
* a --> b means that a can causally affect b

<h2 class="sectionTitle">Logical Clocks</h2>

* Clock = way to give # to event
* Define C_i for every P_i which assigns C_i(a) to any event a in that proc
* Global C(b) = C_j(b) if b occurs in P_j
* Clock condition: For any events a, b: if a --> b then C(a) < C(b)
   * C1. If a and b are events in process P_i and a comes before b then C_i(a) < C_i(b)
   * C2. If a is the sending of msg by proc P_i and b is receipt of msg by proc P_j, then C_i(a) < C_j(b)
* Process' clock "ticks" through every number, ticks occur between process' events
   * IR1. Each process P_i incurs C_i between any two successive events.
   * IR2. (a) If event a is sending of msg m by proc P_i then msg m has T_m = C_i(a). (b) Upon receiving msg m, proc P_j sets C_j greater than or equal to present val and greater than T_m.

<h2 class="sectionTitle">Ordering the Events Totally</h2>


* Define => (where < is any arbitrary total ordering of procs)
   * If a is event in proc P_i and b in P_j then a => b iff (i) C_i(a) < C_j(b) or (ii) C_i(a) = C_j(b) and P_i < P_j
* Define total ordering to solve mutual exclusion problem
   * System composed of fixed collec of procs sharing resource
   * Alg to grant resource to proc w/ 3 conditions
      * Proc must release before resource can be re-granted
      * Requests granted in order made
      * If every proc eventually releases, then every request eventually granted.
   * Each proc maintains request queue
   * Msgs between procs received in same order sent
   * Algorithm to grant resources
      * P_i sends message T_m : P_i requests resource to every other proc and on own request queue (where T_m is timestamp of message)
      * P_j receives msg, queue it, sends ack to P_i
      * P_i releases by removing P_i requests resource msg from queue and sends P_i releases resource msg to every other proc
      * P_j receives P_i releases resource msg and removes any T_m : P_i requests resource
      * P_i granted resource when T_m : P_i requests resource in queue ordered before any other request by relation => and received msg timestamped later than T_m

<h2 class="sectionTitle">Anomalous Behavior</h2>

* Events known to occur before each other can be assigned a different ordering if knowledge propagates outside of system.
* Strong Clock Condition:  For any events a, b in L: if a --> b then C(a) < C(b) where L is the set of all system events.

<h2 class="sectionTitle">Physical Clocks</h2>

* C_i(t) reading of clock C_i at physical time t
* PC1. There exists a constant K << 1 s.t. for all i: | dC_i(t) / dt - 1 | < k
* PC2. For all i, j: | C_i(t) - C_j(t) | < e
* To avoid anomalous behavior:
   * Make sure that C_i(t + u) - C_j(t) > 0
   * PC1 implies that C_i(t + u) - C_i(t) > (1- K)u
   * Using PC2, e / (1 - k) <= u
