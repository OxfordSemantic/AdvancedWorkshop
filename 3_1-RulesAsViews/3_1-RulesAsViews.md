# 3.1 Rules as Views

One way to see rules is to think of them as creating views.

All rules can be thought of in this way but some can be used to create more traditional views.

Views simplify SPARQL, reduce repetition in code, and accelerate query performance - they are incredible useful.

## Example

With the rule below, we create a view of the profit from our best selling products - that is, the products that have sold over 1000 units.

Here is our product data:

```
:productA a :Product ;
    :unitsSold 5000 ;
    :hasCost 9 ;
    :hasSalePrice 10 .

:productB a :Product ;
    :unitsSold 2000 ;
    :hasCost 80 ;
    :hasSalePrice 100 .

:productC a :Product ;
    :unitsSold 10 ;
    :hasCost 30 ;
    :hasSalePrice 50 .
```
And the rule that will create our view:
```
[?product, a, :highPerformingProduct],
[?product, :hasTotalProfit, ?totalProfit] :-
    [?product, a, :Product],
    [?product, :unitsSold, ?units],
    [?product, :hasCost, ?cost],
    [?product, :hasSalePrice, ?price],
    BIND ( (?price - ?cost) * ?units AS ?totalProfit ),
    FILTER (?units > 1000).
```

## Run the script

Run `3_1-RulesAsViews/example/exScript.rdfox` to see the results of this rule.

### You should see...

=== High Performing Products ===
|?product|	?totalProfit|	?units|
|---|---|---|
|<https://rdfox.com/example#productB>|	40000|	2000|
|<https://rdfox.com/example#productA>|	5000|	5000|


### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3Fproduct%20%3FtotalProfit%20%3Funits%0AWHERE%20%7B%0A%20%20%20%20%3Fproduct%20a%20%3AhighPerformingProduct%20%3B%0A%20%20%20%20%20%20%20%20%3AunitsSold%20%3Funits%20%3B%0A%20%20%20%20%20%20%20%20%3AhasTotalProfit%20%3FtotalProfit%20.%0A%7D%20ORDER%20BY%20DESC%28%3FtotalProfit%29).

## Incremental Reasoning

Adding to this, reasoning updates incrementally meaning as new data comes in or as old is removed, the view is always kept consistent with changes to the data store in real-time. This is a very efficient process which considers the smallest necessary part of the graph to compute the change.

## Run the script

Ensuring that you have run the first script...

now run `3_1-RulesAsViews/example/exScript2.rdfox` to add this new product data and re-run the query.

```
:productD a :Product ;
    :unitsSold 10000 ;
    :hasCost 10 ;
    :hasSalePrice 110 .
```
=== High Performing Products ===
|?product|	?totalProfit|	?units|
|---|---|---|
|<https://rdfox.com/example#productD>|	1000000|	10000|
|<https://rdfox.com/example#productB>|	40000|	2000|
|<https://rdfox.com/example#productA>|	5000|	5000|

## Explainable results

RDFox keeps track of the inferences it makes, storing the chain of rules that support each facts existence.

This means all inferred results can be explained in proof trees of the facts and rules that lead to them.

### Exploring explanations

Head to the **Explain** tab of the RDFox console to start visualising an explanation, or right click on an edge in the **Explore** tab and selecting **Explain**.

[Here is a link](http://localhost:12110/console/datastores/explain?datastore=default&fact=%5B%3Chttps%3A%2F%2Frdfox.com%2Fexample%23productA%3E%2C%20%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23type%3E%2C%20%3Chttps%3A%2F%2Frdfox.com%2Fexample%23highPerformingProduct%3E%5D) to the explanation of `productA a :highPerformingProduct`.

## Exercise

Complete the rule `3_1-RulesAsViews/incompleteRules.dlog` so that the query below will return a list of product categories and the **average** units sold for all products in that category.

```
SELECT ?productCategory ?averageCategoryUnitsSold
WHERE {
    ?productCategory :hasAverageUnitsSold ?averageCategoryUnitsSold .
} ORDER BY DESC (?averageCategoryUnitsSold)
```

Here is a representative sample of the data in `3_1-RulesAsViews/exercise/data.ttl`.

```
:product0001 a :Product,
                :Chair ;
    :unitsSold 3000 ;
    :hasCost 40 ;
    :hasSalePrice 50 .

:product0002 a :Product,
                :Sofa ;
    :unitsSold 100 ;
    :hasCost 120 ;
    :hasSalePrice 300 .
```

### Hits & helpful resources

Averages are a form of aggregates.

### Check your work

Run the script below to verify the results.

`3_1-RulesAsViews/exercise/script.rdfox`

## You should see...

=== Product Category Average Profit ===
|?productCategory|?averageCategoryUnitsSold|
|-----------|-------------|
|<https://rdfox.com/example/Chair>|	9943.57894736842105|
|<https://rdfox.com/example/Product>|	4939.885|
|<https://rdfox.com/example/Sofa>|	4879.032|
|<https://rdfox.com/example/Stool>|	2531.395061728395062|
|<https://rdfox.com/example/Beanbag>|	2495.9|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FproductCategory%20%3FaverageCategoryUnitsSold%0AWHERE%20%7B%0A%20%20%20%20%3FproductCategory%20%3AhasAverageUnitsSold%20%3FaverageCategoryUnitsSold%20.%0A%7D%20ORDER%20BY%20DESC%20%28%3FaverageCategoryUnitsSold%29).