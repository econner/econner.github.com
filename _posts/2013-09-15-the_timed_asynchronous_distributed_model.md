---
layout: post
title: An Outline of The Timed Asynchronous Distributed Model
---

# {{page.title}}

<span class="meta">September 15, 2013</span>

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

<h2 class="sectionTitle">I. Introduction</h2>

* "certain communication"
   * At any time min # correct procs exists
   * Msg m sent between correct prods received & processed w/in known amt of time
   * Sync system guarantees "certain comm" all other sys async
   * Async gives lower prob of indiv assumption violated
* Time-free model
   * Services have no time specs, just state transitions
   * Interproc communication is reliable
   * Procs only fail by crashing
   * Procs have no access to hardware clocks
   * Procs cannot distinguish between slow and crashed proc
* Timed async distributed system model
   * All services timed
   * Interproc communication uses unreliable datagram service: only suffer msg omission and performance failures
   * Procs only crash or have performance failures
   * Proces access hardware clocks
   * No bound on failures of comm and prods
* Timed model allows for partitioning of area if becomes unreachable
   * Allows partitions to be modeled naturally
   * Assumps that services timed not restrictive in practice
* Goals of this paper
   * Formal def of timed async distr model
   * Extensive measurements of actual msg process scheduling delays + clock drifts
   * Intuitive explanation of how practical systems (leader election) are implementable in timed model and not in time-free model.

<h2 class="sectionTitle">II. Related Work</h2>

* Timed async model "taxonomy"
   * network topology: procs know complete set and can send msgs to any proc
   * synchrony: services timed & clock drift rate bounded
   * failure model: procs can crash or have perf fails, comm service can suffer omission of perf fails
   * message buff: finite buffers + out of order
* Tuned sys alternates between stability and short periods of instability
   * progress assumption
   * conditional timeliness
   * global stabilization
   * failure detectors


<h2 class="sectionTitle">III. The Model</h2>


* Procs on network communicate via low level datagram service.  Remote if on separate nodes, local otherwise.
* Proc p has access to local hardware clock.
* Proc management service uses clock to wake procs at various times.
* Hardware Clocks

   * Correct clocks have strictly monotonically increasing values

      * H_p : RT --> CT
      * (maps real-time to clock-time)

   * Assume constant max drift rate p << 1
   * Correct clock measures interval \[s, t\] w/ error \[-p(s-t) - G, p(t-s) + G\] (p drift rate, G granularity, 1 ns etc)
   * External sync: deviation between correct clock and real-time bounded by known constant
   * Internal sync: deviation between each other bounded
   * Calibration

      * Speed changed by constant factor c to reduce systematic drift error.
   * Measurements

      * Granularity on order or 1 ns to 1 microsecond, drift rate 10^-4 to 10^-6
   * Clock Failure Assumption
      * Assume non crashed proc has access to correct hardware clock
      * Redundant clocks make prob of faulty read negligible
* Datagram Service
   * Three ops: send(m, q), broadcast(m), deliver(m, p)
   * Three requirements: validity, no-dupe, min-delay
   * Measurements
      * One way timeout delay (del)
      * (del) depends on netwrok load but is protocol dependent
      * Msg omission not independent of load
   * Failure Assumption
      * Ommission / performance failure semantics w/ low probability of source address spoofing, corruption, or duplication.
      * Does not ensure existence of upper bound for transmission delay
* Process Management Service

   * Process Modes
      * up: executing standard code
      * crashed: stopped executing, can't take next step
      * recovering: exec state init code
      * 4 events to transition: start, crash, ready, recover

   * Alarm Clocks
      * p sets alarm to be wakened
      * p wakes if (1) mgmt service wakes it (2) receives msg before wakened, can only have 1 alarm at a time
      * SetAlarm_p_s (T): p requests at real-time s to be woken at future time u s.t. H_p(u) >= T
      * WakeUp_p_u (T): proc mgmt service wakes up p at real-time u

<h2 class="sectionTitle">IV. Extensions</h2>

* Two optional extensions:
   * stable storage: allow procs to store memory state between crashes
   * progress assumption: majority of procs will be "stable" (behave like synchronous system) for bounded amount of time

<h2 class="sectionTitle">V. Communication by Time</h2>

* Sync system detects crash via "I-am-alive" msg not received in time.
* Aync system cannot decide if leader has crash, is slow, or communication is slow.
* Time-based locking
   * p sends info to q saying its only valid for specified amount of time
   * q calculates upper bound on transmission delay of m to determine how long it can use m
   * p consults local hardware clock to determine time beyond which q will stop using m

<h2 class="sectionTitle">VI. Possibility and Impossibility Issues</h2>


* Election and consensus are implementable in actual distributed computing systems, but do not allow deterministic solution in the time-free model or the core timed model.
* Easier guarantee is that as long as there is a majority of delta-F-stable processes in a time interval, then there exists a proc that becomes leader in that interval.

