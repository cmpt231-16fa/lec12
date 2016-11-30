<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
# CMPT231
## Lecture 12: ch 25
### All-Pairs Shortest Paths

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
## James 3:13,17-18 <span class="ref">(NASB)</span>
Who among you is **wise** and **understanding**? <br/>
Let him show by his **good behavior** <br/>
his deeds in the **gentleness** of wisdom.

But the wisdom from above is first **pure**, <br/>
then **peaceable**, **gentle**, **reasonable**, <br/>
full of **mercy** and good **fruits**, <br/>
**unwavering**, without **hypocrisy**.

And the seed whose fruit is **righteousness**  <br/>
is sown in peace by those who **make peace**.

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
## Outline for today
+ **Single-source** shortest paths
  + **Dijkstra**: \`O(|V| log |V| + |E|)\`
+ **All-pairs** shortest paths
  + **Dynamic** programming by *path length*: \`O(|V|^4)\`
  + **Exponential** speedup: \`O(|V|^3 log |V|)\`
  + **Floyd-Warshall** (dyn prog by *vertex subset*): \`O(|V|^3)\`
  + **Johnson** (iterative Dijkstra): \`O(|V|^2 log |V| + |V||E|)\`

---
## Single-source shortest paths
+ **Output**: for each vertex *v* &in; V, store:
  + *v.parent*: links form a **tree** rooted at source
  + *v.d*: shortest-path **weight** from source
    + All initially *&infin;*, except *src.d* = *0*
+ Method: **edge relaxation**
  + Will **using** the edge *(u,v)* give us a
    **shorter** path to *v*?
+ Algorithms differ in **sequence** of relaxing edges

```
def relaxEdge( u, v, w ):
  if v.d > u.d + w( u, v ):
    v.d = u.d + w( u, v )
    v.parent = u
```

---
## Bellman-Ford algo for SSSP
+ Allows **negative-weight** edges
  + If any **net-negative** cycle is reachable, returns *FALSE*
+ **Relax** every edge, *|V|-1* times (**complexity**?)
+ Guaranteed to **converge**: shortest paths &le; *|V|-1* edges
  + Each **iteration** relaxes one edge along shortest path

<div class="imgbox"><div><pre><code data-trim>
def initSingleSource( V, E, src ):
  for v in V:
    v.d = &infin;
  src.d = 0

def ssspBellmanFord( V, E, w, src ):
  initSingleSource( V, E, src )
  for i in 1 .. |V| - 1:
    for (u,v) in E:
      relaxEdge( u, v, w )
  for (u,v) in E:
    if v.d > u.d + w( u, v ):
      return FALSE
</code></pre></div><div>
![Bellman-Ford: Fig 24-4(b)](static/img/Fig-24-4b.png)
</div></div>

---
## Dijkstra's algo for SSSP
+ **Doesn't** handle *negative-weight* edges
  + Assumes *adding* an edge always *increases* cost
+ *Weighted* version of **breadth-first** search
+ Use **priority queue** instead of FIFO
  + Keys are the **shortest-path** estimates *v.d*
  + Similar to **Prim** but calculates *v.d*
+ **Greedy** choice: select *u* with **lowest** *u.d*

<div class="imgbox"><div><pre><code data-trim>
def relaxEdge( u, v, w ):
  if v.d > u.d + w( u, v ):
    v.d = u.d + w( u, v )
    v.parent = u

def ssspDijkstra( V, E, w, src ):
  initSingleSource( V, E, src )
  Q = new PriorityQueue( V )
  while Q.notEmpty():
    u = Q.popMin()
    for v in E.adj[ u ]:
      relaxEdge( u, v, w )
</code></pre></div><div>
![Dijkstra, Fig 24-6(a)](static/img/Fig-24-6a.png)
</div></div>

---
## Dijkstra: correctness
+ **Invariant**: at top of loop, *u.d* = *&delta;(src,u)* &forall; *u* &notin; *Q*
+ **Proof**: suppose **not**:
  let *u* &notin; *Q* with *u.d* &ne; *&delta;(src,u)*
  + Assume *u* is the **first** such vertex popped from *Q*
  + &exist; **path** *src* &#x21DD; *u* (or else *u.d* = *&infin;* = *&delta;(src,u)*)

<div class="imgbox"><div><ul>
<li> Let <em>p</em> be a <strong>shortest</strong> path from <em>src</em> &#x21DD; <em>u</em> <ul>
  <li> Let <em>(x,y)</em> be the <strong>first</strong> edge in <em>p</em> crossing from <em>!Q</em> to <em>Q</em></li>
  <li> Since <em>u</em> is the <strong>first</strong> vertex to have <em>u.d</em> &ne; <em>&delta;(src,u)</em>,
  <br/> hence <em>x.d</em> = <em>&delta;(src,d)</em></li>
  </ul></li>
</ul></div><div>
![Dijkstra proof, Fig 24-7](static/img/Fig-24-7.svg)
</div></div>

---
## Dijkstra: correctness
+ So *x.d* = *&delta;(src,d)* (i.e., *x* has **converged**)
+ By **convergence** property, after we **relaxed** *(x,y)*,
  + because *(x,y)* is on the **shortest path** to *u*,
  + so now *y.d* = *&delta;(src,y)* (i.e., *y* has **converged**)
  + And *y* is on the **shortest path**, so *&delta;(src,y)* &le; *&delta;(src,u)* &le; *u.d*
+ But **both** *y* and *u* &in; *Q* when we `popMin()`, so *u.d* &le; *y.d*
+ Hence *u.d* = *y.d* = *&delta;(src,u)*, a **contradiction**

![Dijkstra proof, Fig 24-7](static/img/Fig-24-7.svg)
<!-- .element: style="width:60%" -->

---
## Dijkstra: complexity
+ **Initialisation**: &Theta;( *|V|* )
+ *Q.popMin()* is run exactly *|V|* times
+ *Q.setPriority()* (called by *relaxEdge*) is run O( *|E|* ) times
+ Using **binary min-heaps**, all ops are O( *lg |V|* )
  + **Total** time: O( *|E| lg |V|* )
+ Using **Fibonacci heaps**:
  + *Q.setPriority()* takes only O(*1*) **amortised** time
  + **Total** time: O( *|V| lg |V|* + *|E|* )

---
## Single-source shortest paths
+ Generic **outline**: *relax* edges
  + To iteratively find *shortest distance* *&delta;* to each vertex
+ **Bellman-Ford**: total time O( *|V| |E|* )
  + Relax *all* edges, *|V|-1* times
  + Only one pass needed if *acyclic*, using *topological sort*
+ **Dijkstra**: O( *|V| lg |V|* + *|E|* )
  + *BFS* from source, updating *shortest distance* as we go
  + Use *Fibonacci heap* for *priority queue*
  + Cannot handle *negative-weight* edges
    + e.g., *forex* (currency exchange): profit/loss
    + e.g., *energy* (chemical reactions): gain/loss of energy

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
## Outline for today
+ Single-source shortest paths
  + Dijkstra: \`O(|V| log |V| + |E|)\`
+ **All-pairs shortest paths**
  + **Dynamic programming by path length:** \`O(|V|^4)\`
  + **Exponential speedup:** \`O(|V|^3 log |V|)\`
  + Floyd-Warshall (dyn prog by vertex subset): \`O(|V|^3)\`
  + Johnson (iterative Dijkstra): \`O(|V|^2 log |V| + |V||E|)\`

---
## All-pairs shortest paths
+ **Input**: directed graph (*V*, *E*), weights *w*
+ **Output**: *|V|* x *|V|* matrix *D* = \`(delta\_(ij))\`
  + **Shortest-path** distances from each vertex *i* to *j*
+ **Naive** solution: do *Bellman-Ford* for each vertex:
  + \`O(|V|^2 |E|)\`: if **dense**, then *|E|* \`in Theta(|V|^2)\`,
    so \`O(|V|^4)\`
+ If no *negative* edges, try **Dijkstra**, *|V|* times:
  + **Binary heap**: \`O( |V| |E| log |V| )\`,
    \`= O(|V|^3 log |V|)\` if *dense*
  + **Fibonacci heap**: \`O(|V|^2 log |V| + |V||E|)\`,
    \`= O(|V|^3)\` if *dense*
+ **Floyd-Warshall**: \`O(|V|^3)\`, *even* if dense,
  <br/> and handles *negative* edges

---
## Dynamic prog: recurrence
+ Recall we have **optimal substructure**:
  + *Subpaths* of shortest paths are *themselves* shortest paths
+ Use **weighted adjacency matrix** *W*:
  + \`w\_(ij)\` = *w(i,j)* for all **edges**
  + \`w\_(ii)=0\`, and \`w\_(ij) = oo\` if **no** edge *(i,j)*
+ Let \`l\_(ij)^((m))\` be the **length** (weight) of a shortest path
  from *i* to *j* which only uses &le; *m* **edges**
  + **Recurrence**:
    \`l\_(ij)^((m)) = min\_(k in V)( l\_(ik)^((m-1)) + w\_(kj) )\`
  + **Base**:
    \`l\_(ii)^((0)) = 0\`, and \`l\_(ij)^((0)) = oo\`, for i &ne; j
+ After **one** iteration: \`l\_(ij)^((1)) = w\_(ij)\`
  + After &ge; *|V|-1* iterations: \`l\_(ij)^((m)) = delta\_(ij)\`
    (i.e., **solved**)

---
![Example of dynamic programming solution, Fig 25-1](static/img/Fig-25-1.svg)
<!-- .element: style="width:80%" -->

---
## Dynamic prog: bottom-up

```
def APSPExtend( L, W ):
  let M = new matrix, |V| x |V|
  for i in 2 .. |V|:
    for j in 1 .. |V|:
      M[ i, j ] = infinity
      for k in 1 .. |V|:
        M[ i, j ] = min( M[ i, j ], L[ i, k ] + W[ k, j ] )
  return M
```

+ **Initialise** (*m=1*): \`L^((1))=W\`, the weighted *adjacency matrix*
+ **Extend** paths \`L^((m-1))\` of length *m-1* to length *m*: \`L^((m))\`
  + Do this *|V|-2* times to get **solution**: \`L^((|V|-1))\`
+ **Complexity**: each *Extend* is \`Theta(|V|^3)\`
  + So **total** is \`Theta(|V|^4)\`: not too hot...
+ **But**: notice *Extend* looks a lot like a **matrix multiply**!

---
## Exponential bottom-up
+ **Matrix power** \`A^n\` can be calculated by
  + \`A \* A \* ... \* A\` (*n-1* times), or by
  + \`A^2 = A \* A\`, then \`A^4 = A^2 \* A^2\`, \`A^8 = A^4 \* A^4\`, etc.
    + Only need *lg n* multiplications
+ Now apply this to *APSPExtend*'s triply-nested *for* loop
  + If *n* is not a **power of 2**, we'll overshoot
  + That's ok, it **converges** after \`L^((|V|))\`
+ **Complexity**: \`Theta(|V|^3 log |V|)\`: much better!

```
def Fast_APSP( W ):
  L = W
  for m in 2 .. ceiling( lg |V| ):
    L = APSPExtend( L, L )
  return L
```

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
## Outline for today
+ Single-source shortest paths
  + Dijkstra: \`O(|V| log |V| + |E|)\`
+ All-pairs shortest paths
  + Dynamic programming by path length: \`O(|V|^4)\`
  + Exponential speedup: \`O(|V|^3 log |V|)\`
  + **Floyd-Warshall (dyn prog by vertex subset):** \`O(|V|^3)\`
  + Johnson (iterative Dijkstra): \`O(|V|^2 log |V| + |V||E|)\`

---
## Floyd-Warshall: substructure
+ **Subtasks**: not by *path length*, but by **subset** of *V*:
  + \`d\_(ij)^((k))\` = weight of shortest-path
    from *i* to *j* where <br/> all **intermediate** vertices
    are in the subset *{1 .. k}* of vertices
+ Let \`p\_(ij)\` be **shortest** path *i* &#x21DD; *j* w/nodes in *{1 .. k}*
  + If *k* is **not** on the path, then *p* has vertices
    only in *{1 .. k-1}*
  + If *k* is **on** the path, then **split** *p*
    into \`p\_(ik) and p\_(kj)\`
    + These two subpaths have intermediate vertices only in *{1 .. k-1}*
  + By **induction**, the subpaths are **optimal**
  + Thus, every **shortest** path in *{1 .. k}* has subpaths <br/>
    that are themselves **shortest** paths

![Floyd-Warshall proof, Fig 25-3](static/img/Fig-25-3.svg)
<!-- .element: style="width:60%" -->

---
## Floyd-Warshall: solution
+ **Recurrence**: \`d\_(ij)^((k))
  = min(d\_(ij)^((k-1)), d\_(ik)^((k-1)) + d\_(kj)^((k-1)))\`

```
def FloydWarshall( W ):
  D = W
  for k in 1 .. |V|:
    for i in 1 .. |V|:
      for j in 1 .. |V|:
        D'[ i, j ] = min( D[ i , j ], D[ i, k ] + D[ k, j ] )
    D = D'
  return D
```

<div class="imgbox"><div><ul>
<li> Either <em>k</em> <strong>not</strong> on path, or <br/>
  two <strong>subpaths</strong> through <em>k</em> </li> <ul>
  <li> <strong>Base</strong> case: \`d\_(ij)^((0))=w\_(ij)\` </li>
  <li> Final <strong>solution</strong>: \`D^((n)) = (d\_(ij)^((n)))\` </li>
  </ul></li>
<li> <strong>Complexity</strong>: \`Theta(|V|^3)\` </li>
</ul></div><div>
![Example Floyd-Warshall: Fig 25-2](static/img/Fig-25-2.svg)
</div></div>

---
## F-W for transitive closure
+ The **transitive closure** \`(V, E\*\*)\` of *(V, E)* is:
  + *(i,j)* &in; \`E\*\*\` &hArr; &exist; **path** *i* &#x21DD; *j*
    + i.e., if *j* is **reachable** from *i*
+ One method: run **Floyd-Warshall** with *w=1* on all edges
  + &exist; path *i* &#x21DD; *j* iff \`d\_(ij) < oo\`
+ Even **simpler**: \`d\_(ij)^((k))\` is just a **bit** (boolean)
  + Tracks if path *i* &#x21DD; *j* **exists** through *{1 .. k}*
+ **Recurrence**: \`d\_(ij)^((k))
  = d\_(ij)^((k-1)) or (d\_(ik)^((k-1)) and d\_(kj)^((k-1)))\`
  + Same *pseudocode* structure as Floyd-Warshall
  + Still \`Theta(|V|^3)\`, but **bitwise** logical ops are
    even *faster*!

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
## Outline for today
+ Single-source shortest paths
  + Dijkstra: \`O(|V| log |V| + |E|)\`
+ All-pairs shortest paths
  + Dynamic programming by path length: \`O(|V|^4)\`
  + Exponential speedup: \`O(|V|^3 log |V|)\`
  + Floyd-Warshall (dyn prog by vertex subset): \`O(|V|^3)\`
  + **Johnson (iterative Dijkstra):** \`O(|V|^2 log |V| + |V||E|)\`

---
## Dijkstra for all-pairs
+ For **sparse** graphs, running **Dijkstra** on each vertex
  is **faster** than Floyd-Warshall:
  + If *|E|* = \`o(|V|^2)\`, then (using *Fibonacci heaps*)
  \`O(|V|^2 log |V| + |V||E|) = o(|V|^3)\`
+ But Dijkstra can't handle **negative weights**:
  + Need to **reweight** without changing shortest paths
+ Given \`h:V -> RR\`, let *w'(u,v)* = *w(u,v)* + *h(u)* - *h(v)*
  + *p* is a **shortest path** *u* &#x21DD; *v* under *w* <br/>
    &hArr; *p* is a **shortest path** under *w'*
  + *w'(p)* = *w(p)* + *h(u)* - *h(v)* is **indep** of
    intermediate vertices
+ This reweighting also preserves **negative-weight** cycles

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" -->
## Outline for today
+ **Single-source** shortest paths
  + **Dijkstra**: \`O(|V| log |V| + |E|)\`
+ **All-pairs** shortest paths
  + **Dynamic** programming by *path length*: \`O(|V|^4)\`
  + **Exponential** speedup: \`O(|V|^3 log |V|)\`
  + **Floyd-Warshall** (dyn prog by *vertex subset*): \`O(|V|^3)\`
  + **Johnson** (iterative Dijkstra): \`O(|V|^2 log |V| + |V||E|)\`

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DLwUVlzrP0Q-waves_rocks.jpg" class="empty" -->
