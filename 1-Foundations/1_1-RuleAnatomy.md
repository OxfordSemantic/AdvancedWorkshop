# 1.1 Anatomy of a Rule

**\[ THEN \] :- \[ IF \] .**

Datalog rules can be read as then-if statements, adding new data to the graph where a specified pattern is matched.

Rules are comprised of two main parts, the head and the body, separate by the inference symbol.

**\[ HEAD \] :- \[ BODY \] .**

Both the head and body can both be made up of several **atoms** - triple (or quad) patterns that the rule is looking to match.

These patterns are described by variables `?x` and data `:myData`.

```
[?node, a, :NewClass] ,
[?node, :newRelationship, :newNode] :-
    [?node, a, :Class] ,
    [?node, :hasProp, "property"] .
```

<br>
<br>

## ℹ️ &nbsp; Syntax requirements

1. Each atom must be separated by a comma.

2. Each entity within an atom, variable or data, must be separated by a comma.

3. The final head atom must be followed by the inference symbol `:-`.

4. The final body atom must be followed by a full stop.
