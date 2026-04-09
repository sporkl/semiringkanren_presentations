Relational/logic programming languages usually choose one of two approaches: top-down search (Prolog, miniKanren, etc.), or bottom-up fact collection (Datalog, SQL, etc.).
Polymorphism in programming languages roughly refers to single functions/relations which can operate on unknown/variable types of data.
While polymorphism usually comes for "free" in top-down languages, it is at odds with the "fact collection" approach of bottom-up languages.
How can we (finitely) state facts like "x is related to y" when we don't know what x and y are?

This talk introduces semiringKanren, a bottom-up variant of miniKanren with weights, algebraic data types, and polymorphic relations.
semiringKanren denotes weighted relations as finite tensors/n-dimensional arrays.
In particular, semiringKanren can (mostly) denote a polymorphic relation as a single tensor/ndarray, without needing to generate a separate version for it for each type.
