# 4.4 Property Paths

<br>

## 🔥 &nbsp; Why are Property Paths helpful?

This is why.

<br>
<br>

## 📖 &nbsp; What are Property Paths?

Property paths in SPARQL represent chains of the same relationship.

They can be very helpful in practice as they enable queries for paths of arbitrary length but they are computationally taxing meaning performance is often slow.

The same information can be encoded using recursive rules (see 3.3) which we can then be queried for efficiently.

<br>
<br>

## ⚡ &nbsp; Real world applications

These are some applications examples.

<br>

### Social Networks

To find indirectly connected members of an extended social network, to identify chains of people of a particular seniority of company, etc.

<br>

### Construction

To identify flaws or weaknesses in designs, to create a bill of materials, to isolate material components, etc.

<br>

### Retail

To monitor and stress-test supply chains, to analyse customer buying patterns and improve recommendations, etc.

<br>
<br>

## 🔬 &nbsp; Example

![Property Paths](../images/propertyPaths.png)

The following rules recursively detect members of Alice's network - including people she both directly and indirectly knows.

```
[?person, a, :MemberOfAlicesNetwork] :-
    [:Alice, :knows, ?person].

[?person2, a, :MemberOfAlicesNetwork] :-
    [?person1, :knows, ?person2],
    [?person1, a, :MemberOfAlicesNetwork].
```

Here is the data we'll be using to show this:

```
:Alice :knows :Bob .

:Bob :knows :Carlos .

:Carlos :knows :Diana .

:Walter :knows :Wendy .
```

<br>
<br>

## ✅ &nbsp; Check the results

Run `3_4-PropertyPaths/example/exScript.rdfox` to see the results of this rule.

<br>

### You should see...

=== Members of Alice's wider Network ===
|?person|	?networkMember|
|---|---|
|:Diana|	:MemberOfAlicesNetwork|
|:Carlos|	:MemberOfAlicesNetwork|
|:Bob|	:MemberOfAlicesNetwork|

<br>

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fperson%20%3FnetworkMember%0AWHERE%20%7B%0A%20%20%20%20%3Fperson%20a%20%3FnetworkMember%20.%0A%7D).

<br>
<br>


## 🚀 &nbsp; Exercise

Complete the rule `3_4-PropertyPaths/incompleteRules.dlog` to find all the paths to people in Alice's network and the path required get there.

Here is a representative sample of the data in `3_4-PropertyPaths/exercise/data.ttl`.
```
:Alice a :Person ;
    :knows :Jacqueline,
        :Melissa,
        :Troy .
```

<br>
<br>

## 📌 &nbsp; Hints & helpful resources

In this file there are 2 sets of recursive rules, the first set iterates forward along the property path to find all network members, the second must iterate backwards to find all the people who created that specific chain.

<br>
<br>

## ✅ &nbsp; Check your answers

Run `3_4-PropertyPaths/exercise/script.rdfox` to see the results of this rule.

<br>

### You should see...

=== How to Reach Alice's Network Members ===
|?endPerson|?hopsAwayFromAlice|?person|
|-----------|-------------|-------------|
|:Clifford	|0|	:Alice|
|:Clifford	|1|	:Melissa|
|:Clifford	|2|	:James|
|:Clifford	|3|	:Susan|
|:Clifford	|4|	:Clifford|
|:Eric	|0|	:Alice|
|:Eric	|1|	:Melissa|
|:Eric	|2|	:James|
|:Eric	|3|	:Susan|
|:Eric	|4|	:Eric|
|:Glen	|0|	:Alice|
|:Glen	|1|	:Melissa|
|:Glen	|2|	:James|
|:Glen	|3|	:Susan|
|:Glen	|4|	:Glen|

<br>

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FendPerson%20%3Fperson%0AWHERE%20%7B%0A%0A%20%20%20%20%3FpathNode%20%3ApathTo%20%3FendPerson%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%3AhasPathLength%20%3FpathLength%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%3ApathIncludes%20%3Fperson%20.%0A%0A%20%20%20%20%3Fperson%20%3AhopsAwayFromAlice%20%3FhopsAwayFromAlice.%0A%0A%7D%20ORDER%20BY%20DESC%28%3FpathLength%29%20ASC%28%3FendPerson%29%20ASC%28%3FhopsAwayFromAlice%29) and **change the layout to Breadth First**.


<br>
<br>
