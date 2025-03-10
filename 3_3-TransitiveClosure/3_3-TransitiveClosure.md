# 3.3 Transitive Closure

<br>

## 🔥 &nbsp; Why is Transitive Closure helpful?

Ever needed to find all nodes that are connected by an arbitrarily long repeating pattern, be that a chain of the same relationships or a more complex shape?

Transitive Closure is what you need!

<br>
<br>

## 📖 &nbsp; What is Transitive Closure?

Recursive rules are self referential to some degree, meaning the facts the infer may create the conditions for additional facts to be inferred.

This iterative cycle will continue until the body conditions of the rules involved are no longer met.

Recursion is the process used in identifying property paths. The rules make their way along the path, incorporating one new link at a time.

<br>
<br>

## ⚡ &nbsp; Real world applications

Recursion is a powerful and advanced tool with specific uses and yet is widely used as many applications would be impossible without it.

<br>

### Network Management

To find arbitrarily long routes between all possible pairs of locations, to determine viable paths where connections between individual nodes are restricted, etc.

<br>

### Finance

To detect chains of fraudulent behavior, to trace money as it is transferred between several accounts, etc.

<br>

### Manufacturing

To model dependencies and availability, to test failure modes and plan maintenance, etc.

<br>
<br>

## 🔬 &nbsp; Example

![Transitive Closure](../images/transitiveClosure.png)

The following rules recursively traverse a network of IT assets to determine each of their direct and indirect dependencies.

```
[?asset1, :hasTransitiveDependency, ?asset2] :-
   [?asset1, :dependsOn, ?asset2] .

[?asset1, :hasTransitiveDependency, ?asset3] :-
   [?asset1, :hasTransitiveDependency, ?asset2],
   [?asset2, :dependsOn, ?asset3] .
```

Here is the data we'll be using to show this:

```
:webServer :dependsOn :machine .

:machine :dependsOn :network ,
                    :powerSupply .
```

<br>
<br>

## ✅ &nbsp; Check the results

Run `3_3-TransitiveClosure/example/exScript.rdfox` to see the results of these rules.

<br>

### You should see...

=== Assets whose function is dependent on others ===
|?upstreamAsset| ?transitiveRelationship|	?downstreamAsset|
|---|---|---|
|:webServer|	:hasTransitiveDependency|	:machine|
|:webServer|	:hasTransitiveDependency|	:network|
|:webServer|	:hasTransitiveDependency|	:powerSupply|
|:machine|	:hasTransitiveDependency|	:network|
|:machine|	:hasTransitiveDependency|	:powerSupply|

<br>

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FupstreamAsset%20%3FP%20%3FdownstreamAsset%0AWHERE%20%7B%0A%20%20%20%20VALUES%20%3FP%20%7B%3AhasTransitiveDependency%7D%0A%20%20%20%20%3FupstreamAsset%20%3FP%20%3FdownstreamAsset%0A%7D%20ORDER%20BY%20DESC%28%3FupstreamAsset%29%20ASC%28%3FdownstreamAsset%29).

<br>
<br>

## ℹ️ &nbsp; Syntax helper

The first rule above is not recursive, it acts as an anchor that provides stable starting conditions.

The second is the recursive rules. It provides conditions for continual looping able to iterate again and again as new data is added by the previous iteration.

While not every application uses this setups, it is common for recursive rule sets to contain an anchor as it restricts inferences to just those that are determined necessary. Recursion has the potential to rapidly expand the size of the dataset if not used carefully.

<br>
<br>

## 🔍 &nbsp; End entities

It can be valuable to know the first or last member of a chain.

By utilising negation (see 2.2) this becomes trivial - we simply look for there not to exist a proceeding or succeeding member of the chain respectively.

The following rule identifies the web server as a top level asset.

```
[?asset1, a, :TopLevelAsset] :-
    [?asset1, :hasTransitiveDependency, ?asset2] ,
    NOT EXISTS ?asset0 IN ( [?asset0, :hasTransitiveDependency, ?asset1] ) .
```

<br>
<br>

## ✅ &nbsp; Check the results

Ensuring you have run the first script...

now run `3_3-TransitiveClosure/example/exScript2.rdfox` to see the results of this rule.

<br>

### You should see...

=== Top Level Assets ===
|?topLevelAsset|
|---|
|:webServer|

<br>
<br>

## 🔍 &nbsp; Cyclic relationships

Recursion can create cyclic relationships, either intentionally or otherwise.

In both cases it can be important to know when and where they exist.

Cyclic relationships that have been created through recursion are very simple to detect - we can simply look for an entity that belongs to its own chain.

The following rule identifies assets that belong to a cycle.

```
[?asset, a, :CyclicAsset] :-
    [?asset, :hasTransitiveDependency, ?asset].
```

To create a cycle, we must also add the following data.

```
:network :dependsOn :webServer .
```

<br>
<br>

## ✅ &nbsp; Check the results

Ensuring you have run the first script...

now run `3_3-TransitiveClosure/example/exScript3.rdfox` to see the results of this rule.

<br>

### You should see...

=== Cycles ===
|?cyclicAsset|	?downstreamAsset|
|---|---|
|:machine|	:network|
|:network|	:webServer|
|:webServer|	:machine|

<br>

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FcyclicAsset%20%3FdownstreamAsset%0AWHERE%20%7B%0A%20%20%20%20%3FcyclicAsset%20a%20%3ACyclicAsset%20%3B%0A%20%20%20%20%20%20%20%20%3AdependsOn%20%3FdownstreamAsset%20.%0A%20%20%20%20%0A%20%20%20%20%3FdownstreamAsset%20a%20%3ACyclicAsset%20.%0A%7D%20ORDER%20BY%20ASC%28%3FcyclicAsset%29).


<br>
<br>

## 🔍 &nbsp; Efficient transitivity rules

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

## 👀 &nbsp; Run the profiler

Run `3_3-TransitiveClosure/example/exScript4.rdfox` to see the performance of the anchored rule set...

### You should see...

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    Addition|      0|             55|             310|           45|            45|          (0) :dependsTransitively[?x, ?z] :- :dependsTransitively[?x, ?y], :dependsOn[?y, ?z] .|
|   2|    RuleAdd|      0|             1|             11|           10|            10|          (0) :dependsTransitively[?x, ?y] :- :dependsOn[?x, ?y] .|
|   3|    RuleAdd|      0|             1|             2|           0|            0|          (0) :dependsTransitively[?x, ?z] :- :dependsTransitively[?x, ?y], :dependsOn[?y, ?z] .|

Notice that the reasoning profiles undergoes 310 iterator operations to import 45 fresh facts in the **Addition** phase (the only significantly different phase).

## 👀 &nbsp; Run the profiler

Run `3_3-TransitiveClosure/example/exScript5.rdfox` to see the performance of the anchored rule set...

### You should see...

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    Addition|      0|             90|             672|           156|            36|          (0) :dependsOn[?x, ?z] :- :dependsOn[?x, ?y], :dependsOn[?y, ?z] .|
|   1|    RuleAdd|      0|             1|             40|           9|            9|          (0) :dependsOn[?x, ?z] :- :dependsOn[?x, ?y], :dependsOn[?y, ?z] .|


This time, notice that even though fewer facts were introduced (36 as the previous rule created some additional connections with the new relationship), the iterator when though more than double the operations at 672!

We can see why if we consider all of the ways to create infer the same facts - with the second rule set there will be large amounts of repetition.

<br>
<br>

## 🚀 &nbsp; Exercise

Complete the rule `3_3-TransitiveClosure/exercise/incompleteRules.dlog` to calculate the transitive dependencies between just primary assets - that is, assets that have a `:backupPriority "Primary"`.

Here is a representative sample of the data in `3_3-TransitiveClosure/exercise/data.ttl`.

```
:webServer1 a :WebServer ;
    :backupPriority "Primary" ;
    :dependsOn :machine1 ;
    :hasBackup :webServer2 .

:webServer2 a :WebServer ;
    :backupPriority "Secondary" ;
    :dependsOn :machine2 .

:machine1 a :ServerMachine ;
    :backupPriority "Primary" ;
    :dependsOn :network1 ,
    :powerSupply1 .

:machine2 a :ServerMachine ;
    :backupPriority "Secondary" ;
    :dependsOn :network2 ,
    :powerSupply2 .
```

<br>
<br>

## ✅ &nbsp; Check your answers

Run `3_3-TransitiveClosure/exercise/script.rdfox` to see the results of this rule.

<br>

### You should see...

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

<br>

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FupstreamAsset%20%3FdownstreamAsset%0AWHERE%20%7B%0A%20%20%20%20%3FupstreamAsset%20%3AhasPrimaryDependency%20%3FdownstreamAsset%0A%7D%20ORDER%20BY%20DESC%28%3FupstreamAsset%29%20ASC%28%3FdownstreamAsset%29) and **change the layout to Circle**.

<br>
<br>

## 👏 &nbsp; Bonus exercise

Write a set of recursive rules that 

Write a query [in the console](http://localhost:12110/console/datastores/sparql?datastore=default) to validate you work.
