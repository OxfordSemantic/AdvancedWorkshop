# 2.4 Named Graphs

Named Graphs offer a way to partition your data and are commonly used to group semantically similar data.

Data in named graphs is stored as quads, with the fourth entry representing the named graph.



## The default graph

The default graph is unlike named graphs in that it has no name and stores triples rather than quads.

Without specifying a named graph RDFox will reference the default graph.

Using a variable graph will **not** include the default graph as data stored there are triples, not quads.

Setting the [default-graph-name](https://docs.oxfordsemantic.tech/data-stores.html#default-graph-name-parameter) parameter on data store creation changes this behaviour.

## Named Graphs syntax

Named graphs can be accessed by including its name (or inserting a variable) after the desired triple pattern within the graph.

When writing rules, `[?S, ?O, ?O] :myNamedGraph` is used to access a specific graph and `[?S, ?O, ?O] ?G` will access all named graphs.

A slightly different syntax is used when writing queries, declaring the Named Graph before the triple pattern with the `GRAPH` keyword.

`GRAPH :myNamedGraph {?S ?P ?O}` or `GRAPH ?G {?S ?P ?O}`.

Rules and queries can reference several named graphs in both the head and body simultaneously.

## Example

The following example shows how the Default Graph and a Named Graph can be used in a single rule.

The Named Graph ':Personnel' contains information about staff for a financial institute and the Default Graph contains information about trades that have been executed:

```
# Personnel Info Named Graph
:PersonnelInfo {
    :p-001 a :StaffMember .
    :p-002 a :StaffMember .
}

# Default Graph
:t-001001 a :Transaction ;
    :executedBy :p-001 .
```
The rules infers information about the staff who are active traders:
```
[?personnel, a, :Trader] :PersonnelInfo :-
    [?personnel, a, :StaffMember] :PersonnelInfo,
    [?trade, :executedBy, ?personnel] .
```

We'll query for the results with the following:
```
SELECT ?personnel ?roll
WHERE {
    GRAPH :PersonnelInfo {?personnel a :StaffMember} .
    OPTIONAL { SELECT ?personnel ?activeTrader
        WHERE {
            VALUES ?activeTrader {:Trader}
            GRAPH :PersonnelInfo {?personnel a ?activeTrader} .
        }
    }
    BIND(COALESCE(?activeTrader, :StaffMember) AS ?roll)
}
```

## Run the script

Run `2_4-NamedGraphs/example/exScript.rdfox` to see the results of this rule.

### You should see...

========= Personnel Rolls =========
|?personnel	|?roll|
|----|----|
|:p-001 |:Trader |
|:p-002 |:StaffMember |

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fpersonnel%20%3Froll%0AWHERE%20%7B%0A%20%20%20%20GRAPH%20%3APersonnelInfo%20%7B%3Fpersonnel%20a%20%3AStaffMember%7D%20.%0A%20%20%20%20OPTIONAL%20%7B%20SELECT%20%3Fpersonnel%20%3FactiveTrader%0A%20%20%20%20%20%20%20%20WHERE%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20VALUES%20%3FactiveTrader%20%7B%3ATrader%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20GRAPH%20%3APersonnelInfo%20%7B%3Fpersonnel%20a%20%3FactiveTrader%7D%20.%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20BIND%28COALESCE%28%3FactiveTrader%2C%20%3AStaffMember%29%20AS%20%3Froll%29%0A%7D%0A).

## Where are Named Graphs relevant?

Named Graphs can be used in almost every use case as a way to partition data - sometimes it simply comes down to preference of the architect whether to use Named Graphs or another single-graph data structure. This is particularly relevant to:

### Access Control & Privacy

To restrict access to specific parts of the data, to separate data definitively, ensure no contamination of data, etc.

## Exercise

Complete the rule `2_4-NamedGraphs/incompleteRules.dlog` that the query below can be used to directly find traders who have breached their limits.

```
SELECT ?personnel ?limit ?trade ?tradeValue
WHERE {
    GRAPH :PersonnelInfo {
        ?personnel :inBreachOfRegulation true ;
            :hasMaxTradeRestriction ?limit.
    } .

    ?trade :executedBy ?personnel ;
        :hasValue ?tradeValue ;
        a :BreachOfRegulation .
}
```
Here is a representative sample of the data in `2_4-NamedGraphs/exercise/data.ttl`.
```
:PersonnelInfo {
    :p-001 a :Trader ;
        :hasMaxTradeRestriction 100000 .
    
    :p-002 a :StaffMember .
}

:t-000001 a :Transaction ;
    :executedBy :p-001 ;
    :hasValue 50000 .
```

### Hits & helpful resources

[Named Graphs syntax in RDFox](https://docs.oxfordsemantic.tech/reasoning.html#named-graphs-and-n-ary-relations)

### Check your work

Run the script below to verify the results.

`2_4-NamedGraphs/exercise/script.rdfox`

## You should see...

=== Traders in Trouble ===
|?personnel|?limit|?trade|?tradeValue|
|-----------|-------------|-------------|-------------|
|:p-075|	80000|	:t-075011|	115606|
|:p-041|	130000|	:t-041021|	185874|
|:p-043|	150000|	:t-043025|	222928|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fpersonnel%20%3Flimit%20%3Ftrade%20%3FtradeValue%0AWHERE%20%7B%0A%20%20%20%20GRAPH%20%3APersonnelInfo%20%7B%0A%20%20%20%20%20%20%20%20%3Fpersonnel%20%3AinBreachOfRegulation%20true%20%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%3AhasMaxTradeRestriction%20%3Flimit.%0A%20%20%20%20%7D%20.%0A%0A%20%20%20%20%3Ftrade%20%3AexecutedBy%20%3Fpersonnel%20%3B%0A%20%20%20%20%20%20%20%20%3AhasValue%20%3FtradeValue%20%3B%0A%20%20%20%20%20%20%20%20a%20%3ABeachOfRegulation%20.%0A%7D).

## BONUS: Tuples tables

RDFox actually stores data as lists of facts in containers called tuple tables.

Two in-memory tuple tables are created automatically upon creating a new data store, the `DefaultTriples` table that stores triples in the default graph and the `Quads` table that contains all named graphs quads.

Alongside others, such as a name, a defining feature of tuple tables are is their 'arity' - the number of columns they contain.

There are actually 3 types of tuple tables:
1. In-memory tuple tables (like DefaultTriples and Quads.)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;These are used to store RDF data. Additional in-memory tuple tables can be created of arity 1-4, or deleted to save memory if required (including those created automatically).

2. Built-in tuple tables

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;These cannot be modified and contain useful information or functional benefits. They include **SKOLEM**, **SHACL**, and **rule dependencies**, each of which we will cover in these exercises (if you've completed 2.3 you will have used the SKOLEM table already).

3. Data source tuple tables

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;These are virtual tables in that they do not themselves exist in-memory - they create a view that references non-RDF data sources (such as CSV, SQL, Solr, etc.) that have been registered by the user. This is covered in more detail in the date management section.

### Tuple table syntax

Accessing custom tuple tables is slightly different again, with rule atoms taking the form:

`TupleTableName(?S, ?P, ?O, ?Q, ...)`

and queries accessing them via the `TT` keyword:

`TT TupleTableName { ?S ?P ?O ?Q ...}`
