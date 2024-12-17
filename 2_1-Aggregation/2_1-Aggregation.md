# 2.1 Aggregation

Aggregate functions are used to calculate expressions over a set of values.

RDFox offers several, including common functions like SUM(V) and COUNT(V), and more specialized functions like COUNT_MAX(V) and MAX_ARGMIN(A,V).

See the [full list shown here](https://docs.oxfordsemantic.tech/querying.html#aggregate-functions) (excluding non-deterministic functions).

## Using aggregates

To use aggregation in a rule, you must declare an aggregate atom in the rule body with `AGGREGATE()`.

The parenthesis `()` of the aggregate atom must contain all the relevant triple atoms.

You must also indicate which variable(s) you're aggregating on with `ON` (known as group variables), and bind the result to a new variable with `BIND`.

## Example

Below is an example of COUNT, used to find the number of 5-star reviews each product has.

Here is the data we'll be using to show this:

```
:productA a :Product ;
    :hasReview :reviewA1 ;
    :hasReview :reviewA2 .

:productB a :Product ;
    :hasReview :reviewB1 ;
    :hasReview :reviewB2 .

:reviewA1 a :Review ;
    :hasStars 5 .

:reviewA2 a :Review ;
    :hasStars 5 .

:reviewB1 a :Review ;
    :hasStars 5 .

:reviewB2 a :Review ;
    :hasStars 4 .
    
```
And here is the rule that performs the count of 5 star reviews for each product:

```
[?product, :hasNumberOfFiveStars, ?countOfFiveStarRatings] :-
    AGGREGATE (
        [?product, :hasReview, ?review],
        [?review, :hasStars, 5]
        ON ?product
        BIND COUNT(?review) AS ?countOfFiveStarRatings
    ).
```

## Run the script

Run `2_1-Aggregation/example/exScript.rdfox` to see the results of this rule.

### You should see...

|=== Number of 5 star ratings per product ===||
|-----------|-------------|
|:productA| 2| 
|:productB| 1| 


## Variable scope

Variables in aggregate atoms are local to the atom unless they unless they are group variables (in the `ON` statement).

Therefore, two aggregates can use a variable with the same name so long as it is not a mentioned group variable.


## Where is aggregation relevant?

An incredibly powerful tool, there are unlimited uses of aggregation in production.

COUNT and SUM are so versatile that they can be used in almost any application, with other functions supporting increasingly targeted and powerful niches.

### Retail

To quantify and analyze user behavior, quantify and rank product sentiment, etc.

### Finance

To sum transactional amounts, isolate extreme values and outliers, validate complex regulation compliance, etc.

### Construction & Manufacturing

To surface and compare high-level compound properties, create a bill of materials, etc.

The list goes on...

## Exercise

Complete the rule `2_1-Aggregation/incompleteRules.dlog` so that the query below can be used to directly find 'Star Products' - that is, products with a **higher than 4 star average** rating AND **more than 5 total reviews**.

```
SELECT ?starProduct ?averageStars ?reviewCount
WHERE {

    ?starProduct a :StarProduct ;
        :hasAverageStars ?averageStars ;
        :hasReviewCount ?reviewCount .

} ORDER BY DESC(?averageStars)

```

Here is a representative sample of the data in `2_1-Aggregation/exercise/data.ttl`.

```
:product0001 a :Sofa ;
    :hasReview :review11.

:review11 :hasStars 5 .
```

### Hits & helpful resources

[Aggregate functions in RDFox](https://docs.oxfordsemantic.tech/querying.html#aggregate-functions)

## Check your work

Run the script below to verify the results.

`2_1-Aggregation/exercise/script.rdfox`

### You should see...

**========================= Our Star Products =========================**
|?starProduct|?averageStars|	?reviewCount|
|-----------|-------------|-------------|
|:product0579|	4.5|	8|
|:product0934|	4.5|	6|
|:product0395|	4.428571428571428571|	7|
|:product0209|	4.375|	8|
|:product0317|	4.285714285714285714|	7|
|:product0222|	4.166666666666666667|	6|
|:product0931|	4.142857142857142857|	7|
|:product0492|	4.142857142857142857|	7|
|:product0137|	4.142857142857142857|	7|
|:product0326|	4.125|	8|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FstarProduct%20%3FaverageStars%20%3FreviewCount%0AWHERE%20%7B%0A%20%20%20%20%3FstarProduct%20a%20%3AStarProduct%20%3B%0A%20%20%20%20%3AhasAverageStars%20%3FaverageStars%20%3B%0A%20%20%20%20%3AhasReviewCount%20%3FreviewCount%20.%0A%7D%20ORDER%20BY%20DESC%28%3FaverageStars%29).
