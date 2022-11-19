---
layout: post
title: "Calculating Edit Distance Between Sequences"
date: 2000-03-30 12:41:00 EST
categories: algorithms nlp
---
# Calculating Edit Distance Between Sequences
## Introduction

This document describes a (simpler) algorithm closely related to the Viterbi decoding method for HMMs. 
The task is to calculate the "edit distance" between two sequences. 
This is defined as the cheapest way of using the elementary operations "insert","delete" and "substitute" to turn one sequence into the other. 
The algorithm below does this efficiently by filling up a square array of distances, starting from the bottom left and working to the top right. 
(the efficiency in question is time efficiency, but space is not too bad either, since typically not every entry in the array is relevant to the 
optimimum path, so there are various opportunities to save space if one wants to take the trouble). </p><p style="text-align:left;">
We work with a rectangular array of cells with dimensions *one bigger*
than the strings to be compared.

 
```py
def sdist(string1,string2):
  delCost = 1.0
  insCost = 1.0
  substCost = 1.0
  m = len(string1)
  n = len(string2)
  d[0,0] = 0.0
  ...
```

  We set up some variables, then seed the recursion with the left hand bottom element. 
  It would be easy to write another version in which the costs were different. 
  If the substitution cost is greater than the sum of the other two costs it will always be better to do 
  an insertion then a deletion, and no substitutions will occur in the minumum cost path.
```py
...
for i in range(m):
  d[i+1,0] = d[i,0] + delCost
...
```

We fill in the first row, adding entries with indices [1,0] through [m,0] ...

```py	
...
for j in range(n):
	d[0,j+1] = d[0,j] + insCost
...
```
  
  
<p style="text-align:left;">
We fill in the first column, adding entries with indices (0,1) through (0,n) </p><pre style="text-align:left;">
	for i below m
	    for j below n
		if string1(i) == string2(j)
		    subst = 0
		else
		    subst = substCost
                endif
		d(i+1,j+1) = min(d(i,j) +  subst,
				 d(i+1,j)+ insCost,
				 d(i,j+1)+ delCost)
           endfor
       endfor
</pre>
<p style="text-align:left;">
Once we have the edges of the array, we can work outwards, filling in the values which depend on these. We look at all three possible predecessors of each cell, taking the one which gives the lowest cost.
    his version says that there is no charge for matching a letter against itself, but that it costs one penalty point to match against anything else.
    It would be easy to vary this if we thought, for example, that it was less bad to confuse some letter pairs than to confuse others. </p>

<pre style="text-align:left;">	return d(m,n)
enddef sdist
</pre>
<p style="text-align:left;">

    At the end, the total distance is in the cell at (m,n). </p><p style="text-align:left;">
This scheme can be adapted to compute the sequence of edit operations rather than just the number. This is done by changing the element type of </p><pre style="text-align:left;">d</pre>
from <pre style="text-align:left;">integer</pre>
to <pre style="text-align:left;">traceback = integer *move* traceback option</pre>. where
<pre style="text-align:left;">move = INITIAL|DEL|INS|MATCH</pre>
(Here we are adopting terminology from the ML programming language. This means that there is a type of thing called a traceback which consists of an integer packaged with a field that points back to a further traceback). The idea is that the traceback information accessible from <pre style="text-align:left;">d(m,n)</pre>
is sufficient to recover the full description of a minimal cost path all the way back to <pre style="text-align:left;">d(0,0)</pre>.
<p style="text-align:left;">
We again start by setting up variables and seeding the recursion at </p><pre style="text-align:left;">d(0,0)</pre>.
<pre style="text-align:left;">def edit_path(string1,string2)
	delCost = 1.0
	insCost = 1.0
	substCost = 1.0
        m = len(string1)
        n = len(string2)
        d(0,0) = (0,INITIAL,None)
</pre>
<p style="text-align:left;">
This notation is intended to convey the fact that d(0,0) has cost zero, and that no further traceback information is necessary. </p><p style="text-align:left;">
Once again we fill in the edges: </p><pre style="text-align:left;">	for i below m
            (sofar,tb) = d(i,0) (*)
	    d(i+1,0) = (sofar + delCost,DEL,d(i,0)) (**)
        endfor
	for j below n
            (sofar,tb) = d(0,j) (*)
	    d(0,j+1) = (sofar + insCost, INS,d(0,j)) (**)
        endfor

</pre>
<p style="text-align:left;">
Here we are using the lines marked (*) to get the cost out of the predecessor, and the lines marked (**) to put appropriate information into the new cell. The traceback information in each new cell points back to its predecessor. For example </p><pre style="text-align:left;">d(0,2)</pre>
has cost <pre style="text-align:left;">2*insCost</pre>
and its traceback field points to <pre style="text-align:left;">d(0,1)</pre>. This in turn has cost
<pre style="text-align:left;">insCost</pre>
and points back to <pre style="text-align:left;">d(0,0)</pre>
which has cost 0 and no further traceback information. <p style="text-align:left;">
Finally, we do the main loop as before, except that more bookkeeping is needed to make sure that best paths are preserved. So </p><pre style="text-align:left;">min</pre>
is replaced by <pre style="text-align:left;">best</pre>, which is provided below.
<pre style="text-align:left;">	for i below m
	    for j below n
		if string1(i) == string2(j)
		    subst = 0
		else
		    subst = substCost
                endif
		d(i+1,j+1) = best ((subst,MATCH,d(i,j))
				   (insCost,INS,d(i+1,j))
				   (delCost,DEL,d(i,j+1)))
           endfor
       endfor

       return d(m,n)
enddef edit_path
</pre>
Now we just need <pre style="text-align:left;">best</pre>, which constructs the tuple that belongs in
<pre style="text-align:left;">d(i+1,j+1)</pre>. This is the one with the lowest cost. It's difficult to make this code clear enough. My best effort uses a little auxilliary function to pull the cost out of the traceback tuple
<pre style="text-align:left;">def cost(tb)
   c,m,rest = tb
   return c
enddef cost


def best(sub_move,ins_move,del_move)
    increment, move1, tb1 = sub_move
    cost_with_sub =  increment + cost(tb1)
    increment,move2,  tb2 = ins_move
    cost_with_ins =  increment + cost(tb2)
    increment,move3,  tb3 = del_move
    cost_with_del =  increment + cost(tb3)

    best_cost = cost_with_sub
    move = sub_move
    tb = tb1
    if cost_with_ins &#60; best_cost then
     move = ins_move
     tb = tb2
    endif
    if cost_with_del &#60; best_cost then
     move = del_move
     tb = tb3
    endif
    return cost,move,tb
enddef best

</pre>


Martin Jansche adds:: there is also the connection between string edit distance and weighted automata. The basic insight is that you view both strings as a special kind of automaton that has precisely one path from the start state to the final state, and you also need a weighted transducer that specifies what operations are possible and what weight is associated with each, then all you do is compose the three machines and find the least-cost path, whose cost is then the string edit distance. I've heard this being mentioned several times before, but I finally found it on some slides by Mohri, Pereira &#38; Riley that came with the AT&#38;T FSM library and toolkit as (or, in lieu of) some kind of documentation. </p>\


Alec Seward found some bugs in an earlier version of this page. I'm grateful. Also to Alex Martelli who found another typo and provided a Python implementation of the algorithm. If there are any more, please let me know.


<address style="text-align:left;"><a href="mailto:cbrew@acm.org" style="text-align:left;" target="_blank">Chris Brew</a></address>

## 22 years later (11-19-2022) Chris adds: 

This post resulted in the creation of at least two implementations. There's 

- A [Perl module called Text::Brew](https://metacpan.org/pod/Text::Brew) by Keith Ivey. 
- A [Python package](https://github.com/dhgutteridge/Brew-Distance) by David Gutteridge, 
 .
David comments that: 

> This implementation is useful for cases where real-time performance is not a consideration, but re-weighting edit types or the post-processing evaluation of edit steps is a requirement. (Also, as it's a simple, pure Python implementation, it could be easily customized, and it could be used as a sample implementation for educational purposes.) 

That's exactly what I had in mind. If you really need fast performance try [polyleven](https://github.com/fujimotos/polyleven) 
in Python or go looking for an implementation in another language. Many exist. 



