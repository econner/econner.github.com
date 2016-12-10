---
layout: post
title: An Outline of A Note on Distributed Computing
category: notes
---

# {{page.title}}

<span class="meta">September 4, 2013</span>

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

<h2 class="sectionTitle">0. Abstract</h2>

* Objs in distributed system treated differently than objs in single address space.
* Four reasons: latency, memory access, concurrency, partial failure
* Systems that paper over distinctions not robust or reliable

<h2 class="sectionTitle">I. Introduction</h2>

* Much work based on assump that objs form single ontological class
* Thesis: above view mistaken.
* Work that ignores differences doomed to failure.
* Terminology
  * local computing: progs confined to single addr space
  * distributed computing: progs that call to other addr spaces
  * middle ground e.g. calls between addr spaces on same machine

<h2 class="sectionTitle">II. The Vision of Unified Objects</h2>

* Obj defined in interface def language and implementation is independent
* "Objects all the way down"
* Three phases of dev
  * Write app without worrying about object location
  * Tune performance by concretizing object locs and communication
  * Final phase to test with "real bullets" then add components to deal ("fault points" will appear)
* Mistaken assumptions we will disprove
  * single natural obj-oriented design exists for app regardless of context
  * failure and performance tied to implementation
  * interface of obj independent of context

<h2 class="sectionTitle">III. Deja Vu All Over Again</h2>

* Two paths for comm protocol dev: integrate w/ current lang model vs solve inherent distributed systems problems
* Difficult part not the communication, hard part is partial failure, lack of central resource manager

<h2 class="sectionTitle">IV. Local and Distributed Computing</h2>

* Latency
  * 4 to 5 orders of magnitude difference
  * False answers: rely on steadily increasing speed of hardware, admin need for tools to see how objects interact and then bring those objects together
  * Could strive for future in which difference is conceptually indistinguishable
* Memory Access
  * Pointers not valid across address spaces
  * Could make all pointer refs return obis but would violate local paradigm
  * Danger: promote myth that remote and local accesses are same without enforcing myth
* Partial failure and concurrency
  * Both worlds contain components subject to periodic failure, but only detectable in local.
  * E.g. failure of network link indistinguishable from failure of processor on other side.
  * In distributed computing, very difficult to restore system to consistent state after failure because no single manager of state
  * Impossible to know state of system before and after failure so single components must be able to state possible causes of failure.
  * There is not indeterminism in how much computation completed in a local system.
  * What is the price to make remote & local identical?
     * Path 1: Treat all as if local.  Result is indeterministic in face of partial failure and therefore fragile.
     * Path 2: Design all interfaces as if remote, but this adds unnecessary complexity to local computing and defeats purpose.
  * Distributed objects must handle concurrent method invocations.
  * Differences from multi-threaded app:
     * In multi-thread, there is no real source of indeterminacy
     * Distributed has no single pt of resource allocation (threaded has OS)

<h2 class="sectionTitle">V. The Myth of "Quality of Service"</h2>

* Could leave it to implementation of object to deal with above issues.
* I.e. to build a more reliable system, choose more reliable implementations of interfaces making up system.
* Robustness also depends on interaction between components (e.g. enqueuing work multiple times).
* What happens if the client chooses to repeat this operation with the exact same params as previously?

<h2 class="sectionTitle">VI. Lessons from NFS</h2>

* Non-distributed API re-implemented in distributed way
* To deal with inaccessible file server soft mount or hard mount
  * Soft mounts expose network failure to client program.
     * Ops fail much more often and programs w/ no allowance for these failures will corrupt FS.
  * Hard mount: hang until server comes back up.
     * One server crashes and many workstations freeze.
  * NFS statelessness reduces complexity of failure modes.
* NFS is successful, but depends on a centralized resource manager (a sysadmin) to deal with resource reclamation, etc.

<h2 class="sectionTitle">VII. Taking the Difference Seriously</h2>

* Do not try to merge local and remote obis.  Instead, be constantly reminded of their differences.
* Need to consider anticipated msg frequency for obj and whether clients can accept indeterminacy implied by remote access.
* Interface must allow for reliability in face of partial failure.

<h2 class="sectionTitle">VII. A Middle Ground</h2>

* "Local-remote" objects do exist.  Objects are in different address space, but managed by a single resource manager.
* Failure modes are more nearly deterministic.
