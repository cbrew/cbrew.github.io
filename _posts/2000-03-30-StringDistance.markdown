---
layout: post
title: "Calculating Edit Distance Between Sequences"
date: 2000-03-30 12:41:00 EST
categories: algorithms nlp
---
## Introduction

This document describes a (simpler) algorithm closely related to the Viterbi decoding method for HMMs. 
The task is to calculate the "edit distance" between two sequences. 
This is defined as the cheapest way of using the elementary operations "insert","delete" and "substitute" to turn one sequence into the other. 
The algorithm below does this efficiently by filling up a square array of distances, starting from the bottom left and working to the top right. 
(the efficiency in question is time efficiency, but space is not too bad either, since typically not every entry in the array is relevant to the 
optimum path, so there are various opportunities to save space if one wants to take the trouble).
We work with a rectangular array of cells with dimensions *one bigger*
than the strings to be compared.

 
```py
def sdist(string1, string2):
    delCost = 1.0
    insCost = 1.0
    substCost = 1.0
    m = len(string1)
    n = len(string2)
    d = dict()
    d[0, 0] = 0.0
```

  We set up some variables, then seed the recursion with the left hand bottom element. 
  It would be easy to write another version in which the costs were different. 
  If the substitution cost is greater than the sum of the other two costs it will always be better to do 
  an insertion then a deletion, and no substitutions will occur in the minumum cost path.
```py
 d[0, 0] = 0.0
    for i in range(m):
        d[i + 1, 0] = d[i, 0] + delCost

```

We fill in the first row, adding entries with indices [1,0] through [m,0] ...

```py	
   for j in range(n):
        d[0, j + 1] = d[0, j] + insCost
```
  
We fill in the first column, adding entries with indices [0,1] through [0,n] 


```python
for i in range(m):
        for j in range(n):
            if string1[i] == string2[j]:
                subst = 0
            else:
                subst = substCost
            d[i + 1, j + 1] = min(
                d[i, j] + subst, d[i + 1, j] + insCost, d[i, j + 1] + delCost
            )
```

Once we have the edges of the array, we can work outwards, filling in the values which depend on these. We look at all three possible predecessors of each cell, taking the one which gives the lowest cost.
	
This version says that there is no charge for matching a letter against itself, but that it costs one penalty point to match against anything else.
It would be easy to vary this if we thought, for example, that it was less bad to confuse some letter pairs than to confuse others.

At the end, the total distance is in the cell at [m,n]. 

This scheme can be adapted to compute the sequence of edit operations rather than just the number. 
This is done by changing the element type of the array from integer to a *traceback* consisting of a cost, a move
and a reference to the previous traceback. In modern Python code one way to express this is as below:
	
```python
class Move(Enum):
    MATCH = 0
    INS = 1
    DEL = 2
    SUBST = 3
    INITIAL = 4


class Traceback(NamedTuple):
    cost: int
    move: Move
    traceback: Optional["Traceback"]
```

The idea is that the traceback information accessible from `d[m,n]` is sufficient to recover the full description of a minimal cost path all the way back to `d[0,0]`.

We again start by setting up variables and seeding the recursion at `d[0,0]`.
	

```python
def edit_path(string1, string2) -> Traceback:
    len1 = len(string1)
    len2 = len(string2)
    match_cost = 0
    ins_cost = 1
    del_cost = 1
    subst_cost = 1
    initial_cost = 0
    distances = dict()
    distances[0, 0] = Traceback(cost=initial_cost, move=Move.INITIAL, traceback=None)
```
This code says that distances[0,0] has cost zero, and that no further traceback information is necessary. 
	
Once again we fill in the edges.

```python
    # Deletions
    for i in range(0, len1):
        so_far = distances[i, 0].cost
        distances[i + 1, 0] = Traceback(
            cost=so_far + del_cost, move=Move.DEL, traceback=distances[i, 0]
        )

    # Insertions
    for j in range(0, len2):
        so_far = distances[0, j].cost
        distances[0, j + 1] = Traceback(so_far + ins_cost, Move.INS, distances[0, j])
```

Here we are first getting the cost out of the predecessor, then putting appropriate information into the new cell. The traceback information in each new cell points back to its predecessor. For example `distances[0,2]` has cost 2
and its traceback field points to `distances[0,1]`. This in turn has cost `insCost`and points back to `distances[0,0]`
which has cost 0 and no further traceback information.
	
Finally, we do the main loop as before, except that more bookkeeping is needed to make sure that best paths are preserved. So `min`
is replaced by `best`, which is provided below.
	
```python
    # Substitutions
    for i in range(0, len1):
        for j in range(0, len2):
            if string1[i] == string2[j]:
                subst = match_cost
            else:
                subst = subst_cost

            distances[i + 1, j + 1] = best(
                Traceback(subst, Move.SUBST if subst else Move.MATCH, distances[i, j]),
                Traceback(ins_cost, Move.INS, distances[i + 1, j]),
                Traceback(del_cost, Move.DEL, distances[i, j + 1]),
            )
    return extract_path(distances[len1, len2])
```

Now we just need `best`, which selects the tuple that belongs in `d(i+1,j+1), This is the one with the lowest cost. 
Pythonistas will probably realize that best is really a version of the method that is called `argmax` in Pandas and Numpy.

If you follow the tracebacks, the operations come out in reverse order, so for completeness, we need `extract_path`, which gets 
the edit operations and costs in their correct order. 

```python
def best(sub_move, ins_move, del_move):
    chosen = sub_move
    if ins_move.cost < chosen.cost:
        chosen = ins_move
    if del_move.cost < chosen.cost:
        chosen = del_move
    return chosen
	
def extract_path(tb:Traceback):
    """
    Extract the series of operations that
    maps one string to the other.
    """
    result = []
    while tb.move != Move.INITIAL:
        result.append((tb.cost,tb.move))
        tb = tb.traceback
    ops = result[::-1]
    return ops
```
This gives the following results
```
foo foot [(1, <Move.INS: 1>), (1, <Move.SUBST: 3>), (0, <Move.MATCH: 0>), (1, <Move.SUBST: 3>)]
foo foo [(0, <Move.MATCH: 0>), (0, <Move.MATCH: 0>), (0, <Move.MATCH: 0>)]
```

Martin Jansche adds:: there is also the connection between string edit distance and weighted automata. The basic insight is that you view both strings as a special kind of automaton that has precisely one path from the start state to the final state, and you also need a weighted transducer that specifies what operations are possible and what weight is associated with each, then all you do is compose the three machines and find the least-cost path, whose cost is then the string edit distance. I've heard this being mentioned several times before, but I finally found it on some slides by Mohri, Pereira &#38; Riley that came with the AT&#38;T FSM library and toolkit as (or, in lieu of) some kind of documentation. </p>\


Alec Seward found some bugs in an earlier version of this page. I'm grateful. Also to Alex Martelli who found another typo and provided a Python implementation of the algorithm. If there are any more, please let me know.


<address style="text-align:left;"><a href="mailto:cbrew@acm.org" style="text-align:left;" target="_blank">Chris Brew</a></address>

## 22 years later (11-19-2022) Chris adds: 

I have revised the original a little. Since Python is now much better known, the demo code is Python. 
There's now [running code](https://github.com/cbrew/Brew-Distance.git) (a fork of David Gutteridge's package).
David's code has all the paraphernalia of a real Python package, with tests, build files and all that. Mine is Python 3.9+ only and
the tests are not yet updated.

This post resulted in the creation of at least two implementations. There's 

- A [Perl module called Text::Brew](https://metacpan.org/pod/Text::Brew) by Keith Ivey. 
- A [Python package](https://github.com/dhgutteridge/Brew-Distance) by David Gutteridge. 


David comments that: 

> This implementation is useful for cases where real-time performance is not a consideration, but re-weighting edit types or the post-processing evaluation of edit steps is a requirement. (Also, as it's a simple, pure Python implementation, it could be easily customized, and it could be used as a sample implementation for educational purposes.) 

That's exactly what I had in mind. If you really need fast performance try [polyleven](https://github.com/fujimotos/polyleven) 
in Python. 

### Other text processing tools

[OpenFST]{https://www.openfst.org/twiki/bin/view/FST/WebHome} is an open version of the A T &ampersand; T toolkit that Martin Jansche mentioned above. 
It's worth your time if you want fast, efficient and composable low-level text processing tools. 
There is also a higher-level language called [Thrax]{https://www.openfst.org/twiki/bin/view/GRM/Thrax} which can greatly speed development.



