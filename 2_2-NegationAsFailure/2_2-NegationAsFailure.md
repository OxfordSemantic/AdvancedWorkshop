# 2.2 Negation as Failure

Negation as Failure (NAF) searches for an absence of data - inferring new facts where a pattern is missing.

### The Closed World Assumption

RDFox employs the Closed World Assumption - that is to say, anything that cannot be proven to be true is considered false.

Negation As Failure implements this assumption as missing data is treated the same as false data.

## NOT vs NOT EXISTS

There are two critical components to understand when using NAF with Datalog rules, **NOT** and **NOT EXISTS**.

Semantically, these two functions are remarkably similar but both uniquely powerful depending on the circumstance.

**NOT** and **NOT EXISTS** are both used to ensure a pattern does not exist within the data store.

All variables used in **NOT** must be bound elsewhere in the rule body, allowing specific patterns relating to specific variables to be considered.

On the other hand, **NOT EXISTS** introduces a new variable, or variables, that are not bound in the wider rule body, allowing more general patterns to be considered.

### Variable scope

Variables introduced by **NOT EXISTS** remain unbound when the rule-body infers new facts as the patterns don't exist.

Therefore, the scope of these variables must be local to the negation atom. They can share the names of a variables in the wider body without sharing a binding.

# Example

The following example highlights the different implications of **NOT** vs **NOT EXISTS** in the context of a network of IT assets that depend on one another.

The data represents devices that depend on a number of switches, giving us two options to model their dependency:

```
:switchA :hasState :on .

:switchB :hasState :off .

:device01 a :Component ;
    :dependsOn :switchA .

:device02 a :Component ;
    :dependsOn :switchB .

:device03 a :Component ;
    :dependsOn :switchA ;
    :dependsOn :switchB .

:device04 a :Component.
```

The rules use both **NOT** and **NOT EXISTS** to create those two dependency models:
```
[?device, :atLeastOneDependencyOn, true] :-
    [?device, a, :Component],
    [?device, :dependsOn, ?switch],
    NOT (
        [?switch, :hasState, :off] 
    ) .

[?device, :noDependenciesOff, true] :-
    [?device, a, :Component],
    NOT EXISTS ?switch IN (
        [?device, :dependsOn, ?switch],
        [?switch, :hasState, :off] 
    ) .
```

### Writing rules with Negation

Negation atoms must be accompanied by at least one non-negation atom directly in the rule body (excluding atoms within the negation atom itself), as is shown above.

In the case that the pattern is matched, this ensures that at least one variable is bound.

RDFox will return an error when this is not the case.

Here, we'll search for the inferred results with the following query:

```
SELECT ?device ?atLeastOneDependencyOn ?noDependenciesOff
WHERE {
    ?device a :Component .
    OPTIONAL {SELECT ?device ?atLeastOneDependencyOn
              WHERE {
                  ?device :atLeastOneDependencyOn ?atLeastOneDependencyOn .
              }
    }
    OPTIONAL {SELECT ?device ?noDependenciesOff
              WHERE {
                  ?device :noDependenciesOff ?noDependenciesOff .
              }
    }
} ORDER BY ASC(?device)
```

## Run the script

Run `2_2-NegationAsFailure/example/exScript.rdfox` to see the results of this rule.

### You should see...

|Device | At least one dependency on | No dependencies off|
|-----------|-------------|-------------|
|:device1| true | true |
|:device2| UNDEF | UNDEF |
|:device3| true | UNDEF |
|:device4| UNDEF | true |

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fdevice%20%3FatLeastOneDependencyOn%20%3FnoDependenciesOff%0AWHERE%20%7B%0A%20%20%20%20%3Fdevice%20a%20%3AComponent%20.%0A%20%20%20%20OPTIONAL%20%7BSELECT%20%3Fdevice%20%3FatLeastOneDependencyOn%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Fdevice%20%3AatLeastOneDependencyOn%20%3FatLeastOneDependencyOn%20.%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20OPTIONAL%20%7BSELECT%20%3Fdevice%20%3FnoDependenciesOff%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Fdevice%20%3AnoDependenciesOff%20%3FnoDependenciesOff%20.%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D%20ORDER%20BY%20ASC%28%3Fdevice%29).

## Incremental retraction

NAF is an incredibly powerful tool and raises an potentially thorny question when combined with the Incremental Reasoning of RDFox.

What happens when new data is added that fills the gap of an absent triple, whose absence was leading to the inference of another fact?

The previously inferred fact will have to be retracted.

Try importing some new data that contains something from the **NOT EXISTS** atom and see what happens.

Here, we'll introduce new dependencies for **device04** - now depending on **both switchA and switchB**:

```
:device04 a :Component;
    :dependsOn :switchA ;
    :dependsOn :switchB .
```

## Run the script

Ensuring you have run the first script...

Now run `2_2-NegationAsFailure/example/exScript2.rdfox` to add the new data and run the same query again.

### You should see...

|Device | At least one dependency on | No dependencies off|
|-----------|-------------|-------------|
|:device1| true | true |
|:device2| UNDEF | UNDEF |
|:device3| true | UNDEF |
|:device4| true | UNDEF |

## Stratification and Cycles in rules

A rule, or set of rules, cannot be imported if they create cyclic logic that involves negation or aggregation.

Some types of cycles are allowed and are in fact incredibly useful (see 3.2), but when negation or aggregation is involves, it can create a situation where the inferred triple no longer holds, and must be changed, only for that change to then re-infer the triple, which would then mean it no longer holds and so on.

Take the following rule as an example.

If a subject node is not a member of **Class**, it is inferred to be a be a member of **Class**, at which point, because it is now a member of **Class**, the rule will no longer hold and the fact that is is a member of **Class** will be retracted, re-entering the initial state and starting the process again.

```
[?x, a, :Class] :-
    [?x, a, ?c] ,
    NOT [?x, a, :Class] .
```

## Run the script

Run `2_2-NegationAsFailure/example/exScript3.rdfox` to see the stratification error created by this rule.

### You should see...
```
An error occurred while executing the command:
    The program is not stratified because these components of the dependency graph contain cycles through negation and/or aggregation:
    ======== COMPONENT 1 ========
        <https://rdfox.com/example#Class>[?x] :- rdf:type[?x, ?c], NOT <https://rdfox.com/example#Class>[?x] .
    -------- Rules in other components whose body atoms contribute to the cycles --------
        <https://rdfox.com/example#atLeastOneDependencyOn>[?device, true] :- <https://rdfox.com/example#Component>[?device], <https://rdfox.com/example#dependsOn>[?device, ?switch], NOT <https://rdfox.com/example#hasState>[?switch, <https://rdfox.com/example#off>] .
        <https://rdfox.com/example#noDependenciesOff>[?device, true] :- <https://rdfox.com/example#Component>[?device], NOT EXISTS ?switch IN (<https://rdfox.com/example#dependsOn>[?device, ?switch], <https://rdfox.com/example#hasState>[?switch, <https://rdfox.com/example#off>]) .
    ========================================================================================================================
```

### Stratification Error

This is known as a **stratification error**.

**Strata**, also called **components**, can be thought of as 'layers' that depend on one another.

When these layers form a loops, and the loop causes previously inferred facts to update with each loop (as with our NOT :Class example above), the rule set is not stratified **stratified** and RDFox will not accept it.

Loops in strata by themselves are not a problem, so long as it does not create changing facts. RDFox will notice when no new facts are inferred and stop the cycle.

### Allowed rule examples

```
[?y, :hasRelation, ?x] :-
    [?x, :hasRelation, ?y] .
```
While the logic is cyclic, facts will not be changed in each iteration.

```
[?x, :hasCount, ?newCount] :-
    [?x, :hasCount, ?count],
    BIND (?count + 1 AS ?newCount) .
```
While this rule creates an infinite loop, it does not cause old facts to be updated - just new to be added.

### What to do when encountering a stratification error?

RDFox will tell you which rule caused a stratification error to occur.

This rule will include either:

1. A negation atom in the body

2. An aggregate atom in the body

3. An atom in the head that is used in another rules negation atom

4. An atom in the head that is used in another rules aggregate atom

Atoms that depend on aggregation or negation are said to have a 'special dependency'.

One of these will be creating a cycle that you must remove.

If you still want to perform this special aggregate or negation on the existing data, without inferring new triples that will impact the special dependency, there is a way.

A parent relationship can be created to separate the loop from the special dependency. Consider the following:

```
[?x, :existingPredicate, ?y] :-
    [?x, :countPredicate, ?y].

[?x, :countPredicate, ?z] :-
    AGGREGATE(
    [?x, :existingPredicate, ?y]
    ON ?x
    BIND COUNT(?y) AS ?z).
```

The rules above cannot be stratified as they form a loop that involves an aggregate.

However, the loop can be extracted by introducing a parent relationship for `:existingPredicate`.

In the rules below, we have removed `:existingPredicate` from the loop, therefore making the rules stratify.

```
[?x, :newPredicate, ?y] :-
    [?x, :existingPredicate, ?y] .
 
[?x, :newPredicate, ?y] :-
    [?x, :countPredicate, ?y].

[?x, :countPredicate, ?z] :-
    AGGREGATE(
    [?x, :existingPredicate, ?y]
    ON ?x
    BIND COUNT(?y) AS ?z).
```

## Where is NAF relevant?

Negation as failure is a particularly powerful tool as the absence of data is a common concept in real-world applications.

### On-device

To encode real-world decision-making rules, map dependencies, repair data, etc.

### Publishing

To generate advanced recommendations, offer detailed search, control user access, etc.

### Construction and Manufacturing

To determine complex compatibilities, comply with schematics and regulations, simulate failure conditions, etc.

## Exercise

Complete the rule `2_2-NegationAsFailure/incompleteRules.dlog` so that the query below can be used to directly locate **devices** with an incomplete digital twin - **devices** that have no registered dependent switches and have not been decommissioned.

```
SELECT ?device
WHERE {

    ?device :hasIncompleteTwin true .

} ORDER BY ASC(?device)
```

Here is a representative sample of the data in `2_2-NegationAsFailure/exercise/data.ttl`.

```
:device001 a :Component ;
    :dependsOn :switchA .

:device002 a :Component ;
    :isDecommissioned true .

:switchA a :Switch ;
    :hasState :on .
```
### Hits & helpful resources

[Negation syntax forms](https://docs.oxfordsemantic.tech/reasoning.html#negation-as-failure)

### Check your work

Run the script below to verify the results.

`2_2-NegationAsFailure/exercise/script.rdfox`

## You should see...

=== Devices Requiring Attention ===
|?device|
|-----------|
|:device175|
|:device290|
|:device354|
|:device456|
|:device585|
|:device590|
|:device656|
|:device684|
|:device727|
|:device770|
|:device860|
|:device874|
|:device903|
|:device910|
|:device930|
|:device969|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fswitch%20%3FpassedMaintenanceCheck%0AWHERE%20%7B%0A%20%20%20%20%3Fswitch%20%3ArequiresMaintenance%20true%20.%0A%20%20%20%20%0A%20%20%20%20OPTIONAL%20%7BSELECT%20%3Fswitch%20%3FpassedMaintenanceCheck%0A%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Fswitch%20%3ApassedMaintenanceCheck%20%3FpassedMaintenanceCheck%20.%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%7D%20ORDER%20BY%20DESC%28%3FpassedMaintenanceCheck%29%20%3Fswitch).
