# 2.4 Debugging Rules with the Reasoning Profiler

<br>

## 🔥 &nbsp; Why is the Reasoning Profiler helpful?

Ever written code that didn't quite do what you expected? 

As far as rules are concerned, the Reasoning Profiler is the place to start!

<br>
<br>

## 📖 &nbsp; What is the Reasoning Profiler?

The primary tool in debugging rules and their performance is the Reasoning Profiler.

It prints information about the progress of the reasoning algorithms to the RDFox shell.

The Reasoning Profiler provides clues as to what is going on rather than providing precise solutions - the information it provides must be interpreted in the context of your rules.

<br>
<br>

## 🔬 &nbsp; Using the profiler

The following example turns on the Reasoning Profiler before importing a tiny data- and rules-set.

Here is the data:

```
:something a :Class .
```

And the rule:

```
[?x, :hasNewProp, "new property"] :-
    [?x, a, :Class] .
```

<br>
<br>

## 👀 &nbsp; Check the results

Run `2_5-ReasoningProfiler/exercise/script.rdfox` to see how RDFox computes this rule.

<br>

### You should see...

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             2|           1|            1|          (0) :hasNewProp[?x, "new property"] :- :Class[?x] .|

Plan Info
|  Sample Count  |  Iterator Open |   Iterator Advance  |  Plan Node  |                                  
|----------------|----------------|---------------------|-------------|
|   0            |    1           |             1       |             [?x, rdf:type, :Class]    {    -->    ?x }    TripleTableIterator|

There is a huge amount of detail with each part providing specific information however, in practice, we focus on just a few values - we'll break them down below.

<br>
<br>

## ℹ️ &nbsp; Syntax helper

This output can be difficult to read, so we recommend to copy it into a blank `.dlog` file - while this is of course not valid Datalog, it allows us to make use of the syntax highlighting provided by the RDFox VS Code Extension.

Here, we have done this for you in `2_5-ReasoningProfiler/exercise/reasoningProfilerOutput.dlog`.

## 🔍 &nbsp; Reading this table in practice

Detailed explanations of all of these columns are given at the end of the chapter but in practice, the relative size of a few key figures is often all we need.

<br>

### Finding slow rule atoms with Iterator Operations

Iterator Operations tells you how many steps have been performed to compute the inferences of the rule.

In other words, this is how much work RDFox has done for this rule - the higher the number, the more taxing a rule or rule atom.

If this value is unexpectedly high the rule needs examining carefully.

In this case the rule took 2 operations, one to match the only available fact, and one to confirm there were no more.

<br>

### Finding the problem with Rule Body Match Attempts and Fresh Facts Produced

Rule Body Match Attempts Fresh Facts Produced show you how many facts have been created for the work that has been done.

**Rule Body Match Attempts** shows the number of times the rule's body shape appears in the data.

**Fresh Facts Produced** shows the number of facts that were actually produced from these matches.

If these values are greatly misaligned, it may means one following:

1. the rule is considering too broad a pattern
2. the rule is creating redundant new facts
3. the rule is attempting to infer repeated facts

In this case both had a value of 1 as the rule matched 1 pattern (our only fact), and added only one new fact.

<br>
<br>

## 🔍 &nbsp; How to use the Profiler

The following RDFox commands were executed by the `2_5-ReasoningProfiler/exercise/script.rdfox`, but in normal operation you will need to enter these yourself.

### Starting the Profiler

To turn on the profiler, you must run `set reason.monitor profile` in the RDFox shell.

### Greater detail

Further details about the rule plans can be printed with `set reason.profiler.log-plans true`.

### Changing the log-frequency

By default, the information is printed after the reasoning has completed however, it can be more helpful to print out the progress at set intervals.

The length of the intervals can be changed with `set log-frequency 10` - setting the print frequency to every 10 seconds.

Learn more about the [Reasoning Profiler settings here](https://docs.oxfordsemantic.tech/reasoning.html#the-reasoning-profiler).

<br>
<br>

## 🚀 &nbsp; Exercise

The following rule and dataset aim to highlight Alice's social success.

Here is our entire dataset:

```
:Alice :knows :Bob ,
    :Carlos ,
    :Danielle ,
    :Erin ,
    :Frank .
```

And here are the rules that we'll use. Using the profiler, try to identify the problematic rule.

```
[?alice, :hasFriend, ?person], 
[?person, :hasFriend, ?alice] :-
    [?alice, :knows, ?person] .

[?alice, :has, :loadsOfFriends]:-
    [?alice, :knows, ?person1] ,
    [?alice, :knows, ?person2] ,
    [?alice, :knows, ?person3] ,
    [?alice, :knows, ?person4] .
```

<br>
<br>

## 👀 &nbsp; Check the results

Run `2_5-ReasoningProfiler/exercise2/script.rdfox` to see how RDFox computes these rules.

Copy the output into `2_5-ReasoningProfiler/exercise2/reasoningProfilerOutput.dlog` for a cleaner view.

<br>

### Reasoning Profile breakdown

One of our rules has a relative number of Iterator Options and similar number (2x) of Rule Body Matches and Fresh Facts Produced (all of which we would expect from this kind of rule) - nothing wrong here.

Our other rule has 1.5k Iterator Options despite our data size of 5 facts, and 600 time more Rule Body Matches than Fresh Facts Produced - this is where our issue is.

When we look at our problematic rule, this becomes clear.

```
[?alice, :has, :loadsOfFriends]:-
    [?alice, :knows, ?person1] ,
    [?alice, :knows, ?person2] ,
    [?alice, :knows, ?person3] ,
    [?alice, :knows, ?person4] .
```

As written, this rule considers every possible combination of the people Alice knows in order to find every single permutation of 4 (not even distinct) people, and with every match, repeatedly infers the same fact: `:Alice  :has :loadsOfFriends`.

We'll see several other, more realistic examples throughout the following chapters.

<br>
<br>

## 📌 &nbsp; Hints & helpful resources

[More about RDFox's Reasoning Profiler](https://docs.oxfordsemantic.tech/reasoning.html#using-reasoning-monitors-in-rdfox)

<br>
<br>

## 👏 &nbsp; BONUS: Rule Info in detail

These bonus details explore the first rule in this exercise that can be imported with `2_5-ReasoningProfiler/exercise/script.rdfox`.

Having imported our data and rule, we will see the **Rule Info** table below printed in the shell, just above the **Plan Info** (we'll come to that in a moment).

Rule Info
|   #  |    Reasoning Phase   |   Sample Count  |  Rule Body Match Attempts |   Iterator Operations  |  Rule Body Matches  |    Fresh Facts Produced    |Rule of Head Atom|                               
|------|----------------------|-----------------|---------------------------|------------------------|---------------------|-------------------|------------------------------|
|   1|    RuleAdd|      0|             1|             2|           1|            1|          (0) :hasNewProp[?x, "new property"] :- :Class[?x] .|

<br>

### Rule Info table columns explained

<br>

**#**

There are often many rules, so the number represents (#) it's position in a list of approximately most to least costly at the time of the report.

<br>

**Reasoning Phase** 

Reasoning Phase denotes the algorithm being used for this part of the reasoning computation (see below).

- Here, RDFox used the **Rule Add** algorithm, as there was data already in the store and we only imported a simple rule.

<br>

**Sample Count**

Sample Count gives the number of times, when checked at regular intervals, compute was working on this part of the rule - providing an estimate of the compute cost. There will often be several sample counts for different parts of the rule, the proportion of the total count observed at each part is as important as the magnitude of the number itself as this tells us exactly which atom(s) is problematic.

- Here, the rule was so fast that RDFox didn't have time to sample it before it completed. 

<br>

**Rule Body Match Attempts**

Rule Body Match Attempts shows the number of unique times RDFox attempted to match the body atoms with data.

- Here there was 1 match attempt as we only have one fact to match - `:something a :Class .` in the data we imported.

<br>

**Iterator Operations**

Iterator Operations is the number of individual steps the reasoning algorithm when through when importing the rules and adding the inferences. This is a very important stat as it is a key indicator of the efficiency of the rule. It serves a similar purpose to **Sample Count** but provides even greater detail.

- Here, the Iterator progressed 2 times during this completion, once to find a fact that matches the body atom and once look for a next fact that matches (in this case there is no next). 2 iterations from a total of 2 were spent on this body atom, meaning 100% of the compute for importing this rule was spent on the atom shown - no surprise as the rule only has 1 atom.

<br>

**Rule Body Matches**

Rule Body Matches is the actual number of times a body atoms matched, leading to the addition of a fact, throughout the operations.

- Here, of the match attempts, 1 was successful because the single fact we have does match our body atom.

- When compared to the **Iterator Operations**, if they are vastly different (orders of magnitude) it can be a sign that something is wrong as lots of data is being considered for relatively few complete patterns - although not necessarily, it is for you to determine. It may indicate you need to be more specific with your body atoms.

<br>

**Fresh Facts Produced** 

Fresh Facts Produced is the total number of new facts produced.

- Here, the only head atom produced 1 fact as there was only 1 body match.

- If this is vastly smaller than the **Rule Body Matches** it may indicate that the rule is inefficient as it is checking lots of facts to produce very few, or it may be producing lots of repeated facts.

<br>

**Rule of Head Atom**

Rule of Head Atom is simply the relevant rule head atom that is being for this step in the reasoning process.

- Here, our rule only has one head atom so we see just that.

<br>

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

<br>
<br>

## 👏 &nbsp; BONUS: Plan Info in detail

These bonus details explore the first rule in this exercise that can be imported with `2_5-ReasoningProfiler/exercise/script.rdfox`.

As we `set reason.profiler.log-plans true`, the rule plans are printed alongside the Rule Info.

This gives us even greater detail about how the rule was executed.

Below the **Rule Info** table, you should see the **Plan Info** table.

|  Sample Count  |  Iterator Open |   Iterator Advance  |  Plan Node  |                                  
|----------------|----------------|---------------------|-------------|
|   0            |    1           |             1       |             [?x, rdf:type, :Class]    {    -->    ?x }    TripleTableIterator|

<br>

### Table columns explained

<br>

**Sample Count**

As in the **Rule Info** table, sample count gives an estimate for the computation cost - this time breaking down the rule into even greater detail. It is the number of times, when checked at regular intervals, compute was working on this part of the rule.

- Here, the rule was so fast that RDFox didn't have time to sample it before it completed.

<br>

**Iterator Open**

This is the number of times the specific iterator attempted to find anything that matched the atom.

- Here, it was just opened 1 time as there is at least 1 fact in the store that matches the atom.

<br>

**Iterator Advance**

This is the number of times the specific iterator tried to find another triple, having found at least the first, that matched the atom, successfully or not.

- Here, it was just advanced successfully 1 time as the there is only 1 fact in the store, so there was nothing next.

<br>

**Plan Node**

Plan node shows specifically which atom was considered during this step, and the bindings that were affected. The braces **{ --> }** show which variables were bound as the iterator was opened, and which were bound when it advanced. The type of iterator is also shown but this is not helpful in practice most of the time.

- Here, no variables were bound as the iterator was opened because this was the first step in the reasoning process and nothing had happened yet. Once the iterator had the fact, the variable `?x` was bound to a value.
