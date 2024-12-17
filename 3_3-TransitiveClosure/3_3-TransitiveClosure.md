# 3.3 Transitive Closure & Recursion

Recursive rules are self referential to some degree, meaning the facts the infer may create the conditions for additional facts to be inferred.

This iterative cycle will continue until the body conditions of the rules involved are no longer met.

Recursion is the process used in identifying property paths. The rules make their way along the path, incorporating one new link at a time.

A common use of recursion, transitive closure directly connects all nodes that are joined by any repeating pattern downstream, be that a chain of the same relationships or a more complex shape.

## Example

The following example shows how recursion can be used to traverse a dependency network of AI assets, with the aim of finding all the things that support the operation of each individual service.

Here is IT asset network data we'll be using.

```
:webServer a :WebServer ;
    :dependsOn :machine .

:machine a :StorageMachine ;
    :dependsOn :network ,
        :powerSupply .

:network a :Network .

:powerSupply a :PowerSupply .
```
And here are the two recursive rules needed to compute the network's transitive closure:
```
[?asset1, :dependsTransitively, ?asset2] :-
   [?asset1, :dependsOn, ?asset2] .

[?asset1, :dependsTransitively, ?asset3] :-
   [?asset1, :dependsTransitively, ?asset2],
   [?asset2, :dependsOn, ?asset3] .
```
### Writing recursive rule sets

The first rule above is not recursive, it acts as an anchor that provides stable starting conditions.

The second is the recursive rules. It provides conditions for continual looping able to iterate again and again as new data is added by the previous iteration.

While not every application uses this setups, it is common for recursive rule sets to contain an anchor as it restricts inferences to just those that are determined necessary. Recursion has the potential to rapidly expand the size of the dataset if not properly contained.

Here, we'll query for the results with the following:

```
SELECT ?upstreamAsset ?downstreamAsset
WHERE {
    ?upstreamAsset :dependsTransitively ?downstreamAsset
} ORDER BY DESC(?upstreamAsset) ASC(?downstreamAsset)
```

## Run the script

Run `3_3-TransitiveClosure/example/exScript.rdfox` to see the results of these rules.

### You should see...

=== Assets whose function is dependent on others ===
|?upstreamAsset|	?downstreamAsset|
|---|---|
|:webServer|	:machine|
|:webServer|	:network|
|:webServer|	:powerSupply|
|:machine|	:network|
|:machine|	:powerSupply|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FupstreamAsset%20%3FdownstreamAsset%0AWHERE%20%7B%0A%20%20%20%20%3FupstreamAsset%20%3AdependsTransitively%20%3FdownstreamAsset%0A%7D%20ORDER%20BY%20DESC%28%3FupstreamAsset%29%20ASC%28%3FdownstreamAsset%29).

## End entities

It can be valuable to know the first or last member of a chain and with negation (see 2.2) this becomes trivial - we simply look for there not to exist a proceeding or succeeding member of the chain respectively.

Below is a rule that identifies the web server as a top level asset:

```
[?asset1, a, :TopLevelAsset] :-
    [?asset1, :dependsOn, ?asset2] ,
    NOT EXISTS ?asset0 IN ( [?asset0, :dependsOn, ?asset1] ) .
```

And query for it with the following:

```
SELECT ?topLevelAsset
WHERE {
    ?topLevelAsset a :TopLevelAsset
}
```

## Run the script

Ensuring you have run the first script...

now run `3_3-TransitiveClosure/example/exScript2.rdfox` to see the results of this rule.

### You should see...

=== Top Level Assets ===
|?topLevelAsset|
|---|
|:webServer|

## Cyclic relationships

Recursion can create cyclic relationships, either intentionally or otherwise, and either way it can be important to know when and where they exist.

Cyclic relationships that have been created through recursion are very simple to detect - we can simply look for an entity that belongs to its own chain.

Below is a rule that does just that:

```
[?asset, a, :CyclicAsset] :-
    [?asset, :dependsTransitively, ?asset].
```

To make this rule infer anything, we need to create a cycle with this small piece of data:

```
:network :dependsOn :webServer .
```

And we will identify assets in a cycle with the query below:
```
SELECT ?cyclicAsset ?downstreamAsset
WHERE {
    ?cyclicAsset a :CyclicAsset ;
        :dependsOn ?downstreamAsset .
    
    ?downstreamAsset a :CyclicAsset .
} ORDER BY ASC(?cyclicAsset)
```

## Run the script

Ensuring you have run the first script...

now run `3_3-TransitiveClosure/example/exScript3.rdfox` to see the results of this rule.

### You should see...

=== Cycles ===
|?cyclicAsset|	?downstreamAsset|
|---|---|
|:machine|	:network|
|:network|	:webServer|
|:webServer|	:machine|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FcyclicAsset%20%3FdownstreamAsset%0AWHERE%20%7B%0A%20%20%20%20%3FcyclicAsset%20a%20%3ACyclicAsset%20%3B%0A%20%20%20%20%20%20%20%20%3AdependsOn%20%3FdownstreamAsset%20.%0A%20%20%20%20%0A%20%20%20%20%3FdownstreamAsset%20a%20%3ACyclicAsset%20.%0A%7D%20ORDER%20BY%20ASC%28%3FcyclicAsset%29).

## Efficient transitivity rules

There is more than one way to write recursive rules that achieve the same result.

As we have above, we can either anchor the recursive rule with an existing relationship:

```
   [?x, :dependsTransitively, ?y] :-
      [?x, :dependsOn, ?y] .
   
   [?x, :dependsTransitively, ?z] :-
      [?x, :dependsTransitively, ?y] ,
      [?y, :dependsOn, ?z] .
```

Or, as is often instinctive, just create a purely transitive property that relies only on itself:

```
[?x, :dependsTransitively, ?y] :-
    [?x, :dependsOn, ?y] .

[?x, :dependsTransitively, ?z] :-
    [?x, :dependsTransitively, ?y],
    [?y, :dependsTransitively, ?z] .
```

These rule will end up considering many more facts than the previous one, making it significantly less efficient.

We'll use this simple data to show how the rule sets perform differently:

```
:a :dependsOn :b .
:b :dependsOn :c .
:c :dependsOn :d .
:d :dependsOn :e .
:e :dependsOn :f .
:f :dependsOn :g .
:g :dependsOn :h .
:h :dependsOn :i .
:i :dependsOn :j .
:j :dependsOn :k .
```

## Run the script

Run `3_3-TransitiveClosure/example/exScript4.rdfox` to see the performance of the anchored rule set...

### You should see...

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    Addition|      0|             55|             310|           45|            45|          (0) :dependsTransitively[?x, ?z] :- :dependsTransitively[?x, ?y], :dependsOn[?y, ?z] .|
|   2|    RuleAdd|      0|             1|             11|           10|            10|          (0) :dependsTransitively[?x, ?y] :- :dependsOn[?x, ?y] .|
|   3|    RuleAdd|      0|             1|             2|           0|            0|          (0) :dependsTransitively[?x, ?z] :- :dependsTransitively[?x, ?y], :dependsOn[?y, ?z] .|

Notice that the reasoning profiles undergoes 310 iterator operations to import 45 fresh facts in the **Addition** phase (the only significantly different phase).

## Run the script

Run `3_3-TransitiveClosure/example/exScript5.rdfox` to see the performance of the anchored rule set...

### You should see...

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    Addition|      0|             90|             672|           156|            36|          (0) :dependsOn[?x, ?z] :- :dependsOn[?x, ?y], :dependsOn[?y, ?z] .|
|   1|    RuleAdd|      0|             1|             40|           9|            9|          (0) :dependsOn[?x, ?z] :- :dependsOn[?x, ?y], :dependsOn[?y, ?z] .|


This time, notice that even though fewer facts were introduced (36 as the previous rule created some additional connections with the new relationship), the iterator when though more than double the operations at 672!

We can see why if we consider all of the ways to create infer the same facts - with the second rule set there will be large amounts of repetition.

## Exercise

Complete the rule `3_3-TransitiveClosure/exercise/incompleteRules.dlog` so that the query below can be used to directly find the transitive dependencies between just primary assets - that is, assets that have a `"Primary"` `:backupPriority`:

```
SELECT ?upstreamAsset ?noOfPrimaryDependencies ?downstreamAsset
WHERE {
    ?upstreamAsset :hasPrimaryDependency ?downstreamAsset

    # Here we count the number of dependencies to return an ordered hierarchical list
    {
        SELECT ?upstreamAsset (COUNT(?downstreamAsset) AS ?noOfPrimaryDependencies)
        WHERE{
            ?upstreamAsset :hasPrimaryDependency ?downstreamAsset 
        } GROUP BY ?upstreamAsset
    }
    
} 
ORDER BY DESC(?noOfPrimaryDependencies) DESC(?upstreamAsset) ASC(?downstreamAsset)
```
Here is a representative sample of the data in `3_3-TransitiveClosure/exercise/data.ttl`.

```
:webServer1 a :WebServer;
    :backupPriority "Primary";
    :dependsOn :machine1 ;
    :hasBackup :webServer2 .

:webServer2 a :WebServer;
    :backupPriority "Secondary";
    :dependsOn :machine2 .

:machine1 a :ServerMachine;
    :backupPriority "Primary";
    :dependsOn :network1, :powerSupply1 .

:machine2 a :ServerMachine;
    :backupPriority "Secondary";
    :dependsOn :network2, :powerSupply2 .
```

### Check your work

Run the script below to verify the results.

`3_3-TransitiveClosure/exercise/script.rdfox`

## You should see...

=== Primary Assets Transitive Dependencies ===
|?upstreamAsset|	?noOfPrimaryDependencies|	?downstreamAsset|
|-----------|-------------|-------------|
|:onlineStoreSystem	|9|	:machine1|
|:onlineStoreSystem	|9|	:machine5|
|:onlineStoreSystem	|9|	:machine9|
|:onlineStoreSystem	|9|	:onlineStoreApplicationServer1|
|:onlineStoreSystem	|9|	:onlineStoreApplicationService|
|:onlineStoreSystem	|9|	:onlineStoreDataBase1|
|:onlineStoreSystem	|9|	:onlineStoreDatabaseService|
|:onlineStoreSystem	|9|	:webServer1|
|:onlineStoreSystem	|9|	:webServerService|
|:webServerService	|2|	:machine1|
|:webServerService	|2|	:webServer1|
|:onlineStoreDatabaseService	|2|	:machine9|
|:onlineStoreDatabaseService	|2|	:onlineStoreDataBase1|
|:onlineStoreApplicationService	|2|	:machine5|
|:onlineStoreApplicationService	|2|	:onlineStoreApplicationServer1|
|:webServer1	|1|	:machine1|
|:onlineStoreDataBase1	|1|	:machine9|
|:onlineStoreApplicationServer1	|1|	:machine5|

### Visualise the results

The power in these results is much clearer when visualised.

Open them in the RDFox Explorer [here](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FupstreamAsset%20%3FdownstreamAsset%0AWHERE%20%7B%0A%20%20%20%20%3FupstreamAsset%20%3AhasPrimaryDependency%20%3FdownstreamAsset%0A%7D%20ORDER%20BY%20DESC%28%3FupstreamAsset%29%20ASC%28%3FdownstreamAsset%29) and **change the layout to Circle**.
