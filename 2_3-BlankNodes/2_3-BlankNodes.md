# 2.3 Blank Nodes

Blank nodes enable us to represent entities that themselves have properties or relate to other nodes, without giving the node itself an IRI.

This is particularly relevant if the node itself is uninteresting or unknown but its relationships to the rest of the graph are, in which case a composite object that provides several details is more helpful.

Blank nodes are created without specifying an IRI - RDFox will generate a random identifier for this node in order to keep track of it. This can present challenges as the identifier is not generated deterministically and will be different every time the data is imported.

While blank nodes can be incredible useful, they can be difficult to work with due to this inconsistent anonymized identity which is why, in practice, we highly encourage using them with SKOLEM.

## SKOLEM

SKOLEM generates a blank node with a deterministic label from a series of input parameters of both variables and literals.

This means the new node retains a referenceable, reversible, meaningful IRI that can be based solely on selected properties.

### SKOLEM syntax

SKOLEM("literalOr", ?variable, ..., ?result)

The last argument given to SKOLEM is the variable that will be bound to the generated value. All preceding arguments contribute to its generation.

Arguments can be either a variable or string.

## Example

This example examines sensor readings of a liquid bath that is being monitored by two sensors in a manufacturing process.

We'll create a blank node with SKOLEM that represents a summary of the bath's temperature.

Here are the two sensors and their readings:

```
:readingA a :SensorReading ;
    :fromSensor :sensorA ;
    :hasTemperature 19 .

:readingB a :SensorReading ;
    :fromSensor :sensorB ;
    :hasTemperature 21 .
```

And the rule that will create a SKOLEM node to represent them:

```
[?bathReading, a, :BathSummary],
[?bathReading, :hasTemperatureEstimate, ?tempEstimate] :-
    [?reading1, :fromSensor, :sensorA],
    [?reading2, :fromSensor, :sensorB],
    [?reading1, :hasTemperature, ?temp1],
    [?reading2, :hasTemperature, ?temp2],
    BIND((?temp1 + ?temp2)/2 AS ?tempEstimate),
    SKOLEM("Bath Summary", ?tempEstimate, ?bathReading) .
```

## Run the script

Run `2_3-BlankNodes/example/exScript.rdfox` to see the results of this rule.

### You should see...

================= Average Bath Temperature Estimate Over Time =================
|?bathReadingEncoded|	?temperatureEstimate|
|----------------------------------------------|----|
|_:_.05QmF0aCBTdW1tYXJ5AA.16FAAAAAAAAAAAAAAAAAAAAA|	20.0|

### Visualise the results

Open this query in the [RDFox Explorer](http://localhost:12110/console/datastores/explore?datastore=default&query=SELECT%20%3FbathReadingEncoded%20%3Ftime%20%3FtemperatureEstimate%20%0AWHERE%20%7B%0A%20%20%20%20%3FbathReadingEncoded%20a%20%3ABathSummary%20%3B%0A%20%20%20%20%20%20%20%20%3AhasTimeStamp%20%3Ftime%20%3B%0A%20%20%20%20%20%20%20%20%3AhasTemperatureEstimate%20%3FtemperatureEstimate%20.%0A%7D%20ORDER%20BY%20ASC%28%3Ftime%29).

## Reversing SKOLEM

SKOLEM is a reversible process. Given a SKOLEMized node IRI, we can determine the resources that were used to create it.

Here is a rule that unpicks the SKOLEM node we just created:

```
[?sting, :derivedFrom, ?bathReading],
[?tempEstimate, :derivedFrom, ?bathReading] :-
    [?bathReading, a, :BathSummary],
    SKOLEM(?sting, ?tempEstimate, ?bathReading) .
```

And a query that returns the derived components:
```
SELECT ?SKOLEMNode ?originalComponent
WHERE {
    ?originalComponent :derivedFrom ?SKOLEMNode .
}
```

## Run the script

Ensuring you have already run the first script...

now run `2_3-BlankNodes/example/exScript2.rdfox` to see the results of this rule.

### You should see...

================= Average Bath Temperature Estimate Over Time =================
|?SKOLEMNode|	?originalComponent|
|----------------------------------------------|----|
|_:_.05QmF0aCBTdW1tYXJ5AA.16FAAAAAAAAAAAAAAAAAAAAA|	20.0|
|_:_.05QmF0aCBTdW1tYXJ5AA.16FAAAAAAAAAAAAAAAAAAAAA|	"Bath Summary"|

### MD5 Encoding

As a practical alternative to SKOLEM, the MD5 hash generator function sometimes is instead to create an ID for a node.

MD5 creates IDs more efficiently than SKOLEM but is non-reversible, meaning retraction of facts is vastly less efficient.

Additionally, it is not strictly 1-1, so it is possible, albeit unlikely, to have two different inputs create the same output, 

MD5 is a function not a tuple table, so the output must be bound to a new variable, and it's input must be strings.

`BIND ( MD5("stringIn") AS ?stringOut )`

## Where are blank nodes relevant?

Blank nodes are incredibly versatile, allowing abstract representations of data to be added directly to the graph.

As such, they can be used in almost any application, including:

### Finance

To generate objects that capture sprawling details such as in fraud chains, suspicious activity, or regulation breaches etc.

### Iot & On-device

To structure event data, surface high level system properties, catalog time-series events, etc.

### Data Management

To create useful structures from complex data, generate abstractions, etc.

## Exercise

Complete the rule `2_3-BlankNodes/incompleteRules.dlog` to monitor the average temperature of several liquid baths as they change over time.

Both baths contain exactly two sensors

```
SELECT ?bath ?time ?temperatureEstimate
WHERE {
    ?bathReading a :BathSummary ;
        :locatedIn ?bath ;
        :hasTimeStamp ?time ;
        :hasTemperatureEstimate ?temperatureEstimate .
} ORDER BY ASC(?bath) ASC(?time)
```
Here is a representative sample of the data in `2_3-BlankNodes/exercise/data.ttl`.
```
:readingA001 a :SensorReading ;
    :fromSensor :sensorA ;
    :hasTemperature 18 ;
    :hasTimeStamp "2024-12-25T12:00:00"^^xsd:dateTime .

:readingB001 a :SensorReading ;
    :fromSensor :sensorB ;
    :hasTemperature 20 ;
    :hasTimeStamp "2024-12-25T12:00:00"^^xsd:dateTime .
```

### Hits & helpful resources

[SKOLEM in RDFox](https://docs.oxfordsemantic.tech/tuple-tables.html#skolem)

### Check your work

Run the script below to verify the results.

`2_3-BlankNodes/exercise/script.rdfox`

### You should see...

=== Bath temperature over time ===
|?bath|?time|?temperatureEstimate|
|-----------|-------------|-------------|
|:bathAlpha|	"2025-12-25T00:00:00"^^xsd:dateTime|	16.0|
|:bathAlpha|	"2025-12-25T02:00:00"^^xsd:dateTime|	16.5|
|:bathAlpha|	"2025-12-25T04:00:00"^^xsd:dateTime|	17.5|
|:bathAlpha|	"2025-12-25T06:00:00"^^xsd:dateTime|	19.0|
|:bathAlpha|	"2025-12-25T08:00:00"^^xsd:dateTime|	20.0|
|:bathAlpha|	"2025-12-25T10:00:00"^^xsd:dateTime|	22.0|
|:bathAlpha|	"2025-12-25T12:00:00"^^xsd:dateTime|	23.5|
|:bathAlpha|	"2025-12-25T14:00:00"^^xsd:dateTime|	23.5|
|:bathAlpha|	"2025-12-25T16:00:00"^^xsd:dateTime|	24.0|
|:bathAlpha|	"2025-12-25T18:00:00"^^xsd:dateTime|	23.5|
|:bathAlpha|	"2025-12-25T20:00:00"^^xsd:dateTime|	24.5|
|:bathAlpha|	"2025-12-25T22:00:00"^^xsd:dateTime|	24.0|
|:bathBeta|	"2025-12-25T00:00:00"^^xsd:dateTime|	16.0|
|:bathBeta|	"2025-12-25T02:00:00"^^xsd:dateTime|	16.0|
|:bathBeta|	"2025-12-25T04:00:00"^^xsd:dateTime|	16.5|
|:bathBeta|	"2025-12-25T06:00:00"^^xsd:dateTime|	16.5|
|:bathBeta|	"2025-12-25T08:00:00"^^xsd:dateTime|	17.5|
|:bathBeta|	"2025-12-25T10:00:00"^^xsd:dateTime|	19.5|
|:bathBeta|	"2025-12-25T12:00:00"^^xsd:dateTime|	20.5|
|:bathBeta|	"2025-12-25T14:00:00"^^xsd:dateTime|	21.5|
|:bathBeta|	"2025-12-25T16:00:00"^^xsd:dateTime|	22.5|
|:bathBeta|	"2025-12-25T18:00:00"^^xsd:dateTime|	23.5|
|:bathBeta| "2025-12-25T20:00:00"^^xsd:dateTime |24.5 .|
|:bathBeta |"2025-12-25T22:00:00"^^xsd:dateTime |25.5 .|
|:bathGamma |"2025-12-25T00:00:00"^^xsd:dateTime |15.5 .|
|:bathGamma |"2025-12-25T02:00:00"^^xsd:dateTime |16.5 .|
|:bathGamma |"2025-12-25T04:00:00"^^xsd:dateTime |17.5 .|
|:bathGamma |"2025-12-25T06:00:00"^^xsd:dateTime |19.5 .|
|:bathGamma |"2025-12-25T08:00:00"^^xsd:dateTime |19.5 .|
|:bathGamma |"2025-12-25T10:00:00"^^xsd:dateTime |20.0 .|
|:bathGamma |"2025-12-25T12:00:00"^^xsd:dateTime |21.0 .|
|:bathGamma |"2025-12-25T14:00:00"^^xsd:dateTime |23.0 .|
|:bathGamma |"2025-12-25T16:00:00"^^xsd:dateTime |23.5 .|
|:bathGamma|	"2025-12-25T18:00:00"^^xsd:dateTime|	24.5|
|:bathGamma|	"2025-12-25T20:00:00"^^xsd:dateTime|	26.5|
|:bathGamma|	"2025-12-25T22:00:00"^^xsd:dateTime|	26.0|
