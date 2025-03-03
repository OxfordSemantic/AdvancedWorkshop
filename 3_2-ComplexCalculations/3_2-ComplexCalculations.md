# 3.2 Complex Calculations

Reasoning can be used to perform complex calculations and analysis as `BIND` offers the flexibility to describing mathematical expressions using variables and in-build functions.

Complex calculations often require complex patterns, so layering rules can helpful - referring to patterns in one rule that were created by others.

This is very common in practice, not just in calculations, to construct intricate patterns over several rules - breaking the problem down into bit sized chunks.

This can also greatly benefit the efficiency of reasoning - some common examples are explored throughout the workshop.

## Example

Below is an example of how multiple rules can work together in order to calculate a result - in this case to perform Term Frequency analysis for articles with specific content tags in order to recommend similar articles.

Here is our article data:

```
:article1 a :Article ;
        :hasTag :AI ,
                :semanticReasoning .

:article2 a :Article ;
        :hasTag :semanticReasoning .

:article3 a :Article ;
        :hasTag :AI .

:article4 a :Article ;
        :hasTag :AI .
```

Our Term Frequency analysis rules assign common terms a lower weighting than rare terms that receive a much higher weighting.

When used to generate a recommendation, this is used to ensure common terms contribute less than specific terms that have a larger impact.

A node is created between two articles, calculating a similarity score based on shared weighted tags:

```
[?tag, :hasLogFrequency, ?termLogFrequency] :-
    AGGREGATE (
        [?article, :hasTag, ?tag]
        ON ?tag
        BIND COUNT(?article) AS ?totalTagMentions
    ),
    BIND (LOG(1/?totalTagMentions) AS ?termLogFrequency) .

[?recommendationNode, :hasRecommendedArticle, ?article1],
[?recommendationNode, :hasRecommendedArticle, ?article2],
[?recommendationNode, :pairHasSimilarityScore, ?similarityScore] :-
    AGGREGATE(
        [?article1, :hasTag, ?tag],
        [?article2, :hasTag, ?tag],
        [?tag, :hasLogFrequency, ?termLogFrequency]
        ON ?article1 ?article2
        BIND SUM(-1/?termLogFrequency) AS ?similarityScore
    ),
    SKOLEM(?article1,?article2,?recommendationNode),
    FILTER (?article1 > ?article2) .
```

We'll query for the results with the following:
```
SELECT ?article1 ?article2 ?similarityScore
WHERE {
    ?recommendationNode :hasRecommendedArticle ?article1 ;
        :hasRecommendedArticle ?article2 ;
        :pairHasSimilarityScore ?similarityScore .
    FILTER(?article1 < ?article2)
} ORDER BY ASC(?article1) DESC(?similarityScore)
```

## Run the script

Run `3_2-ComplexCalculations/example/exScript.rdfox` to see the results of this rule.

### You should see...

=== Similar Articles ===
|?article1	|?article2|	?similarityScore|
|---|---|---|
|:article1|	:article2|	1.4426950408889634e+0|
|:article1|	:article4|	9.1023922662683732e-1|
|:article1|	:article3|	9.1023922662683732e-1|
|:article3|	:article4|	9.1023922662683732e-1|

## info rulestats

Run `info rulestats` in the RDFox shell to show information about the rules in your data store.

For this example, so far, you will see:

=================== RULES STATISTICS =====================
|Component|    Nonrecursive rules|    Recursive rules|    Total rules|
|---------|----------------------|-------------------|---------------|
|        1|                     1|                  0|              1|
|        2|                     1|                  0|              1|
|Total:|                        2|                  0|              2|

We have imported two rules above, so the total rules is 2.

Both are non-recursive (see 3.3 for recursion) which is also reflected in the table.

Since one rule depends on the other, we end up with 2 components (also called layers, or strata)

This can be a great launching point for debugging rules, particularly if the values shown are not what you expect.

Find out more about the `info` command in the docs [here](https://docs.oxfordsemantic.tech/rdfox-shell.html#info).

## Unnecessarily complex rules

It can be tempting to write huge rules that match sprawling patterns and infer many heads at once, but this can be deeply inefficient.

Rules should be streamlined, only considering the minimum number of body atoms required to infer the relevant head facts, otherwise computation will be spent matching irrelevant body patterns for some head atoms.

This often means splitting large rules into several parts.

Take these rules for examples:

```
[?a, :hasNewProp, "new prop"],
[?x, a, :newClass] :-
    [?a, :hasProp, "prop" ],
    [?x, a, :Class ].
```

Although we can write this as one rule, there is no relation between the atoms involving ?a and ?x.

```
[?a, :hasNewProp, "new prop"]:-
    [?a, :hasProp, "prop" ].

[?x, a, :newClass] :-
    [?x, a, :Class ].
```
Here, we're split that rule into two rules that results in the same facts inferred.

We'll use this simple data to show how the rule sets perform differently:
```
:a :hasProp "prop" .

:x a :Class .
```

## Run the script

Run `3_2-ComplexCalculations/example/exScript2.rdfox` to see the performance of the combined rule...

### You should see...

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             6|           1|            1|          (0) :hasNewProp[?a, "new prop"], :newClass[?x] :- :hasProp[?a, "prop"], :Class[?x] .|
|   |    |      |             |             |           |            1|          (1) :hasNewProp[?a, "new prop"], :newClass[?x] :- :hasProp[?a, "prop"], :Class[?x] .|

Notice that the reasoning profiles undergoes 6 iterator operations. This is because the combo rule creates a cross product of patterns that it must check to infer each head atom, despite being unrelated.

## Run the script

Now run `3_2-ComplexCalculations/example/exScript3.rdfox` to see the performance of the two separate rules.

### You should see...

1

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             2|           1|            1|          (0) :newClass[?x] :- :Class[?x] .|

2

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             2|           1|            1|          (0) :hasNewProp[?a, "new prop"] :- :hasProp[?a, "prop"] .|

This time, notice that even though there are more rules imported, the rules only require 2 iterator operations each, for a total of 4.

### A problem of scale

While 6 iterations rather than 4 is almost invisible, this problem scales poorly with the number of triples and complexity of the rule and can cause drastic rules slowdowns.

## Exercise

Complete the rule `3_2-ComplexCalculations/incompleteRules.dlog` so that the query below will return the percentage of articles each tag appears in, rounded to the nearest percentage point:

```
SELECT ?percentage ?tag
WHERE {
    ?tag :mentionedInPercentageOfArticles ?percentage .
    FILTER (?percentage > 1)
} ORDER BY DESC (?percentage)
```

Here is a representative sample of the data in `3_2-ComplexCalculations/exercise/data.ttl`:

```
:blogA :containsArticle :article001,
                :article002 .

:article001 a :Article ;
        :hasTag :AI ,
                :semanticReasoning .
```

### Hits & helpful resources

[Mathematical functions in RDFox](https://docs.oxfordsemantic.tech/querying.html#mathematical-functions)

### Check your work

Run the script below to verify the results.

`3_2-ComplexCalculations/exercise/script.rdfox`

## You should see...

=== Percentage of articles mentioning tags ===
|?percentage|?tag|
|-----------|-------------|
|16.0|	:SemanticReasoning|
|14.0|	:Technology|
|12.0|	:KnowledgeRepresentation|
|10.0|	:RDFox|
|8.0|	:Datalog|
|7.0|	:LLMs|
|7.0|	:AI|

## BONUS: Transactions

A transaction is the window in which RDFox performs one or several read and/or write operations, ending when the operations are totally complete and the data store is self-consistent.

Without specifying otherwise, a transaction will be created when any command is executed and will automatically close when its function has been achieved.

However, transactions can be manually opened with `begin` and closed with `commit` or `rollback` depending on whether the results of the transaction should be committed or discarded.

Multiple rule files can be imported in one transaction. This can be much more efficient if the rules interact with one another as RDFox computes an optimized order in which to import the rules.

Eg.

`begin`\
`import rule1.dlog`\
`import rule2.dlog`\
`commit`
