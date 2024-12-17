# 2.5 Debugging Rules with the Reasoning Profiler

The primary tool in debugging slow rules is the Reasoning Profiler.

It prints information about the progress of the reasoning algorithms to the RDFox shell.

The Reasoning Profiler provides clues as to what is going on - the information it provides must be interpreted in the context of your rules.

It  does not provide one-size-fits-all solutions, instead providing an excellent place to start when debugging.

## Starting the Profiler

To use it, you must first to enable it by running `set reason.monitor profile` in the RDFox shell.

Once the reasoning is complete, either from importing rules or data, a table detailing the computation involved will be shown.

### Printing more details

The rule plans can be printed alongside the profiler information with `set reason.profiler.log-plans true`, providing a greater level of detail for when it's needed.

### Changing the log-frequency

By default, the information is printed after the reasoning has completed however, this can be unhelpful as it is possible to write rules that are very long running or even infinite, in which case it is much more helpful to know the progress of the reasoner.

A log of the profiles can be printed at set intervals with `set log-frequency 10` - setting the print frequency to every 10 seconds.

This is helpful in the majority of cases.

Learn more about the [Reasoning Profiler settings here](https://docs.oxfordsemantic.tech/reasoning.html#the-reasoning-profiler).

## Exercise

Below are a series of exercises that show how to use the profiler and telltale signs that can indicate certain issues.

You must run these commands in the RDFox shell to see the output of the profiler.

### Reading the output

The info that the reasoning profiler outputs can be confusing, so it's important to cover it step by step.

Running the script `2_5-ReasoningProfiler/exercise/script.rdfox` to start the profiler and see the results of importing a rule.

It first adds this simple piece of data:

```
:thing a :Class .
```
Then it starts the profiler and imports this simple rule.

```
[?thing, :hasProp, "Property"] :-
    [?thing, a, :Class] .
```

The Reasoning Profiler will print a log of what happens. 

The output can be very difficult to read, so it is recommended to copy and paste it into a blank `.dlog` file in order to get the syntax highlighting.

## Using Rule Info and Plan Info in practice

Having imported our data and rule, we will see the following tables printed in the shell.

Together, **Rule Info** and **Plan Info** provide information about the work RDFox is doing when performing the inference, broken down by **each rule atom**.

There is a huge amount of detail with each part providing specific information however, in practice, we focus on just a few values.

For a detailed explanation and the number shown explained with context, see the bonus sections below.

### The Profiler printed in the shell

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             2|           1|            1|          (0) :hasProp[?thing, "Property"] :- :Class[?thing] .|

Plan Info
|  Sample Count  |  Iterator Open |   Iterator Advance  |  Plan Node  |                                  
|----------------|----------------|---------------------|-------------|
|   0            |    1           |             1       |             [?thing, rdf:type, :Class]    {    -->    ?thing }    TripleTableIterator|

### Resource intensive rules

The first thing to look at is the **Iterator Operations** (or **sample count** - a legacy metric use as a proxy for Iterator Operations).

This tells you how many steps have been performed to compute the inferences of the rule.

In other words, this is how much work RDFox has done for this rule - the higher the number, the more taxing a rule, or part of a rule, is.

When this value is very high, or higher than you might expect, the rule is worth digging into.

### Finding the problem

The next thing to look at is the **Rule Body Match Attempts** and **Fresh Facts Produced** - together they show you how many facts have been created for the work that has been done.

**Rule Body Match Attempts** shows the number of times the rule's body shape appears in the data and **Fresh Facts Produced** shows the number of facts that were actually produced from these matches.

Either or both of these values can be smaller or larger than expected, indicating your rule is doing something you didn't intent - often either considering too much data or adding too many new facts, in which case you should adjust your rule accordingly.

If **Fresh Facts Produced** is significantly smaller than **Rule Body Match Attempts**, your rule is attempting to infer many repeated facts, causing RDFox to perform unnecessary work - a more restrictive body is often the solution.

We will see this in action in later exercises but you're encouraged check the profiler output as we go.

## BONUS: Rule Info in detail

Having imported our data and rule, we will see the **Rule Info** table below printed in the shell, just above the **Plan Info** (we'll come to that in a moment).

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             2|           1|            1|          (0) :hasProp[?thing, "Property"] :- :Class[?thing] .|

### Rule Info table columns explained

**#**

There are often many rules, so the number represents (#) it's position in a list of approximately most to least costly at the time of the report.

**Reasoning Phase** 

Reasoning Phase denotes the algorithm being used for this part of the reasoning computation (see below).

- Here, RDFox used the **Rule Add** algorithm, as there was data already in the store and we only imported a simple rule.

**Sample Count**

Sample Count gives the number of times, when checked at regular intervals, compute was working on this part of the rule - providing an estimate of the compute cost. There will often be several sample counts for different parts of the rule, the proportion of the total count observed at each part is as important as the magnitude of the number itself as this tells us exactly which atom(s) is problematic.

- Here, the rule was so fast that RDFox didn't have time to sample it before it completed. 

**Rule Body Match Attempts**

Rule Body Match Attempts shows the number of unique times RDFox attempted to match the body atoms with data.

- Here there was 1 match attempt as we only have one fact to match - `:thing a :Class .` in the data we imported.

**Iterator Operations**

Iterator Operations is the number of individual steps the reasoning algorithm when through when importing the rules and adding the inferences. This is a very important stat as it is a key indicator of the efficiency of the rule. It serves a similar purpose to **Sample Count** but provides even greater detail.

- Here, the Iterator progressed 2 times during this completion, once to find a fact that matches the body atom and once look for a next fact that matches (in this case there is no next). 2 iterations from a total of 2 were spent on this body atom, meaning 100% of the compute for importing this rule was spent on the atom shown - no surprise as the rule only has 1 atom.

**Rule Body Matches**

Rule Body Matches is the actual number of times a body atoms matched, leading to the addition of a fact, throughout the operations.

- Here, of the match attempts, 1 was successful because the single fact we have does match our body atom.

- When compared to the **Iterator Operations**, if they are vastly different (orders of magnitude) it can be a sign that something is wrong as lots of data is being considered for relatively few complete patterns - although not necessarily, it is for you to determine. It may indicate you need to be more specific with your body atoms.

**Fresh Facts Produced** 

Fresh Facts Produced is the total number of new facts produced.

- Here, the only head atom produced 1 fact as there was only 1 body match.

- If this is vastly smaller than the **Rule Body Matches** it may indicate that the rule is inefficient as it is checking lots of facts to produce very few, or it may be producing lots of repeated facts.

**Rule of Head Atom**

Rule of Head Atom is simply the relevant rule head atom that is being for this step in the reasoning process.

- Here, our rule only has one head atom so we see just that.

### RDFox Algorithms

RDFox uses different algorithms depending on whether the datastore has any data in it, and whether data or rules are being added or deleted.

In most cases, only one option is compatible so RDFox makes this choice for you. For example, the **Add Rules** algorithm cannot be used to add data.

However, there is one important exception.

Data can be added either by Materialisation (**Mat**) or via Incremental Addition (**Addition**).

RDFox used **Mat** when adding data into a store that doesn't yet contain data (rules are allowed). This occurs when importing data for the first time or it can be forced by using the shell command **remat** to re-materialise the entire datastore.

On the other hand, **Incremental Addition** is used when data already exists in the store and more is added.
 
**Incremental Addition** strictly considers only the change, which is optimal for small changes but may have to retract some inferred facts because of the change which is very computationally expensive.

**Materialisation** on the other hand will never have to compute retraction as it always starts from a clean sheet, so can be much more efficient for large data changes where many previously inferred facts no longer hold.

Depending on the contents of the datastore, one can be vastly more efficient than the other and can make a huge difference to your load times.

## BONUS: Plan Info in detail

As we `set reason.profiler.log-plans true`, the rule plans are printed alongside the Rule Info.

This gives us even greater detail about how the rule was executed.

Below the **Rule Info** table, you should see the **Plan Info** table.

|  Sample Count  |  Iterator Open |   Iterator Advance  |  Plan Node  |                                  
|----------------|----------------|---------------------|-------------|
|   0            |    1           |             1       |             [?thing, rdf:type, :Class]    {    -->    ?thing }    TripleTableIterator|

### Table columns explained

**Sample Count**

As in the **Rule Info** table, sample count gives an estimate for the computation cost - this time breaking down the rule into even greater detail. It is the number of times, when checked at regular intervals, compute was working on this part of the rule.

- Here, the rule was so fast that RDFox didn't have time to sample it before it completed.

**Iterator Open**

This is the number of times the specific iterator attempted to find anything that matched the atom.

- Here, it was just opened 1 time as there is at least 1 fact in the store that matches the atom.

**Iterator Advance**

This is the number of times the specific iterator tried to find another triple, having found at least the first, that matched the atom, successfully or not.

- Here, it was just advanced successfully 1 time as the there is only 1 fact in the store, so there was nothing next.

**Plan Node**

Plan node shows specifically which atom was considered during this step, and the bindings that were affected. The braces **{ --> }** show which variables were bound as the iterator was opened, and which were bound when it advanced. The type of iterator is also shown but this is not helpful in practice most of the time.

- Here, no variables were bound as the iterator was opened because this was the first step in the reasoning process and nothing had happened yet. Once the iterator had the fact, the variable `?thing` was bound to a value.
