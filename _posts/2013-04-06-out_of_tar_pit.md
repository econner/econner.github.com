---
layout: post
title: Out of the Tar Pit
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

<h2 class="sectionTitle">I. Introduction</h2>

- Major contributor of complexity is handling of state
- Others: code volume, concern with flow of control
- Typical approaches
  - Object oriented: couple state w/ relational behavior
  - Functional: no state, no side-effects

<h2 class="sectionTitle">II. Complexity</h2>

* Root cause of software problems is unmanageable complexity
* Dijkstra: “...we have to keep it crisp, disentangled, and simple if we refuse to be crushed by the complexities of our own making…”
* Complexity = "that which makes large systems hard to understand"

<h2 class="sectionTitle">III. Approaches to Understanding</h2>

* Two approaches to understanding

  * Testing: system is black box

    - Leads to more erros being detected

  * Informal reasoning: examine from the inside

    * Leads to less errors being created

* All methods to understand system have limits

* Invest in simplicity over testing

<h2 class="sectionTitle">IV. Causes of Complexity</h2>

* Complexity caused by State
  * State makes programs hard to understand and therefore complex.
  * Too difficult to enumerate all possible states which causes unreliability
  * Impact of State on Testing
     * Test in one state tells you nothing about behavior in another
     * Tests start w/ clean state
     * Num of inputs to system large, num of states larger

  * Impact of State on Informal Reasoning

     * Num states grows exponentially, each bit of state doubles total

* Complexity caused by Control
  * Choosing an ordering of statements that do not depend on that ordering introduces unnecessary complexity
  * Concurrency
     * Shared state sucks
     * Testing no longer is consistent

* Complexity caused by Code Volume
  * Volume increases nonlinearly with size
  * Dijkstra: People tend to think intellectual effort to understand system grows by square of size .. i.e. because components of the system could all interact
  * Abstraction reduces interaction

* Other causes of complexity
  * Duplicated code, dead code, unnecessary abstraction, …
  * Complexity breeds complexity

     * If it is not clear that functionality exists, tend to dupe
  * Simplicity is Hard
  * Power corrupts
     * In the absence of language-enforced guarantees, mistakes will happen.
     * The more possible in a language the harder it is to understand.

<h2 class="sectionTitle">V. Classical approaches to managing complexity</h2>

* Object-Orientation
  * State access procedures must enforce constraint for every piece of state manipulated
  * Where does constraint enforcement go for multiple-object relationships?
  * Identity and State
     * Every obj is independent entity regardless of attributes
     * Leads to awkward middle ground "Value Objects" that you'd like to have extensional identity

* Functional Programming
  * Roots in stateless lambda calculus of Church
  * State

     * No state gives referential transparency: given set of args, always return same result

  * Control

     * Encourage more abstract use of control (map) than explicit looping

  * Kinds of State
     * Mutable vs. immutable state.
     * Functional way to write counter method below.
     * Caller becomes responsible for making sure fn called appropriately.

<pre style="margin-top: 10px; margin-bottom:10px;">
    <code>
    function (int, int) getNextCounter(int oldCounter) {
        let int result = oldCounter + 1
        let int newCounter = oldCounter + 1
        return (newCounter, result)
    }
    </code>
</pre>

* Cont'd...

  * State and Modularity
     * Trade-off between modularity given by state and explicitness given by functional
     * Trade-off is between complexity and simplicity..i.e. you know the outcome of a procedure by its args

* Logic Programming
  * Specify what needs to be done rather than how to do it
  * State a set of axioms along with attributes required of a solution

<h2 class="sectionTitle">VI. Accidents and Essence</h2>

* Essential complexity: Inherent in the problem as seen by the users
* Accidental complexity: All the rest (performance, etc)
* In ideal world, we could use lang and infra which would let us express users' problem directly

<h2 class="sectionTitle">VII. Recommended General Approach</h2>

* What complexity can't be avoided even in ideal world?
* Ideal World
  * Start with informal requirements
  * Translate to formal ones

    * Must be done w/ no view to execution (i.e. only ensure there are no omissions)

  * State in ideal world

    * All data either provided directly (input) or derived

      * Derived data immutable if only for display or mutable if users can update
  * Input data
    * Derived data is always accidental state
    * For mutable state, function must have inverse
* Theoretical and Practical Limitations
  * Systems have state as part of essence
  * Control generally is accidental
  * Practical (e.g. efficiency) dictates use of accidental state
  * Formal Spec Languages
    * Property based: what is required rather than how
    * Model-based: construct a potential model for the system and specify how it must behave
    * Generally formal langs not expressive enough for all specs
  * Ease of Expression

    * Sometimes it is more natural to express derived state (e.g. loc on a map rather than seq of moves)
  * Required Accidental Complexity
    * Performance
    * Ease of expression

* Recommendations

  * Avoid and separate
     * Avoid state and control where not essential
     * Separate it out from rest of system
  * Required accidental complexity
     * Performance: leave to separate system
     * Ease of expression
        * Occurs when derived state is most natural way to express parts of system
        * Use an external component to mock derived state
  * Separation and the relationship between components
     * Separate all complexity from pure logic of sys
     * Divide complexity into accidental and essential
     * 3 expected components
        * Accidental state and control: Never should affect other parts of system
        * Essential Logic: "heart" of the system, expresses in terms of state what must be true, must not be able to change state
        * Essential State: makes no reference to either of the other parts

<h2 class="sectionTitle">VII. The Relational Model</h2>

* Four aspects
  * Structure: relations are means to represent data
  * Manipulation: means to specify derived data
  * Integrity: inviolable restrictions on data
  * Data Independence: sep logical from physical
* Structure
  * Set of records each of which has uniquely named attrs
  * Base relations vs derived relations
  * Access path independence

     * No up-front decisions have to be made about access paths used to query and process
  * Contrast w/ hierarchical

     * e.g. Department chosen to contain employees so to find employee you must access through department
  * Contrast w/ network

     * Predict all access paths…bad
* Manipulation
  * Restrict, project, product, union, intersection, difference, join, divide
  * Property of closure: operands and results are of same kind (relations)
* Integrity
  * Specify set of constraints which must hold at all times
  * e.g. foreign key, primary key
* Data Independence

  * Sep logical from physical, not unlike accidental / essential split

<h2 class="sectionTitle">IX. Functional Relational Programming</h2>

* Architecture

  * Essential state

     * relational def of stateful components of system
     * all state stored solely in relations
     * treat data as essential state only when input directly by a user
  * Essential logic

     * Derived-relation defs, integrity constraints, (pure) functions
  * Accidental state
     * Declarative spec of performance opts
     * State-related hints e.g. seldom used subset of attrs stored elsewhere
  * Other: User and sys interfaces

     * Highly app specific

  * Feeders
     * Convert input into relational assignments
     * Need some form of state manipulation language
  * Observers

     * Generate output in response to changes in vale of relvars
  * Infra

     * …for essential state

