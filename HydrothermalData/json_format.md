# Storage and Retrieval


The proposed file format is a valid JSON (JavaScript Object Notation) document. This enables files to be human and machine readable. JSON objects are key-value mappings enclosed by curly braces ({ ... }). JSON arrays are comma-separated lists of values enclosed by square braces ([...]).


## Problem Formulation

Compared to linear programs, multistage stochastic programs problems have the unique feature that every model paramater can be random so that variables and constraints are path-dependent. To account for this feature, the proposed format allows all model parameters to be expressed as random variables.

We define a model parameters by a JSON array of numeric values and text strings that are enclosed in square braces:

- **Deterministic:** If a model parameter is non-random or zero, we use a single float value, e.g., [1.0], [0.0], [8.1], etc.
- **Random:** If a model parameter is random, we use a text string, e.g., ["demand"], ["price1"], ["price2"], etc. ("inf" and "-inf" are literals and will not be parsed as random parameters)
- **Both (version 1.0):** If a model parameter is the product of random variables and numeric values, we use list of values and/or strings, e.g., [0.9,"price"], ["demand","price"], [1.1,"demand","price"], etc.
- **Both (version 1.1):** The format has been revised as of version 1.1 to account for non-linear random variables. Every random variable is now represented by an array of key-value pairs with the key being the function name and the value being a string or a real-value (as before), e.g. [{"ADD":"demand"},{"MUL":"price"},{"MUL":1.1}]. The first key is always "ADD" by default. Allowed functions are "ADD", "MUL", "POW", "EXP", "MAX", "MIN", "IND".

The complete model can then be defined like a conventional linear program, with the exception that every decision variable has a dedicated stage identifier and that model parameters are expressed as JSON arrays of string and/or values. At the top level of the JSON document, there are five keys:

- `"version"` The version of the file format, e.g., "MSMIP 1.0" for multistage stochastic mixed-integer program version 1.0.
- `"name"` The model name, e.g. "Newsvendor Model".
- `"maximize"` Whether it is a maximization problem or not, i.e., "true" or "false".
- `"variables"` A JSON array of decision variables expressed as JSON objects with six keys:
    - `"name"` The name of the variable that must be unique at the given stage.
    - `"stage"` An integer indicating the stage at which the decision is made.
    - `"obj"` The objective coefficient of this variable as JSON array, e.g, [1.0], [0.0], ["price"], [0.9,"price"], [{"ADD":0.9},{"MUL":"price"}], [{"EXP","log-price"}]
    - `"lb"` The lower bound of this variable as JSON array. The keyword "-inf" can be used, e.g., ["-inf"].
    - `"ub"` The upper bound of this variable as JSON array. The keyword "inf" can be used, e.g., ["inf"].
    - `"type"` A string indicating the type of variable: "CONTINUOUS", "INTEGER", "BINARY". 

- `"constraints"`: A JSON array of constraints expressed as JSON objects with four keys:
    - `"name"` The unique name of the constraint or "" if none.
    - `"type"` A string indicating the type of constraint: "LEQ"=less-or-equal, "EQ"=equality, "GEQ"=greater-or-equal, "NEQ"=not equal.
    - `"lhs"` The left-hand side as a JSON array of the product of variables and coefficients expressed as JSON objects with three keys (repeated variables ought to be collected in one term):
        - `"name"` The name of the decision variable.
        - `"stage"` An integer indicating the stage at which the decision is made.
        - `"coefficient"` The left-hand-side coefficient as JSON array, e.g, [1.0], [0.0], ["yield"], [0.9,"yield"]  
    - `"rhs"` The right-hand-side coefficient as JSON array, e.g, [1.0], [0.0], ["demand"], [0.8,"demand"],[{"ADD","demand"},{"MAX",0}]

# Scenario Lattice

A lattice is a graph that is organized in a finite number of layers. Each layer is associated with a discrete point in time and contains a finite number of nodes. Successive layers are connected by arcs. A node represents a possible state of a stochastic process, and an arc indicates the possibility of a state transition between the two connected nodes. Each arc is associated with a probability weight, and the weights of outgoing arcs of a node add up to one. 

At the top level of the JSON document, there are as many keys as there are nodes in the lattice, where each node has a unique integer identifier starting at zero. Each node has got three keys:
- `"stage"` Indicates at which layer the node is placed.
- `"state"` A JSON object with state variable names as keys and outcomes as values, e.g., `"state": {"demand": 10.3, "price": 21.45}`.
- `"successors"` A JSON object with nodes as keys and transition probabilities as values, e.g., `"successors": {2 : 0.4, 3: 0.6}`.

