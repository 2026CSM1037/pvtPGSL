Edge cases to defensively handle no matter the topic (this is the recurring "trap" across every spec you've shown me):
isolated vertices / empty adjacency lists
self-loops (count once, don't double for undirected)
parallel edges (preserved, not deduplicated, unless spec says otherwise)
disconnected graphs
zero and negative weights (where valid)
dangling/degree-0 vertices (PageRank-style — redistribute rank)
V not divisible by block size (blocking tasks)
empty clusters (K-Means — keep old centroid)


## Gradient Descent
**The problem:** you have some function (a curve) and you want to find its lowest point — the minimum.

**The idea:** imagine you're standing on a hilly curve, blindfolded, and you can only feel which way is "downhill" right where you're standing (that's the *slope*, or derivative). So you take a small step downhill, feel again, take another small step, and repeat. Eventually you settle at the bottom.

- The "slope" is the derivative `f'(x)`.
- `learning_rate` = how big a step you take each time. Too big and you overshoot and bounce around; too small and it takes forever.
- You stop when the slope is nearly flat (`|f'(x)| ≤ tolerance`) — meaning you've basically reached the bottom.

That's it. In your assignment it's just a polynomial like `f(x) = x² - 6x + 9`, and you're finding the `x` that minimizes it. This exact idea (walk downhill repeatedly) is literally how neural networks are trained, just in thousands of dimensions instead of one.

## Maxflow–Mincut
**The problem:** think of a network of pipes carrying water from a source (s) to a sink/destination (t). Each pipe has a maximum capacity. What's the *most* water you can push from s to t?

- **Max-flow**: the largest total amount of water you can send, respecting every pipe's capacity limit.
- **Min-cut**: if you had to "cut" a set of pipes to completely disconnect s from t, what's the cheapest set of pipes to cut (by total capacity)?

The beautiful (and non-obvious) fact this assignment is built around: **max-flow always equals min-cut**. They're the same number, viewed two different ways — how much you *can* push through, vs. the smallest bottleneck that's *stopping* you from pushing more.

**How you compute it (Dinic's / Ford-Fulkerson style):** keep finding a path from s to t that still has spare capacity, push as much flow as possible along it, subtract that from the pipes used (and add a "reverse edge" so the algorithm can undo a bad earlier choice later). Repeat until no more augmenting paths exist. Whatever's left unreachable from s in this "residual" network — that boundary — is your min-cut.

Real-world flavor: max data through a network, max cars through road capacity limits, max goods through a supply chain.

## Vertex Coloring
**The problem:** color every vertex of a graph so that no two *connected* vertices share the same color — using as few colors as possible.

Think of it like a map: no two neighboring countries can be the same color. Or exam scheduling: two courses that share a student can't be in the same time slot (color = time slot).

**Your heuristic (Welsh-Powell):**
1. Sort vertices by how many neighbors they have (most-connected first) — because "popular" vertices are the hardest to color, so color them first while you still have colors available.
2. Go through vertices in that order. For each one, look at the colors already used by its neighbors, and give it the smallest color number *not* already used by a neighbor.

It won't always find the mathematically perfect minimum number of colors (that's NP-hard), but it gives a valid, reasonably good coloring fast.

## PageRank
**The problem:** rank the "importance" of pages/vertices in a network based on who links to whom — this is literally the original Google algorithm.

**The intuition:** a page is important if *important pages* link to it. It's circular (importance depends on importance), so you solve it by repeatedly refining a guess:

1. Start by giving every page equal importance (`1/N`).
2. Each round, every page "gives away" its current importance, split evenly among the pages it links to.
3. Each page's new importance = sum of what it received from everyone linking to it (with a small damping factor `d` mixed in, representing "a random surfer sometimes just jumps to a totally random page instead of following links" — this keeps the math well-behaved).
4. Repeat until the numbers stop changing much (converge).

Dangling pages (no outgoing links) are a gotcha — if you don't handle them, all the importance flowing into them just vanishes. So you spread their importance back out to everyone instead.

End result: a probability distribution over all vertices — every vertex's rank is between 0 and 1, and they all sum to 1.

## K-Means Clustering
**The problem:** given a bunch of points scattered in space, group them into K clusters of "similar" points, with no labels given in advance (unsupervised).

**The idea (like sorting fruit into K baskets by size):**
1. Pick K starting points as initial "centroids" (cluster centers) — you're told to just use the first K points for simplicity.
2. **Assign step**: put every point into whichever cluster's centroid is closest (straight-line/Euclidean distance).
3. **Update step**: recompute each centroid as the *average* position of all points now in that cluster.
4. Repeat assign → update → assign → update... The clusters slowly settle into place, like the baskets shifting until each one really does contain the closest points.
5. Stop when points stop switching clusters (or centroids barely move).

**WCSS** (within-cluster sum of squares) is just "how tightly packed are the points around their centroid" — lower is better, and it's your goodness-of-fit metric.

## FastMap
**The problem:** you don't have coordinates for your objects — you only know the *distance* between every pair of them (e.g., "flight time between city A and city B"). You want to place them on a 2D or 3D map such that the distances on the map roughly match the real distances.

**The intuition:** Imagine you only know how far apart cities are (as numbers), not their actual map positions — FastMap reverse-engineers a plausible map.

1. Find two objects that are (roughly) the farthest apart from each other — call them the "pivots" for this dimension. (Heuristic: pick a random object, find what's farthest from it, then find what's farthest from *that* — two hops usually finds a good far-apart pair without checking every pair.)
2. Draw an imaginary line between the two pivots. Project every other object onto that line using the law of cosines (basically: "given how far X is from pivot1 and pivot2, and how far the pivots are from each other, where does X sit along that line?"). That projected position becomes the object's coordinate for this dimension.
3. "Subtract" (deflate) the part of each distance already explained by this dimension, so the next dimension captures new information instead of repeating the same one.
4. Repeat for however many dimensions (k) you want.

It's a cheap, fast alternative to more "proper" dimensionality-reduction math (like PCA/MDS), used a lot when you only have pairwise similarity scores and no underlying coordinates to begin with.

---


Here's the compact reference — complexities + core pseudocode (V = vertices, E = edges, N = points/objects, K = clusters, D = dimensions, B = block size, I = iterations).

## Assignment 3

### Kruskal's MST
**Time:** O(E log E) — dominated by sorting edges. **Space:** O(V + E) for DSU + edge list.

```
sort all edges by weight ascending
for each edge (u, v, w) in sorted order:
    if find(u) != find(v):          # different components?
        union(u, v)                 # merge them
        add edge to MST, add w to total
stop when V-1 edges added
```

### Prim's MST
**Time:** O(E log V) with a min-heap. **Space:** O(V + E).

```
start with vertex 0 in tree, push its edges into min-heap
while heap not empty and tree incomplete:
    pop cheapest edge (u, v, w) from heap
    if v already in tree: skip
    add v to tree, add w to total, mark v visited
    push all edges from v to unvisited neighbors
```

### Gradient Descent
**Time:** O(d · I) — d = polynomial degree (evaluating f' each iteration), I = iterations. **Space:** O(d) for coefficients.

```
x = initial_x
for i in 1..max_iterations:
    grad = f'(x)                    # evaluate derivative at x
    if |grad| <= tolerance: break
    x = x - learning_rate * grad
return x, f(x)
```

### Maxflow–Mincut (Dinic's)
**Time:** O(V² · E) general graphs (much faster in practice). **Space:** O(V + E) for residual graph.

```
while BFS from s can reach t in residual graph (build level graph):
    while DFS finds an augmenting path s -> t using level graph:
        push = min residual capacity along path
        update residual capacities forward (-push) and reverse (+push)
maxflow = sum of all pushes
mincut = vertices still reachable from s in final residual graph
```

## Assignment 4

### Vertex Coloring (Welsh-Powell)
**Time:** O(V² ) naive (O(V + E) with smarter neighbor-color lookups). **Space:** O(V + E).

```
compute degree of every vertex
sort vertices by degree, descending
for each vertex u in that order:
    used_colors = colors of u's already-colored neighbors
    assign u the smallest color not in used_colors
```

### PageRank
**Time:** O((V + E) · I) — each iteration scans all edges once. **Space:** O(V + E).

```
rank[v] = 1/N for all v
repeat up to max_iterations:
    new_rank[v] = (1-d)/N + d * sum(rank[u]/outdeg[u] for u -> v)   # for every v
    redistribute dangling nodes' rank evenly to all vertices
    if sum(|new_rank - rank|) <= tolerance: break
    rank = new_rank
```

### K-Means Clustering
**Time:** O(N · K · D · I). **Space:** O(N · D + K · D).

```
centroids = first K points
repeat until no point changes cluster or max_iterations:
    for each point: assign to nearest centroid (Euclidean distance)
    for each cluster: new centroid = mean of its assigned points
    (if cluster empty: keep old centroid)
```

### FastMap
**Time:** O(N · k) — k = target dimensions, each dim needs a few O(N) passes to pick pivots + project. **Space:** O(N · k) for coordinates (avoid storing full N×N if you can).

```
for each of k dimensions:
    pick pivots (a, b) ~farthest apart (2-hop heuristic: random -> farthest -> farthest again)
    for every object x:
        coord[x] = (d(a,x)^2 + d(a,b)^2 - d(b,x)^2) / (2 * d(a,b))   # law of cosines
    deflate remaining pairwise distances to remove this dimension's contribution
```

---

**Quick complexity summary table:**

| Algorithm | Time | Space |
|---|---|---|
| Kruskal | O(E log E) | O(V+E) |
| Prim | O(E log V) | O(V+E) |
| Gradient Descent | O(d·I) | O(d) |
| Maxflow (Dinic's) | O(V²·E) | O(V+E) |
| Vertex Coloring | O(V+E) | O(V+E) |
| PageRank | O((V+E)·I) | O(V+E) |
| K-Means | O(N·K·D·I) | O(N·D + K·D) |
| FastMap | O(N·k) | O(N·k) |

If you want, I can fold this + the beginner explanations into the strategy doc from before as one consolidated cheat sheet.
