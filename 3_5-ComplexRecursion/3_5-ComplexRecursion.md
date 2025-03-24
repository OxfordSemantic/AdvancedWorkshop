# 3.5 Complex Recursion

Recursive rules can be incredible powerful, able to be taken even further than the examples shown in 3.3 & 3.4 by incorporating calculations.

Rules like these can be used to solve traditionally challenging data problems such as the bill of materials, where an aggregated cost must be calculated for hierarchical parts.

<br>
<br>

## 🔬 &nbsp; Example

Critically for this example, we want to calculate the aggregated cost at each point in the tree, not just the highest level

This is the most conceptually complex example in this workshop, so it's important to work up one layer at a time.

First lets look at some example data of compound parts with associated prices:

```
:compoundA :contains :compoundB,
                :part1 .

:part1 :hasCost 10 .

:compoundB :contains :part2,
                :part3 ,
                :part4 .

:part2 :hasCost 1 .
:part3 :hasCost 1 .
:part4 :hasCost 1 .
```

<br>
<br>

## ✅ &nbsp; Visualise the data

It can help to visualise this problem, so run the command `3_5-ComplexRecursion/example/exScript.rdfox` to import the data...

then click here to [explore the data](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FS%20%3FP%20%3FO%0AWHERE%20%7B%0A%20%20%20%20%3FS%20%3FP%20%3FO%0A%7D) and change the layout to **Breadth First**.

<br>
<br>

## 🔍 &nbsp; Why is this a problem?

The challenge here is that we wish to calculate the aggregate cost at each compound point - both for the collections of parts, and for the collection of sub-compounds.

However, RDFox will not accept aggregates (or negation) that rely on recursive results (see 2.2), so we cannot write a simple recursive rule set.

To demonstrate that, below is a rule that might naively be written to solve this.

```
[?compound, :hasCost, ?totalCost] :-
    AGGREGATE(
        [?compound, :contains, ?thing],
        [?thing, :hasCost, ?cost]
        ON ?compound
        BIND SUM(?cost) AS ?totalCost
    ).
```

<br>
<br>

## 🚫 &nbsp; Check the error

When trying to import this rule with `import 3_5-ComplexRecursion/example/exRules.dlog`, RDFox will throw a stratification error.

<br>

### You should see...

Adding data in file '3_5-ComplexRecursion/example/exRules.dlog'.
An error occurred while executing the command:
    The program is not stratified because these components of the dependency graph contain cycles through negation and/or aggregation:
    ======== COMPONENT 1 ========
        <https://rdfox.com/example#hasCost>[?compound, ?totalCost] :- AGGREGATE(<https://rdfox.com/example#contains>[?compound, ?thing], <https://rdfox.com/example#hasCost>[?thing, ?cost] ON ?compound BIND SUM(?cost) AS ?totalCost) .
    ========================================================================================================================

<br>
<br>

## 🔥 &nbsp; Solving the Bill of Materials

To get around this, we must perform the aggregates in a deterministic, stratified way - we must state the order in which it will occur.

One way to do this is to create an order for the parts and components, and use `BIND` to sum the running total from one part to the next. This way no recursive rules incorporate an aggregate.

<br>
<br>

## 🔬 &nbsp; Ordering entities for an arbitrary compound node

To order arbitrary nodes, we first have to find each node's relative position to one another (whether it is before or after), then give them a specific position within the list by putting node next to one another where there is no other node in between:

```
[?thing1, :hasGreaterThing, ?thing2] :-
    [?compound, :contains, ?thing1],
    [?compound, :contains, ?thing2],
    FILTER (?thing2 > ?thing1).

[?thing1, :hasNextThing, ?thing2] :-
    [?thing1, :hasGreaterThing, ?thing2],
    NOT EXISTS ?thing3 IN ( 
        [?thing1, :hasGreaterThing, ?thing3], 
        [?thing3, :hasGreaterThing, ?thing2]
    ).
```

<br>
<br>

## ✅ &nbsp; Check the results

Run `import 3_5-ComplexRecursion/example/exRules2.dlog` to see the results of these rules.

[View the results of these rules here](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FS%20%3FP%20%3FO%0AWHERE%20%7B%0A%20%20%20%20%3FS%20%3FP%20%3FO%0A%7D) and **change the layout to Breadth first**.

**Use the bulb to highlight the facts that have been inferred.**

<br>
<br>

## 🔬 &nbsp; Finding the first and last members of the chain

Next we need to identify the first and last members of the chains we've created:

```
[?compound, :hasFirstThing, ?firstThing] :-
    [?compound, :contains, ?firstThing],
    NOT EXISTS ?something IN ([?something, :hasGreaterThing, ?firstThing]).

[?compound, :hasLastThing, ?lastThing] :-
    [?compound, :contains, ?lastThing],
    NOT EXISTS ?something IN ([?lastThing, :hasGreaterThing, ?something]).
```

It may not be immediately clear why we do this, but they will both be helpful in the final step.

<br>
<br>

## ✅ &nbsp; Check the results

Run `import 3_5-ComplexRecursion/example/exRules3.dlog` to see the results of these rules.

[View the results of these rules here](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FS%20%3FP%20%3FO%0AWHERE%20%7B%0A%20%20%20%20%3FS%20%3FP%20%3FO%0A%7D) and **change the layout to Breadth first**.

Use the bulb to highlight the facts that have been inferred.

<br>
<br>

## 🔬 &nbsp; Calculating a running total

Now that all the pieces are in place, we can recursively calculate a running total.

```
[?firstThing, :hasRunningTotal, ?cost] :-
    [?compound, :hasFirstThing, ?firstThing],
    [?firstThing, :hasCost, ?cost].

[?thing2, :hasRunningTotal, ?runningTotal] :-
    [?thing1, :hasNextThing, ?thing2],
    [?thing1, :hasRunningTotal, ?oldRunningTotal],
    [?thing2, :hasCost, ?cost],
    BIND (?oldRunningTotal + ?cost AS ?runningTotal).

[?compound, :hasCost, ?compoundTotal] :-
    [?compound, :hasLastThing, ?lastThing],
    [?lastThing, :hasRunningTotal, ?compoundTotal].
```

The running count begins at the first thing in the compound and is passed up to the compound node once it has reached the last thing.

<br>
<br>

## ✅ &nbsp; Check the results

Run `import 3_5-ComplexRecursion/example/exRules4.dlog` to see the results of these rules.

[View the results of these rules here](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FS%20%3FP%20%3FO%0AWHERE%20%7B%0A%20%20%20%20%3FS%20%3FP%20%3FO%0A%7D) and **change the layout to Breadth first**.

Use the bulb to highlight the facts that have been inferred.

<br>
<br>

## 👏 &nbsp; Congratulations!

You've solved the bill of materials - not a simple task!

<br>
<br>

## 🚀 &nbsp; Bonus: Explainability

As complex as the emerging results may be, rules simply offer a series of logical inferences, and this means that every result is explainable.

We saw simple example of this in 3.1, but this proof tree is significantly more intricate.

[Click here](http://localhost:12110/console/datastores/explain?datastore=default&fact=%3AhasCost%5B%3AcompoundA%2C%2013%5D) see the explanation of the cost of `compoundA`.

Continue the discussion with others in the `RDFox-Workshop` channel of our [Slack Community](https://join.slack.com/t/rdfox/shared_invite/zt-1z7dnm2ad-WoKRf~~3CynB_KTi5X0RHg)!
