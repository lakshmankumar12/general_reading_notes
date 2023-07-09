Algorithm Analysis

:toc:

# Asymptotic analysis

```
n! << 2**n  << n**3  << n**2  << n log n << n << log n << 1
```

Now other functions of n

* `n**0.5` or square-root of n
* `5**n` vs `2**n`
* `0.5**n` vs plain-n
* `2**sqrt(n)`

# Master Method

* Applies when all sub-problems are of same size.

`
T(n) =  a T(n/b) + O(n**d)
`

## Cases

* case1: `a=b**d   : T(n) = O((n**d)logn)`
* case2: `a<b**d   : T(n) = O(n**d)`
* case3: `a>b**d   : T(n) = O(n**logba)`

* a and `b**d` are the opposing forces.
* a is the bad force, which tracks how the sub-problem
  proliferates. More a its bad.
* `b**d` is the good force. It tracks by how much work gets
  reduced(b) at a given level. This raised to d, more we gain,
  as each level is working in as much smaller set.

# Data Structures

## Strings

### Substring Algorithms

#### Boyer-Moore

* Explained in Goodrich/Tamassia/Goldwasser
* Idea is to build a hash for each char, the last index it
  appears in pattern. If a char is non-existent, then make it -1.
* So, when a mismatch happens, we check if the mismatching char is
  present in sub-string or not. If not we slide the sub-string
  fully past there. If not, we slide the string till the index
  we know its present.
* Since we store the last index, we start matching from end of
  pattern to beg of pattern - so that the last-index adjusting
  works.

#### KMP (Knuth Morris Pratt) Pattern Searching

* Explained in http://www.geeksforgeeks.org/searching-for-patterns-set-2-kmp-algorithm/[geeks for geeks]
    ```
    lps[i] = the longest proper prefix of pat[0..i]
    which is also a suffix of pat[0..i].
    ```
* Having built this lps array, when a mismatch happens at a index, move the
  substring by the lps[i-1]th spaces - as we know the prefix of that size is
  already found
* But trick is to build lps array! I didn't get this fully. See at geeks site.
    ```
    def compute kmp fail(P):
      '''Utility that computes and returns KMP fail list.'''
      m = len(P)
      fail = [0]*m           # by default, presume overlap of 0 everywhere
      j = 1
      k = 0
      while j < m:           # compute f(j) during this pass, if nonzero
        if P[j] == P[k]:     # k + 1 characters match thus far
          fail[j] = k + 1
          j += 1
          k += 1
        elif k > 0:          # k follows a matching prefix
          k = fail[k−1]      #  <-- This is what i dont follow
                             #      Also note no increment of j
        else:                # no match found starting at j
          j += 1
      return fail
    ```

# Sorting and Related Algorithms

## Inversion

The inversion count for any array is the number of steps it will take for the
array to be sorted, or how far away any array is from being sorted. If we are
given an array sorted in reverse order, the inversion count will be the maximum
number in that array

Inversion will be counted if arr[i] is greater than arr[j] where i is lesser
than j.

Its some number between 0(already sorted) and nC2(reverse sorted)

Eg:

```
[1,2,4,3,5] -- There is 1 inversion (4,3)
[5,4,3,2,1] -- there are 10 inversions [(5,4),(5,3),(4,3),(5,2)...(2,1)]
[1,2,3,4,5] -- 0.
```

## Counting Inversions

* This is same as merge sort. As part of merging, just count inversions.
* Remember, u should also do sorting alongside.

## Max-sum contiguous array

### Problem

Given an array arr[] of integers(postive/negative) size N. The task is to
find the sum of the contiguous subarray within a arr[] with the largest sum.

Eg:
```
[ -2, -3, 4, -1, -2, 1, 5, -3 ]
          <------------->
          Max=7

```

### Kadane's algorithm.

* Keep tracking contiguous sum. If that falls below zero, the prefix so
  far is no good and doesn't contribute to answer. So, start window
  from next index.
* Otherwise, keep including current index. Keep updating best answer
  at every point
* The following implementation gives the max-sum assuming its positive,
  otherwise zero. If you want to include 0/-ve numbers in your answer,
  you will have to add updates in the go-negative part

```
def max_contiguos_subarray(A):
    max_ending_here = max_so_far = 0
    bs = bl = 0  #best start, last
    cs = cl = 0  #curr_start, last
    for i,x in enumrate(A):
        max_ending_here = max(0, max_ending_here + x)
        if not max_ending_here:
          cs = i+1
          cl = i+1
        else:
          cl = i
        if max_so_far < max_ending_here:
          max_so_far = max_ending_here
          bs = cs
          bl = cl
    if max_so_far:
      return (max_so_far, bs, bl)
    else:
      return (0, None, None)
```

# Binary Trees

* A tree is also a graph. Any undirected graph, where any 2 vertices are
  connected in exactly one path is a tree. In other words any connected
  graph, without cycles is a tree.

## Diameter of a binary tree

* Treating a tree as a graph, the diameter is the longest path from one
  leaf node to another leaf node.
* To find the diameter of a tree, try this algorithm as
  explained in this
  http://www.geeksforgeeks.org/diameter-of-a-binary-tree/[geek-for-geek]
* If you are given a graph and u want to find the diameter, try this
  http://stackoverflow.com/questions/25649166/linear-algorithm-of-finding-tree-diameter[stack-overflow]
  idea.

## Binary Search Tress

* Each node is such that all the left nodes are lesser and right nodes are
  higher.
* Note that if tree isn't balanced, there are many ways of building a tree for
  the same set of numbers. Balancing is a different act.
* Options for balancing
    * Red-Black Trees
    * AVL Trees
    * Splay Tress
    * B Trees

### Operations comparison

 Operations                   | Running time for BST  | Running time for sorted array  | Comment
 ----------                   | ------------          | -----------
 Search                       | O(log n)              | O(log n)                       | Same
 Select ith order statistic   | O(log n)              | O(1)                           | Bst costlier
 Min/Max                      | O(log n)              | O(1)                           | Bst costlier
 Predecessor/Successor        | O(log n)              | O(1)                           | Bst costlier
 Rank                         | O(log n)              | O(log n)                       | Same
 Output in sorted order       | O(n)                  | O(n)                           | Same
 Insert                       | O(log n)              | O(n)                           | Bst cheap
 Delete                       | O(log n)              | O(n)                           | Bst cheap

### Operations

#### Min/Max

* Start from root.
* Keep following left(or right) points until there is no left pointer. Return
  the leaf node you are at, that has no left(right) child, which will be the
  minimum so far.

#### Predecessor of a key k. (Turn words for Successor)

* If left-child is non-empty, get the max element of this sub-tree
  (by travelling right pointers)
* If left-child is empty, keep travelling up parent pointers, until u find a
  parent that is lesser in value.
    * This node is higher than parent if its a right-child. In that case,
      parent is the lesser valued node of interest.
        * [TO FIND ]Why can't there be an element between this parent and
          this node?
    * This node is lesser than parent (then parent is higher) and we are
      left child. So, keep moving up until the current-node is a right-child.

#### Deleting a node

* Search node first if just key is given.
* If node has no children, delete node.
* If node has just one child, just replace that child in place of the node.
* If node has 2 children, compute k's predecessor (That is the right most node
  of left child)
    * Swap that predecessor node and this node and remote it off.
    * This will equally work by using the successor.

#### Selecting ith order statistic

Unclear: Define ith order statistic

* Store the size of tree below in every node.
    * Let size(a) be the size of tree at a given node.
    * By defn, sizeof(a) = 1 + sizeof(a->left) + sizeof(a->right)
    * sizeof(a->left) = 0 if a has no left child.
* Algo
    ```
    def get_ith_statistic(i,root):
      ''' i is from 0 to n-1 '''
      a=root
      if i == sizeof(a->left):
        return a
      if i < sizeof(a->left):
        get_ith_statistic(i,a->left)
      else:
        get_ith_statistic(i-(sizeof(left))-1,a->right)
    ```

Unclear: what is rank

* Rank (My own..not vetted from sources)
    * Find node first
    * Rank is simply sizeof(a->left)+1

#### Rotations

* Rotations are fundamental to any balancing of binary tress.
* Its basically rewiring of pointers.
    ```

         y                   x
        / \    right       /  \
      x     C  ----->     A    y
     / \       <-----         / \
    A   B      left          B   C

    Note that in both arrangements:   A < x < B < y < C
    ```

### Red Black Trees

#### Rules

* Each node is red or black
* Root is black
* No 2 reds come in row
    * [red node => black children]
* Every root->NULL (leaf path, like in unsuccessful search) will have equal
  number of black nodes.

#### Observations

* Because of the rules, there can be utmost log(n+1) black nodes in every
  path.
* Because of the rules, there can be utmost log(n+1) red nodes stuffed in
  between black nodes.
* So the tree is utmost 2 log (n+1) height tall.
* Note a fully black tree is also a valid RB tree.

#### Inserting in a RB Tree

* We will refer the node to insert as N, parent as P, grand-parent as G and
  sibling of P as uncle(U)
* We just Regular node insert in a BST and color this node N red (except in
  case 1).
* Case 1
   * Tree is empty. N is the root and is black
* Case 2: Parent(P) is black.
    * No property is disturbed by adding a Red child N.
* Case 2: Parent and Uncle are both Red.
    * If parent(P) is red, we are violating - as there are 2 consecutive reds.
      However, note that here grand-parent(G) definitely exists and is black,
      by RB-tree rules. Root has to be black - so there is definitely one
      parent to P. G, by definition has to be Black, as P is already red.
        ```


                 G-B
               /   \
             P-R    U-R
             /
           N-R     (N could be placed anywhere in the 4 slots, its holds good)
        ```
    * recolor P and U as Black and G as red. This wont break the rule-4 (equal
      blacks on path, we are only bringing black one level below)
        ```


                 G-R
               /   \
             P-B    U-B
             /
           N-R     (N could be placed anywhere in the 4 slots, its holds good)
        ```
    * But rule-3 of no red's together may or may not be broken. If we are lucky,
      G's parent is Black, then we can stop. If not, we need to repeat this
      treating G as the new node added. (Beneath G rule-4 is preserved)
* Case 4: Parent is R, and Uncle is B. N is added as a left child of parent,
  which is a right child of G. (Or vice versa)
    * (Self note:) This arrangement can never happen when we start, as at a level
      how can there originally be a P-node as leaf and red, with its sibling a
      black. Isn't this violating rule-4? Perhaps this is a possible configuration
      as we iterate on case-3 above and proceed to top?
    * In this case, we rotate N and P and make it same as case-5 below
        ```


                 G-B                    G-B
               /   \                  /   \
             P-R    U-B   ==>       N-R    U-B
            /  \     / \           /  \     / \
           A   N-R  D   E        P-R   C   D   E
               / \               /\
              B   C             A  B
        ```
* Case 5: Parent is R, and Uncle is B. N is added as a left child of parent,
    which is a left child of G.
    * Rotate as shown below. Note that originally G-U path had 2 blacks and that
      is still maintained. This wasn't violated when we started with (what was
      violated was 2 consecutive reds and that is now fixed)
    ```


             G-B                         P-R
           /   \                       /   \
         P-R    U-B                  N-R    G-R
        /  \     / \     ==>        /  \     / \
      N-R   C   D   E              A    B   C   U-B
      /\                                       / \
     A  B                                     D   E
    ```

#### Deletion

## Ternary Search Tress

* Each node has a data-member (one char - or trie's equivalent), and an
  optional end-of-word marker. The end of word doesn't necessarily imply
  termination of search. For eg, cat, cats will have cat's t having
  end-of-word, but there is also a cats
* Each Node has a {lo,eq,hi}-kid pointer. I personally want to call it
  low-peer, high-peer and child pointers
* Operations are insert, search, get-next-char, get-next-string,
  get-all-substrings.

### Links to read

* http://www.drdobbs.com/database/ternary-search-trees/184410528[Dobbs journal]

# Heaps

* A simple binary tree-like looking structure where the only condition is that
  the parent node is less(or greater) than its children. This ensure the root
  is the min(or max) element
    * The tree is naturally full-complete or partially complete
    * There are many ways of arranging a given set of numbers in a heap - as
      the only condition is the heap-property.
* Two main operations that it supports
    * insert
    * extract-min
* The next operations are
    * heapify
        * This will prepare a heap from a random collection of items.
        * The standard way will take n $$*$$ log n time. But, there is a
          slick way to do it in O(n), if all numbers are available a-priori.
        * This is explained in this http://stackoverflow.com/a/9755805/2587153[stack overflow answer]
          and is further explained why heapify is O(n) and heap-sort is still
          O(nlogn) in this http://stackoverflow.com/a/18742428/2587153[answer]
        * The idea is that when you heapify from the leaf nodes, n/2 nodes have
          0 operations, n/2 have 1 operation, n/4 have 2,.. and only the root
          has log(n) operations. So is accepted to be O(n)
            ```
            (0 * n/2) + (1 * n/4) + (2 * n/8) + ... + (h * 1).
            ```
        * However, in case of heap sort itself, work for each node decreases by
          size of 1.

## Uses of heap

* Heap sort
* find median in a collection of number
* Any algo that needs to keep picking minimum, like Dijkstra's shortest path
  algo

## Implementation

* Heaps are usually implemented as trees. But array way of representing heaps
  are more common.
* In a 0-based array
    ```
    For, index i
    2i+1 and 2i+2 are its 2 children
    i-1/2 is its parent
    ```
* For sifting up, add a node to the end of tree. Shift it up, till it is at the
  right position. This is part of the heapify operation.
* Extract min is simply taking the root first. Now swap the last node of the
  heap as root and bubble down.

# Hashes

* Every effective of lookups / insertions / deletions
* Can't handle the sort/ordering of keys
    * min-max, next/prev, select/rank are out.
* Typically the Universe(U) of all possible elements is too huge and we deal
  with a small subset of elements(S) at a given time.
    * Hash has number of bins that is comparable to the cardinality of S
* It takes just sqrt(n) elements to have a 50% probability of collision, even
  if n elements have equal probability of coming.
    * What that means is , even if u have 10K buckets in your hash, and your
      universe is pretty HUGE and there is a probability for an element to take
      any of the 10K buckets, then it still takes only 100+ elements to have a
      50% probability for collision!
* Prevalent options for handling collisions
    * Chaining
        * Simple linked list at each bucket
    * Open addressing
        * Basically put the elements in the next empty slot. BUt how to derive
          this next slot can be different
        * Linear probing
            * Just keep searching n+1....Nth.0th..n-1th slots after nth slot is
            taken.
        * Double hashing.
            * Improvement over Linear probing, where a second hash-function
              gives an offset. This liner probing has a offset of always 1.
        * How to stop searching?
            * Stop when you find an empty slot.
            * To distinguish deleted and really empty, you have a tri-state
              for slots - filled, empty, deleted.
        * The only adv of open-addressing is that it is space-effective.
          (doest waste linked list keeping)
* Load of a Hash-table
    ```
      Load = No of elements
            -----------------
             No of buckets
    ```
    For a `load > 1`, open-addressing is not possible. Only chaining is
    possible
* Another bell and whistle is to adjust size of hash table.

## Hash functions

* Easy to make mistakes
  * For eg, taking 3 MSB numbers for telephones is a very bad choice.  Bad
    choices may expose patterns of numbers that aren't visible to naked eye.
  * Memory address are mostly always multiple of 4. So if the lsb 2  bits are
    used as is, 3/4th of hash-buckets will be unused!
  * If all data is multiple of N and hash-bucket-number is a multiple of N, we
    may have unfilled buckets.
* For a simple modulus like hash-function, choosing number of buckets(n) to be
  a prime closest to our desired N range.
    * The prime shouldn't be too close to a power of 2 or power of 10


### A sample hash function for a generic string

Taken from http://stackoverflow.com/questions/2624192/good-hash-function-for-strings[a stack overflow answer]

* Note the choice of 2 primes.
* First prime (7) adds some init-value. The second one is the multiplication factor.
* We multiply second-prime with previous result so far and then add the current char.

```
int hash = 7;
for (int i = 0; i < strlen; i++) {
   hash = hash*31 + charAt(i);
}
```

### A sample hash function for integers

Taken from http://stackoverflow.com/questions/664014/what-integer-hash-function-are-good-that-accepts-an-integer-hash-key[a stack overflow answer]

* Knuth's multiplicative method
    ```
    hash(i)=i*2654435761 mod 2^32
    ```
* In general, you should pick a multiplier that is in the order of your hash
  size (2^32^ in the example) and has no common factors with it
* The biggest *disadvantage* of this hash function is that it preserves
  divisibility, so if your integers are all divisible by 2 or by 4 (which is
  uncommon), their hashes will be too. This is a problem in hash tables:
  you can end up with only 1/2 or 1/4 of the buckets being used.
    * (my own?)This can be avoided with a modulo that is a prime.
* Another one:
    ```
    unsigned int hash(unsigned int x) {
      x = ((x >> 16) ^ x) * 0x45d9f3b;
      x = ((x >> 16) ^ x) * 0x45d9f3b;
      x = (x >> 16) ^ x;
      return x;
    }
    ```

## Consistent Hashing

This is the case where the number of buckets is dynamic.

http://www.tom-e-white.com/2007/11/consistent-hashing.html[Good link]

When the buckets are dynamic, tradition mod%N can't be employed as N is
changing.

* We have the keys - which are hashed from 0 to K. This range is mapped on to a
  circle
* We have the nodes(buckets). Now based on another hashing strategy each node
  is also randomly assigned points in the circle.
* Now when we have a key, we go to the circle and pick the node that appears
  next in the clock-wise direction.

Thus when a node leaves, since its present in multiple  points in the circle,
all these points disappear, and the load is distributed to rest of nodes as
the next successor for each of its previous point
Idea is that each hashed value lies in a circle. And select points in the
circle are buckets points. Each hash gets put in the bucket next in clock-wise
direction.

## Universal Hashing

This is a case where the hash functions are also many and we pick our hash
function itself from the family of functions and then hash!

Not sure I understand this. Need some simple explanation from somewhere.


# Graphs

* Represents pair-wise relationship among objects
* Terminology
  * Vertices or nodes
  * Edges
    * Careful. Don't confuse edge (having smaller spelling for a vertex, having
      a bigger spelling!)
* Directed or Undirected edges
  * In directed, first is tail and second is head. That is direction is
    from tail to head.
* Parallel edges are those that have same origin and destination vertex.
  This may or may not be meaningful to a given problem
* Typically m is number of edges, and n is number of vertices.
  (Mnemonic: m>>n, we have far more edges than there are vertices.
  Number of lines in m(3) is more than n(2)!
* A degree of a vertex, is referred as the number of edges that start out from
  that vertex.

## Cuts in a graph

* A Cut in a graph is a split of vertices into 2 non-empty groups(A and B).
* For undirected, Crossing edges are those that have one end-point each in A and B
* For Directed, Crossing edges are those that have tail in A, head in B


## Directed Graph

* Strongly connected:
  * If you can reach any vertex from any other vertex, following the edges in
    their given direction.
* Weakly connected:
  * If you can reach any vertex from any other vertex, following the
    edges in any direction (i.e assuming the graph as undirected
    check if u can reach all vertices)

## Numerical Facts

* If there are n vertices, assuming no parallel edges, there should be
  minimum of n-1 edges to have all the vertices connected (in one line) and
  utmost `n-C-2 = n(n-1)/2` number of edges (where all edge is connected to the
  other edge) in a undirected graph
* If there are n vertices in a graph, we can have up `2**n-2` possible cuts for
  this graph. Each vertex can be in either set A or B independent of the choice
  of other vertices. We just subtract 2 as we can't have all vertices in
  each set.

## Graph Representations

### Adjacency Matrix

* We have NxN matrix (vertices x vertices matrix).
* Each non-primary diagonal represents a possible Edge. Its 0/1 based on if
  that edge exists.
* Add bells and whistles to what the matrix element is to accommodate directed
  (+ve/-ve), parallel edges, weighted edges
* Super waste of memory for a sparse graph.

### Adjacency List

#### Algo course style

* Have 2 different lists - one for vertices and one for edges
* They cross reference each other.
  * Each edge points to its 2 vertices. This way edge struct is of fixed size.
  * Each vertex points to all edges incident on it. The vertex thus should
    have a list of edge-pointers.
* Note that the sum of cross-references from edges to vertices is exactly
  same as vertices to edges. The edges to vertices are exactly 2 per edge,
  while in vertices to edges, it varies on degree of each vertex.

#### Skiena book style

* Kind of a 2D linked list.
* We have a linked-list of vertices.
* For each vertex, we have a list of edges that originate from that vertex.
* For undirected, the edge appears twice, once is each vertex's list. For
  directed graph, it appears in the vertex which is its tail.
* Here we show the list of vertices as an array and the vertices as linked-list.

```
#define MAXV 1000 /* maximum number of vertices */

typedef struct {
  int y;                 /* adjacency info */
  int weight;            /* edge weight, if any */
  struct edgenode *next; /* next edge in list */
} edgenode;

typedef struct {
  edgenode *edges[MAXV+1]; /* adjacency info */
  int degree[MAXV+1];      /* outdegree of each vertex */
  int nvertices;           /* number of vertices in graph */
  int nedges;              /* number of edges in graph */
  bool directed;           /* is the graph directed? */
} graph;
```

The Skienna book style and algo-course styles are kind of same. In both ways, u
can walk over vertices and then for each vertex walk over its edges. Just that
the algo-course suggests to keep the actual vertices and edges separately in
lists of their own.

### Comparision

|Comparison                             | Winner
|---------------------------            | ----------
|Faster to test if (x,y) is in graph?   | adjacency matrices
|Faster to find the degree of a vertex? | adjacency lists
|Less memory on small graphs?           | adjacency lists (m + n) vs. (n2)
|Less memory on big graphs?             | adjacency matrices (a small win)
|Edge insertion or deletion?            | adjacency matrices O(1) vs. O(d)
|Faster to traverse the graph?          | adjacency lists Θ(m + n) vs. Θ(n2)
|Better for most problems?              | adjacency lists

## Graph classifications

* Directed, Undirected
* Sparse, Dense
  * Sparse has edge-number closer to the linear bound (n-1), while dense
    matrix is where edge-number is closer to upper bound ~n~C~2~

## Graph Traversal

* Before traversal, we can mark each node as one of the 3 states
  * Undiscovered
  * Discovered
  * Processed
* Most search graph algorithms consider one vertex as the source/start vertex.

### Breadth First Traversal

* Note the presence of a Queue in BFS
* It grabs territory layer by layer from source vertex.

* Skienna code
```
BFS(G,s)
  for each vertex u ∈ V [G] − {s} do                    /* Each node has 3 states - "undiscovered", "discovered", "processed" */
    state[u] = "undiscovered"                           /*     Depending on ur alog, u may/may-not need the processed state */
    p[u] = nil, i.e. no parent is in the BFS tree
  state[s] = "discovered"
  p[s] = nil
  Q = {s}
  while Q ≠ ϕ do
    u = dequeue[Q]                   /* Note that u is already discovered *
    process vertex u as desired       * However processing of u happens now */
    for each v ∈ Adj[u] do
      process edge (u,v) as desired  /* For undirected graph, this edge may be already processed
                                        So, if u want only one time, track that as well */
      if state[v] = "undiscovered" then
        state[v] = "discovered"
        p[v] = u                    /* This parent path from v to source s, is the shortest path from s to v
                                       for undirected graphs. For directed graphs, there may be back-pointing
                                        edges! */

        enqueue[Q,v]
    state[u] = "processed"
```
* Interview-Bit code
```
public void bfs(Node *rootNode) {
    Queue q = new LinkedList();
    q.add(rootNode);
    processNode(rootNode);
    rootNode->visited = true;
    while(!q.isEmpty()) {
        Node *n = q.remove();
        Node *child = null;
        while((child = getUnvisitedChildNode(n)) != null) {   <-- This basically iterates every node of n and returns unvisited nodes
            child.visited = true;
            processNode(child);
            q.add(child);
        }
    }
    //Clear visited property of nodes
    clearNodes();
}
```

* Whatever is marked processed at the end of BFS is what is reacheable from s.

#### Applications

* Find all connected nodes to a given graph.
  * You can keep all nodes in a bigger outer queue.
  * Start with one node, and do a BFS from here. You will pick all nodes that
    is connected with this. Dequeue from outer queue as you meet nodes.
  * Keep doing BFS, till outer queue is empty. That way you get all groups in
    the graph.
  * This could be done DFS way also (my own observation!)
* Find distance of each vertex from source vertex s.
  * This is easily achieved by storing the distance in each vertex - as part of
    processing of node.
* Find the path to each vertex from source vertex s.
  * If we build the parent of each node info in our BFS, we can use recursive
    approach to build the root from the parent info.
    ```
    parents_arr=[...]
    find_parent(parents_arr, parent_desired, node)
    {
      if ( node == parent || parents_arr[node] == -1) {
        printf("%d",parent_desired);
      } else {
        find_parent(parents_arr, parent_desired, parents_arr[node]);
        printf("%d",node);
      }
    }
    ```
* Find if a graphs is bipartite (can you assign vertices either of 2 colors,
  such that no two adjacent vertex is of same color) [ Also note: bipartite
  graphs will always have a cut, that has all edges traversing the cut! ]
  * Keep running BFS and see if you can successfully finish BFS.
  * In each iteration, color RED/BLACK alternatively. If you successfully finish
    this coloring, u have a bipartite graph.

### Depth First Traversal

* It just goes all in into one path
* We can technically modify BFS slightly by replacing Queue with stack, or
  leverage recursion to naturally achieve our stacking.
```
time = 0          /* is a global var */    |
DFS(G,u)                                   |    DFS(G,u)
  state[u] = "discovered"                  |     state[u] = "discovered"
  process vertex u if desired              |        ... early_process_vertex ...
  entry[u] = time                          |        ...
  time = time + 1                          |        ...
  for each v ∈ Adj[u] do                   |     for each v ∈ Adj[u] do
    process edge (u,v) if desired          |        ... process_edge (u,v)
    if state[v] = "undiscovered" then      |        if state[v] = "Undiscovered" then
      p[v] = u                             |          ...
      DFS(G,v)                             |          DFS(G,v)
  state[u] = "processed"                   |
  exit[u] = time                           |
  time = time + 1                          |
```
* Interview-Bit code
    ```
    public void dfs(Node *rootNode) {
        //DFS uses Stack data structure
        Stack s = new Stack();
        s.push(rootNode);
        rootNode.visited = true;
        processNode(rootNode);
        while(!s.isEmpty()) {
            Node n = (Node)s.peek();
            Node child = getUnvisitedChildNode(n);  // Essentially this function goes through the neighbors of n, and finds the one with node.visited = false
            if(child != null) {
                child.visited = true;
                processNode(child); // print the node as you like.
                s.push(child);
            }
            else {
                s.pop();              /* No child left. So done with n */
            }
        }
        //Clear visited property of nodes
        clearNodes();
    }
    ```
* The notion of parent is looking superfluous to me.
* The time spent at a parent will be a superset of time
  spent at children. Not sure how otherwise time is useful.

#### Applications

* Find cycles
  * DFS by its nature, can classify a edge into tree edges and back
    edges. Edge that explores new vertex is a tree edge, while an
    edge that goes back into a ancestor is a back-edge
  * A acyclic graph is one, that has no back-edge during a traversal
* Topological sort for directed graph
  * Topological sort is the ordering of what should be completed before
    another.
  * Topological order is possible only if there are no cycles. Further
    this implies that acyclic graphs should have at least one sink vertex.
    * A sink vertex is a vertex that has no outgoing edges. If there
      are no sink vertices in a graph, then it definitely cyclic.
    * You can start DFS at any vertex. You will end up walking all nodes
      from this. But some vertices may have higher precedence(before)
      than the chosen vertex.
      * There are 2 solutions.
        * First
          * Start with a sink vertex(you need to find this some how). Give
            this the value N (no of vertices in graph). This has to be done
            last by definition.
          * Now remove that node from Graph. Repeat the algorithm. (Due to
            loss of this node, there will be new sink vertex(ices) created
        * Second / slick way.
          * This is same as DFS algo. Just have a current-label extra with
            initial value as N, number of edges
          * Start DFS from any node. Once you hit the depth of recursion,
            which is end of for-loop, assign the current-label value to
            that node.
          * This will neatly assign the Nth value for the first sink you
            discover and back-track from there and so on.
    ```
    topological_sort(G)
      all vertices = "Undiscovered"
      current_label = N  /* number of vertices */
      for each vertex u
        if u is "Undiscovered":
          DFS(G,u)

    DFS(G,u)
     state[u] = "discovered"
     for each v ∈ Adj[u] do
       if state[v] = "Undiscovered" then
          DFS(G,v)
       level-of-u = current_label
       current_label--
    ```
* Articulate edge detection
  * Given in skienna.
  * I didn't follow it

## General Algorithms in a Graph

### Minimum cuts

Given a graph, find the cut that has the minimum number of cross-over edge
(Min-cut) This is useful, to find closely related vertices in a graph.

#### Karger Algorithm

The solution allows parallel edges for this graph. This goes as follows:

* Keep proceeding till the node-count reduces to 2.
* In every iteration, *randomly* pick an edge and collapse the 2 vertices that
  it connects into one fused-super-vertex. Remove this chosen edge.
* Remove any edges that start and end at same-edge.
* When you are left with 2, all vertices part of each fused/orig vertex is
  the resulting graph-cut.

But this is just a random algo. There is no guarantee that the resulting cut
is a min-cut.

##### Analysis of this algorithm

* If a edge that should remain as part of min-cut, ends up getting randomly
  chosen, then the algo will fail.
* But if run a few times, this algorithm will succeed with a high degree of
  probability.

### Prim's Algorithm

* Spanning Tree
  * A spanning tree is a connected graph using all vertices in which there are
    no circuits. In other words, there is a path from any vertex to any other
    vertex, but no circuits.
* Minium Spanning Tree
  * A minimum spanning tree is a spanning tree in which the sum of the weight
    of the edges is as minimum as possible

Find the minimum spanning tree.

* Pick any node.
* Pick the least weighted edge going out. Consume this node.
* Now among all outbound edges from the current set of picked notes,
  and pick the least weighted node.
* Keep repeating till the whole graph is covered.

```
Initialize X = {s} [s ∈ V chosen arbitrarily]
- T = ∅ ;          [invariant: X = vertices spanned by tree-so-far T]
- While X ≠ V
  - Let e = (u,v) be the cheapest edge of G with u ∈ X, v ∉ X.
  - Add e to T
  - Add v to X.
```


Order of running:

* Assuming the edges are all maintained in a heap, we have O(|E|log(|V|))
* Use a min-heap to pick the next edge.
* Pick any first node. Now sort the other edges that reach from this vector
  and build the heap. Nodes that aren't directly reachable are value inf.
* Pick the cheapest edge. The thing is once you pick, the vertices values
  change as they are now even more reachable.
* Each heap operation is log(vertex) sized. There will be utmost Edge
  number of operations. Because at each iteration, we adjust those
  vertices, there are reachable from the newly added vertex
* so, its O(|E|log(|V|)). There is a heapify up front, but that's O(|V|).
  So we can ignore that.

### Kruskal's Algorithm

Find the minimum spanning tree

* pick the cheapest edge as long as its not creating cycles, i.e it shouldn't
  connect 2 vertices that are already connected

```
sort edges in order of increasing cost
T = NULL
for i = 1 to m
  if T,[i] is not creating cycle
    add i to T
return T
```

Order of running:

* Sort of edges takes O(mlogm) == O(mlogn), m-no of edges > n-no of vertices.
  But we can utmost m=n-power-2 number of edges, so, O(mlogm) = O(mlogn)
* Now, there are O(m) iterations, and each iteration takes O(1) to find if edge
  i is going to create a cycle or not.
* There will be utmost log-n leader updates and each update is O(n). So there will
  be upto O(nlogn)  updates)

So, total = O(mlogn) + O(m) + O(nlogn) = O(mlogn), which is same as Prim

* Kruskal's algorithm'ish can also be used to do a clustering algo.
  Clustering algo is to group n vertices into k clusters/groups so
  that the closest are together.
* Just as how kruskal tries to fuse vertices to build a spanning tree, we
  will fuse vertices till the cluster count comes down to k!


### Single Source Shortest Path - Dijkstra Algorithm

* Find the shorter path to all vertices for a given vertex.
* Graph should not have negative edges.

```
function Dijkstra(Graph, source):

     for each vertex v in Graph.Vertices:
         dist[v] ← INFINITY
         prev[v] ← UNDEFINED
         add v to Q
     dist[source] ← 0

     while Q is not empty:
         u ← vertex in Q with min dist[u]
         remove u from Q

         for each neighbor v of u still in Q:
             alt ← dist[u] + Graph.Edges(u, v)
             if alt < dist[v]:
                 dist[v] ← alt
                 prev[v] ← u

     return dist[], prev[]
```

Progression of above
* Initially set all vertices as unreachable(∞) except for source(0).
* So, obviously in the first iteration of the while loop, the source node
  will be u.
  * In every iteration of while loop, it will be the next shortest reachable
    vertex. Its distance has alrady been computed in previous iterations.
  * So, if your goal is to just find distance to a specifc target, you can
    stop once u is the target vertex.
* Now, from this new vertex as source, update the distances to all neighbout
  vertexes. If they are smaller than what was previously noted, you can update
  it here.

* Dijkstra's algo computes Single-Source-Shortest-Path in O(m log-n),
  but it will only compute it for non-negative edge weights

### Single Source Shortest Path - Bellman Ford Algorithm

* Work when there are negative edges.
* BUt note, there can't be negative cycles - as then we can keep cycling
  in that loop forever and reduce costs.

* Note that for any shortest path w/o -ve cycles, there will be utmost
  (n-1) edges for any 2 vertices in that graph.

Algorithm

```
function BellmanFord(list vertices, list edges, vertex source) is

    // This implementation takes in a graph, represented as
    // lists of vertices (represented as integers [0..n-1]) and edges,
    // and fills two arrays (distance and predecessor) holding
    // the shortest path from the source to each vertex

    distance := list of size n
    predecessor := list of size n

    // Step 1: initialize graph
    for each vertex v in vertices do

        distance[v] := inf             // Initialize the distance to all vertices to infinity
        predecessor[v] := null         // And having a null predecessor

    distance[source] := 0              // The distance from the source to itself is, of course, zero

    // Step 2: relax edges repeatedly
    repeat |V|−1 times:
        for each edge (u, v) with weight w in edges do
            if distance[u] + w < distance[v] then
                distance[v] := distance[u] + w
                predecessor[v] := u

    // Step 3: check for negative-weight cycles
    for each edge (u, v) with weight w in edges do
        if distance[u] + w < distance[v] then
            predecessor[v] := u
            // A negative cycle exist;
            // The edge (u, v) must be reachable from a negative cycle, but
            // it isn't necessarily part of the cycle itself
            // So find a vertex on the cycle - this is simply looping
            // back on predecessor till we find a predecessor that is already
            // visited, and start from there.
            visited := list of size n initialized with false
            visited[v] := true
            while not visited[u] do
                visited[u] := true
                u := predecessor[u]
            // u is a vertex in a negative cycle, find the cycle itself
            ncycle := [u]
            v := predecessor[u]
            while v != u do
                ncycle := concatenate([v], ncycle)
                v := predecessor[v]
            error "Graph contains a negative-weight cycle", ncycle
    return distance, predecessor

```

#### Historic notes

Not sure i follow fully:

Building optimal substructure
* We build up on the number of edges in our solution.
* At a given iteration, we update the shortest source of all edges
  based on 1+in-degree of that vertex.
  * Either the old solution itself is better.
  * Or one of the in degree is better.

Running time:
* n iterations (1 for each edge addition from none to max(n-1))
* In each iteration, we have upto `n * (in-degree)` lookups. This
  `n * (in-degree-of-each-vertex)` is essentially the number of
  edges in that graph. Thus each iteration we do work totalling
  number of edges.

#### Running complexity

So final running time is `O(nm)`

#### Space optimization

* By defn, we need O(n-square) space.
* But note that we can discard a prev iteration, after we update all
  vertices. So we need only O(n) space.
* Also you can store predecessor points to reconstruct the actual path.
* To find a -ve cycle, use DFS to find the predecessor path has a cycle.
  This cycle will be a -ve cost cycle.

#### Applications

Internet routing is more-or-less bellman-ford in action, as each router
updates its route-info from its previous hop. (Distance vector protocol -
like RIP that just worry about the distance cost). But this has flaw like
the counting to infinity, where we update a route, in which we participate
ourself, but has failed. BGP - path-vector protocol builds up using full
paths (?) is resilient to this.

### All Pair Shortest Path(APSP)

* Run Single-Source-Shortest-Path n times
  * If Dijkstra, then for sparse graph `{O(m) =~ O(n)}`, for dense graphs
    {O(m) =~ O(n-square)}, APSP running time is O(n-square.log-n) and
    O(n-cube.log-n) respectively.
  * If Bellmanford, then, APSP running time is `O(n-cube)` and `O(n-power-4)`
    respectively!
* Note that we should also report if there exists atleast 1 -ve cycle in the
  whole graph

### Floyd-Warshall Algorithm

* For All Pair shortest path.
* Works for even -ve edges.
* Runs in O(n-cube) .. thus better than Dijkstra is 2 cases,
  Competitive in 1 case.

Subproblem structure:

* Name vertices arbitrarily. Each iteration, you start by adding one vertex in consideration.
  k = 1..n is the outer iteration.
* Each iteration, goes over all n-C-2 pairs updating their shortest path.
* However, at each iteration, you can only use vertices from 1 to k (the outer interation).
Now, the sub-structure has 2 cases
* the new k is not part of the Shortest path between the 2 nodes under consideration
* the new k is part of the shortest path.
To evaluate both, we just need to know u-to-k,k-to-w, both sub-problems definitely already
known from previous iterations.

Algo:

Let A = 3-D array (indexed by i, j, k)
Base cases: For all i, j ∈ V :
            +-- 0   if i = j
A[i,j,0] =  |   cij if (i,j) ∈ E
            +-- +∞  otherwise
For k = 1 to n
  For i = 1 to n
    For j = 1 to n
      A[i, j, k] = min ( A[i, j, k-1]                   # k not part
                         A[i, k, k-1] + A[k, j, k-1] )  # k part.

If graph has -ve cycles, then our algo will report some
A[i,i] path (i.e the diagnal of final result) is -ve, instead of 0.

To reconstruct, have another B[i,j] which stores the last updated k of case-2.
We can then use this to reconstruct a shortest path.



# Design Paradigm

* Divide & Conquer
* Randomized Algorithms
* Greedy Algorithms
* Dynamic Programming

# Greedy Algorithms

Definition: Iteratively make "myopic" decisions, hope everything works out
at the end.

* Easy to propose
* Easy run-time analysis
* Hard to establish Correctness
* Danger: Most times, can be wrong.

To Establish proof:

* Use Induction
* Exchange Argument.
* Whatever works

Optimal-caching: Didn't really follow why this came up.
* Furthest in future
* Is clairvoyant.
* Useful as a guide to analyze other algo?
* Serves as a benchmark for caching algo.

### Scheduling Problem

Problem:

* Each job has length(li) and weight(wi).
* Completion time (Ci) is the wait-time and length. Thus first job has C1 = l1,
  while C2 = (l1+l2) etc..
* Minimize Sum Σ Ciwi across all jobs

greedy soln:

* Sort each item by wi/li.
  * More the weight it comes first, lesser the time time, it comes first.
* Schedule in that order.


## Union-Find Data Structure

* O(1) to tell if a given node is in a group
* O(nlogn) to work for n unions
  * Note that there will utmost log-n leader updates and each update is O(n)
    update

### Lazy unions

* Find: O(logn)  [ Only if we unionize by the rank ]
* Union: O(1) + 2*Find

Rank: Maximum length of the root->parent path. When fusing, choose the tree
      with the higher rank as the new parent.

Rank lemma: (Controls the population of a given rank)
For an arbitrary size, the utmost n/(2-power-r) objects with rank r.

### Path Compression

* Every find re-writes its path, so that over time, a previous find
  will give us O(1) in future

Hopcroft-Ullman Theorm: If you do path-compression, then a sequence
of m operations(union and find) will cost O(mlog*n) time.

Tarjan's Bound: *Same as above*, will cost O(m-alpha(n)) time.
alpha(n) is inverse Ackermann function

## Weight-Independent Set in Path Graph

* path graph is a liner graph
* Given a graph with weights associated to vertices, find the
  subset of vertices that are not adjacent and the sum of weights
  is maximum

* O(n) time. A[0] = 0 , A[1] = 1
             A[i] = max{ A[i-2] + Vi , A[i-1] }

# Dynamic Programming

* Identify a small number of sub-problems
* Correctly solve the bigger problem given the solutions of the sub-problem
* Ensure, the final solution is available by solving the sub-problems

* Memoization/overlapping subproblems: This will be very frequent in dynamic
  programming as you are frequently recursing into smaller problems

## Knapsack Problem

Problem:

* We have a knapsack with integral weight W.
* We have n items, each with value vi and weight wi.
* We need to maximize our value, but at same time not exceed weight of
  knapsack.

Algo:

```
Let A=2D-array   # x-axis represents item-0..n, y-axis represents weight 1..W
# set first column to all 0 (0th item doesn't exist)
Initialize A[0,x] = 0 for all x=0,1...W
For i = 1...n
  For x = 0..W
    A[i,x] = max{A[i-1,x], A[i-1,x-wi] + vi}  # first one drop this item, second one picks this item
Return A[n,W]
```

## Sequence Alignment Problem

Problem:
Strings X = x1 ... xm, Y = y1 ... yn over some alphabet Σ (like {A,C,G,T})
  - Penalty αgap for inserting a gap,
  - αab for matching a & b [presumably αab = 0 of a = b]

Find the alignment with minimum possible penalty.

Solution:

```
Notation: Pij = penalty of optimal alignment of Xi & Yj.
Recurrence:
For all i = 1 ... m and j = 1 ... n:
          Pij = min +-- (1) αxiyj  + Pi−1,j−1       #Edit the last char
                    |   (2) αgap   + Pi−1,j         #insert a char in Xi
                    +-- (3) αgap   + Pi,j−1         #insert a char in Yj
```


Algorithm:

```
A = 2-D array.
A[i;0] = A[0; i] = i · αgap; ∀ i ≥ 0
For i = 1 to m
   For j = 1 to n
      A[i; j] = min +-- (1) A[i-1,j-1] + αxi yj
                    |   (2) A[i-1,j] + αgap
                    +-- (3) A[i,j-1] + αgap
```


## Optimal Binary Tree

Problem in English:
We have to construct a binary tree, but we are also given a bunch of weights
against every item.  We need to construct most optimal tree, so that search
costs are minimal.

Formal Statement:

```
Input: Frequencies p1, p2, ..., pn for items 1, 2, ..., n.
       [Assume items in sorted order, 1 < 2 < ... < n]
Goal: Compute a valid search tree that minimizes the weighted (average) search time.
       C(T) = Σ (over all items i)  pi * [search time for i in T]   #The search time is the depth of i in T.
```

Notes:
* We can't work like huffman as we need to preserve order. (huh?)
* We can't greedily pick the top occurring as root - as that may be a
  sub-optimal choice. Consider 1(0.01), 2(0.34), 3(0.33), 4(0.32). Here 3
  should be root, although 2 is indeed the highest occurring frequency.

Solution:

* The trick here is to pick the root. Assume if root is correct, then Left and
  right sides have to be optimal binary trees themselves. Thus we need to build
  optimal binary trees for every contiguous subset in the given input.
* And we evaluate the cost of picking every element as root.

```
                  j                 j
optcost(i,j) =  Σ      freq[k] + min    [optcost(i,r-1) + optcost(r+1,j)]
                 k=i                r=i

Final = optcost(0,n-1)
```
* Why is `Σ freq[k]` and not just `freq[k]`. Because the optcost(i,r-1) for
  example gives the cost assuming the tree is rooted there, but then its only
  a subtree. So before going there you have to spend a time at this root. So
  essentially this means adding every frequency.
* Initially it might look like its a lot of work - but with memoization, it
  can really speed up.

Not clear what??

Algorithm:
* Solve smallest subproblems (with fewest number (j − i + 1) of items) first.

```
Let A = 2-D array. {A[i,j] represents opt BST value of items {i...j}
For s = 0 to n − 1 [s represents j − i]
  For i = 1 to n   [so i + s plays role of j]
     A[i, i + s] =     min          { Σ (k over i to i+s)  pk + A(i,r−1) + C(r+1,s) }
                   (r over i to i+s)
Return A[1,n]
                       Interpret as 0 if 1st index > 2nd index. Available for O(1)-time lookup
```

Running time
* O(n-square) number of sub-problems
* O(j-i) time to compute A[i,j]
* O(n-cubed) overall running time.



## Johnson's Algorithm

* Runs one iteration of Bellman-Ford and n iterations of Dijkstra.
* The idea is to run the Bellman-Ford and covert all edge weights to +ve.
* Note that we can't just like that add a positive weight, as that will
  displace paths with unequal number of edges in them.

Idea to reweight edges:
* Reweighting using vertex weights {Pv} adds the same amount (namely, ps - pt)
  to every s-t path
* Reweighting always leaves the shortest path unchanged
* Image a new vertex and a path from here to every other vertex of value 0.
  These edges are invisible to the original Graph.
* Run Bellman-Ford on this new graph. The shortest path from S to every node
  is now the weight of that n


Find out weights to every vertex
Re-adjust cost of every edge from:
 newL = L + Ps - Pt

Question in Quiz: (To see discussion forum)

# Polynomial Time Solvable Problem

Definition: A problem is polynomial-time solvable if there is an
algorithm that correctly solves it in O(n-power-k) time, for some constant
k.
Where n = input length]
Yes, even k = 10,000 is sufficient for this definition
Comment: Will focus on deterministic algorithms, but to first order doesn’t matter.

Class-P is the set of polynomial time solvable problems.

Examples of NP:
* Detect shortest path in graphs, with negative cycles, that don't form cycles.
* Knapsack, where n is proportional to O(log-W) [Weights are way too high]
* Travelling Salesman problem

## Reduction

[A little informal] Problem Π1 reduces to problem Π2 if: given a
polynomial-time subroutine for Π2, can use it to solve Π1 in polynomial time.

## Completeness

Let C be a set of problems. The problem PI is C-Complete, if
* PI ∈ C, and
* everything in C, reduces to PI
(That is: PI is the hardest problem of C)

Note:
TSP is not as hard as halting problem. But TSP is as hard as any brute-force
only solvable problems.

# Class NP
(Non Deterministic Polynomial)

A problem is in NP if:
(1) Solutions always have length polynomial in the input size
(2) Purported solutions can be verified in polynomial time

## Algorithmic Approaches

* Focus on special cases
  * Knapsack where W is polynomial (and not exponential) proportional to N
  * 2SAT (P) instead of 3SAT (NP)
  * Vertex cover when OPT is small.
* Heuristics
* Solve in exp time, but faster then brute force

### Vertex Cover Problem

Input: An undirected graph G = (V ; E).

Goal: Compute a minimum-cardinality vertex cover { a subset
S ⊆ V that contains at least one endpoint of each edge of G}

Comments in lecture: This problem was probably better named as Edge cover,
as actually, its all the edges the we are trying to get covered!

To get the picture, note that the vertex cover of a star-graph is just the
central vertex and the vertex cover of the clique (every vertex connected to
every other vertex) graph has a size n-1.

While vertex-cover is generally NP complete, the following special cases
are polynomial
* Tree (looks like dynamic programming)
* Bipartite graphs (classify vertices in 2 sets and no edge connects vertices
                    from same set.)
* When optimal soln itself is small (log(n) or less).

Say, there exists a soln of size k. To pick k vertices out of n vertices, it
takes n-C-k (and we try from 1...k, too), so this brute force solution itself
is O(n-power-k)

#### Substructure Lemma

Consider graph G, edge (u, v) ∈ G, integer k ≥ 1.

Let Gu = G with u and its incident edges deleted (similarly, Gv).
Then G has a vertex cover of size k ⇔ Gu or Gv (or both) has a vertex cover of size (k − 1)

In English: If you take one vertex out and all its incident edges, then the remaining
graph, will have a vertex cover of size k-1. Note that either side may hold good for
the arbitrarily chosen edge.

#### Algorithm to solve vertext cover

Given undirected graph G = (V , E), integer k
Ignore base cases
* Pick an arbitrary edge (u, v) ∈  E.
* Recursively search for a vertex cover S of size (k − 1) in Gu
    (G with u + its incident edges deleted).
  If found, return S ∪ u
* Recursively search for a vertex cover S of size (k − 1) in Gv.
  If found, return S ∪ v
* FAIL. [G has no vertex cover with size k]


#### Run time Analysis

* We have 2-power-k recursions. (Each iteration we reduce k by 1)
* We have O(m) work per iteration. (m = No of edges)

Running time is O(2-power-k*m). Since k is less =~ O(log-n), we
can say this is polynomial time.
Definitely way better than O(n-power-k)

### Travelling Salesman Problem

Input: A complete undirected graph with nonnegative edge costs.
Output: A minimum-cost tour (i.e., a cycle that visits every vertex exactly once)
        and returns to the start.

Brute force takes n! time
Dynamic programming can take O(n-power-2 * 2-power-n) time.

Remember Bellman-ford, we increase an edge in every interation.

* Directly using it doesn't solve the original problem. We can take an
  arbitrary vertex to start, add edges. In any iteration, we may pick
  a path that skips a vertex. So, in the end, we have a path that
  does not include all vertices.
* Even if we add a constraint that we need every iteration to have n
  vertices, we may end up picking cycles.
* If we add a constraint that we have n vertices in each iteration and
  there shouldn't be repititions, we are okay.

Thus we need to remember the actual path we used.

Algorithm:

```
Let A = 2-D array, indexed by subsets S ⊆ { 1,2,....,n } that contain 1
                   and destinations   j ∈ { 1,2,....,n }

Base case:

A[S, 1] = +-- 0 if S == 1,
          +-- ∞ otherwise  [no way to avoid visiting vertex (twice)]
```

While we are building a 2D array, while one dimension is the size of problem m,
the other index is the current-sub-set of S of size m. So the size of n-C-m

```
For m = 2, 3, ...n [m = subproblem size]
  For each set S ⊆ { 1, 2, ... , n} of size m that contains 1  <-- This is of size  n-C-m
    For each j ∈ S, j != 1
       A[S, j] = min k∈S,k!=j{A[S − {j}, k] + Ckj} [same as recurrence]
Return min(j=2,....n){ A[{1, 2, ...., n}, j] + Cj1 }
```

To understand that better:

```
For m = 2, 3, ...n [m = subproblem size]
  For each set S ⊆ { 1, 2, ... , n} of size m that contains 1  <-- This is of size  n-C-m
    For each j ∈ S, j != 1
       Let S* = S - {j}
       A[S, j] = min of all k∈S*{A[S*, k] + Ckj} [Kind of same as Bellman-Ford. Just that
                                                  this is an iteration on its own]
```
Finally, we should complete our tour back to 1. So, run the minium of all last-round results.
Return min(j=2,....n){ A[{1, 2, ...., n}, j] + Cj1 }

Note: The tour is the cheapeast cycle having all vertices. So, source-vertex can be anything.
We will have to complete the cycle anyway.

My worked out example to understand the iterations above.
```
Consider the following graph
        1
      / | \
    a/  |b \c
    /   |   \
   v  d v e  v
   2<---3--->4
   \    |   /
   f\  g|  /h
     \  | /
      v vv
        5
Outer Iteration -- size of problem. This is from 1 to 5.
First Inner Iteration -> Choosing m out of n-1
Most Inner Iteration -> Choosing n out previous solutions.

m = 1
S = {1}
Most inner is trival, as this is base-case.
Array at end of this iteration
 +-----+------+
 |  1  |  0   |
 +-----+------+
 |  2  |  ∞   |
 +-----+------+
 |  3  |  ∞   |
 +-----+------+
 |  4  |  ∞   |
 +-----+------+
 |  5  |  ∞   |
 +-----+------+

m = 2
S = [ {1,2} , {1,3} , {1,4} , {1,5} ]
Again inner is trival, as we just use up edges directly.
 +--------+------+
 |  1,2   |  a   |
 +--------+------+
 |  1,3   |  b   |
 +--------+------+
 |  1,4   |  c   |
 +--------+------+
 |  1,5   |  ∞   |
 +--------+------+

m=3
S= [ {1,2,3} , {1,2,4} , {1,2,5}, {1,3,4}, {1,3,5} , {1,4,5} ]
First inner will have 2 loops for every S. At this time, the most inner loop
   just copies the previous results as-is, as there is only 1 subproblem
   to take min-of.
 +----------+------+--------+
 |  1,2,3   |  2   |  b+d   |
 +          +------+--------+
 |          |  3   |   ∞    | # there is no 2->3
 +----------+------+--------+
 |  1,2,4   |  2   |   ∞    | # there is no 4->2
 +          +------+--------+
 |          |  4   |   ∞    | # there is no 2->4
 +----------+------+--------+
 |  1,2,5   |  2   |   ∞    | # {1,5} from prev table is ∞
 +          +------+--------+
 |          |  5   |  a+f   |
 +----------+------+--------+
 |  1,3,4   |  3   |   ∞    | # there is no 4->3
 +          +------+--------+
 |          |  4   |  b+e   |
 +----------+------+--------+
 |  1,3,5   |  3   |   ∞    | # {1,5} from prev table is ∞
 +          +------+--------+
 |          |  5   |  b+g   |
 +----------+------+--------+
 |  1,4,5   |  4   |   ∞    | # {1,5} from prev table is ∞
 +          +------+--------+
 |          |  5   |  c+h   |
 +----------+------+--------+
m=4
S= [ {1,2,3,4} , {1,2,3,5} , {1,2,4,5}, {1,3,4,5}]
First inner will have 3 loops for every S. At this time, the inner loop
   does a min of prev 2 options.
 +-------------+------+----------------------+------+
 |  1,2,3,4    |  2   |{1,3,4} has 2 options |  ∞   | ending-3 itself is ∞, and 4->2 is ∞
 +             +------+----------------------+------+
 |             |  3   |{1,2,4}               |  ∞   | both prev problems are ∞
 +             +------+----------------------+------+
 |             |  4   |{1,2,3}               |  ∞   | no 2->4, and ending 3 is ∞
 +-------------+------+----------------------+------+
 |  1,2,3,5    |  2   |{1,3,5}               |  ∞   | ending-3 itself is ∞, and 5->2 is ∞
 +             +------+----------------------+------+
 |             |  3   |{1,2,5}               |  ∞   | ending-2 itself is ∞, and 5->3 is ∞
 +             +------+----------------------+------+
 |             |  5   |{1,2,3}               |b+d+f | ending-3 itself is ∞
 +-------------+------+----------------------+------+
 |  1,2,4,5    |  2   |{1,4,5}               |  ∞   | ending-4 itself is ∞, and 5->2 is ∞
 +             +------+----------------------+------+
 |             |  4   |{1,2,5}               |  ∞   | ending-2 itself is ∞, and 5->4 is ∞
 +             +------+----------------------+------+
 |             |  5   |{1,2,4}               |  ∞   | both prev problems are ∞
 +-------------+------+----------------------+------+
 |  1,3,4,5    |  3   |{1,4,5}               |  ∞   | ending-4 itself is ∞, and 5->3 is ∞
 +             +------+----------------------+------+
 |             |  4   |{1,3,5}               |  ∞   | ending-3 itself is ∞, and 5->4 is ∞
 +             +------+----------------------+------+
 |             |  5   |{1,3,4}               |b+e+h | ending-3 itself is ∞
 +-------------+------+----------------------+------+
m=5
S= [ {1,2,3,4,5} ]
First inner will have 4 loops for every S. At this time, the inner loop
   does a min of prev 3 options.
 +-------------+------+----------------------+------+
 |  1,2,3,4,5  |  2   |{1,3,4,5}             |  ∞   | Only ending 5 is non-∞, and 5->2 is ∞
 +             +------+----------------------+------+
 |             |  3   |{1,2,4,5}             |  ∞   | all prev problems are ∞
 +             +------+----------------------+------+
 |             |  4   |{1,2,3,5}             |  ∞   | Only ending 5 is non-∞, and 5->4 is ∞
 +             +------+----------------------+------+
 |             |  5   |{1,2,3,4}             |  ∞   | all prev problems are ∞
 +-------------+------+----------------------+------+
```

#### Running time:

Outerloop - n times
First inner loop - n-C-m times = 2-power-n
Inner loop - n times (There can by utmost n prev sub-problems)

Total - O(n*2-power-n*n) = O(n-power-2 * 2-power-n)

#### Knapsack with greedy hueristic

Note that if we try to pick items sorted on Vi/Wi, we wont be picking the best.
So, we can either pick items like this or pick the item with the most highest Value.

It turns out that the last strategy is always alteast 50% of best solution!
