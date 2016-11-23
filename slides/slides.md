# CMPT231
## Lecture 12: ch 25, 34
### Shortest Paths, NP Completeness

---
## Devotional

---
## Outline for today
+ Single-source shortest paths: Dijkstra
+ All-pairs shortest paths
+ Tractability

---
## Dijkstra's algo for SSSP
+ **Doesn't** handle negative-weight edges
+ Weighted version of **breadth-first** search
+ Use **priority queue** instead of FIFO
  + Keys are the **shortest-path** estimates *v.d*
  + Similar to **Prim** but calculates *v.d*
+ **Greedy** choice: select *u* with **lowest** *u.d*

<div class="imgbox"><div><pre><code data-trim>
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
+ **Proof**: suppose **not**: <br/>
  let *u* be the **first** vertex popped from *Q*
  with *u.d* &ne; *&delta;(src,u)*
  + &exist; **path** *src* &#x21DD; *u* (or else *u.d* = *&infin;* = *&delta;(src,u)*)
+ Let *p* be a **shortest** path *src* &#x21DD; *u*
  + Let *(x,y)* be the **first** edge in *p* crossing from *!Q* to *Q*
  + So *x.d* = *&delta;(src,d)*, since *u* is **first** with *u.d* &ne; *&delta;(src,u)*
+ We **relaxed** *(x,y)*, so *y.d* = *&delta;(src,y)* (by **convergence**)
  + And *y* is on the shortest path, so *&delta;(src,y)* &le; *&delta;(src,u)* &le; *u.d*
+ But **both** *y* and *u* &in; *Q* when we `popMin()`, so *u.d* &le; *y.d*
+ Hence *u.d* = *y.d* = *&delta;(src,u)*, a **contradiction**

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
+ **Bellman-Ford**:
  + Relax *all* edges, *|V|-1* times
  + **Total** time: O( *|V| |E|* )
  + Only one pass needed if *acyclic*, using *topological sort*
+ **Dijkstra**:
  + *BFS* from source vertex
  + Update *shortest distance* as we go
  + Use *Fibonacci heap* for *priority queue*
  + **Total** time: O( *|V| lg |V|* + *|E|* )

---
## Outline


