# 3.4 Property Paths

Property paths in SPARQL represent chains of the same relationship.

They can be very helpful in practice as they enable queries for paths of arbitrary length but they are computationally taxing meaning performance is often slow.

The same information can be encoded using recursive rules (see 3.3) which we can then be queried for efficiently.

## Example

The following example shows how a property path chain can be found between two nodes that aren't directly related.

Below is a social network, showing direct personal connections between people:

```
:Alice :knows :Bob .

:Bob :knows :Carlos .

:Carlos :knows :Diana .

:Walter :knows :Wendy .
```
The rules below will recursively identify all of the people in Alice's extended network, that is to say, all of the people who indirectly know Alice:
```
[?person, a, :MemberOfAlicesNetwork] :-
    [:Alice, :knows, ?person].

[?person2, a, :MemberOfAlicesNetwork] :-
    [?person1, :knows, ?person2],
    [?person1, a, :MemberOfAlicesNetwork].
```

## Run the script

Run `3_4-PropertyPaths/example/exScript.rdfox` to see the results of this rule.

### You should see...

=== Members of Alice's wider Network ===
|?person|	?networkMember|
|---|---|
|:Diana|	:MemberOfAlicesNetwork|
|:Carlos|	:MemberOfAlicesNetwork|
|:Bob|	:MemberOfAlicesNetwork|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fperson%20%3FnetworkMember%0AWHERE%20%7B%0A%20%20%20%20%3Fperson%20a%20%3FnetworkMember%20.%0A%7D).

## Exercise

Complete the rule `3_4-PropertyPaths/exercise/incompleteRules.dlog` so that the query below can be used to directly find all the paths to people in Alice's network and the path required get there:

```
SELECT ?endPerson ?hopsAwayFromAlice ?person
WHERE {

    ?pathNode :pathTo ?endPerson;
            :hasPathLength ?pathLength;
            :pathIncludes ?person .

    ?person :hopsAwayFromAlice ?hopsAwayFromAlice.

} ORDER BY DESC(?pathLength) ASC(?endPerson) ASC(?hopsAwayFromAlice)
LIMIT 15
```
Alice is very popular, so we've limited the results to 15!

Here is a representative sample of the data in `3_4-PropertyPaths/exercise/data.ttl`.
```
:Alice a :Person ;
    :knows :Jacqueline,
        :Melissa,
        :Troy .
```

### Hits & helpful resources

In this file there are 2 sets of recursive rules, the first set iterates forward along the property path to find all network members, the second must iterate backwards to find all the people who created that specific chain.

### Check your work

Run the script below to verify the results.

`3_4-PropertyPaths/exercise/script.rdfox`

## You should see...

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

### Visualise the results

The power in these results is much clearer when visualised.

Open them in the RDFox Explorer [here](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FendPerson%20%3Fperson%0AWHERE%20%7B%0A%0A%20%20%20%20%3FpathNode%20%3ApathTo%20%3FendPerson%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%3AhasPathLength%20%3FpathLength%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%3ApathIncludes%20%3Fperson%20.%0A%0A%20%20%20%20%3Fperson%20%3AhopsAwayFromAlice%20%3FhopsAwayFromAlice.%0A%0A%7D%20ORDER%20BY%20DESC%28%3FpathLength%29%20ASC%28%3FendPerson%29%20ASC%28%3FhopsAwayFromAlice%29) and **change the layout to Breadth First**.
