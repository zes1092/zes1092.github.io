+++
title = "All Pairs Shortest Paths, Rings and Semi Rings"
description = "An application of algebraic structure to a common computer science problem"
type = ["posts","post"]
tags = []
date = "2022-03-02"
categories = []
series = ["algos"]
toc = true
[ author ]
  name = "Gary Sun"
+++

## Introduction

The Single Source Shortest Paths (SSSP) problem is to find the shortest distance from one node to every other node in a graph. The All Pairs Shortest Paths (APSP) problem, is to determine for every node in the graph, the shortest distance to every other node in a graph. APSP typically has applications in routing algorithms, i.e. finding the shortest path from one location to another.

## Usual algorithms

Depending on the type of graph, for both SSSP and APSP there are different algorithms to solve the relevant problems. The tables below provide the fastest well known algorithm to solve SSSP and APSP

### Single Source Shortest Path

| Graph Type                   | Algorithm                       | Runtime                |
| ---------------------------- | ------------------------------- | ---------------------- |
| Unweighted                   | BFS                             | $O(E)$                 |
| Non Negative Edge Weights    | Dijkstra's                      | $O(V \text{lg} V + E)$ |
| General                      | Bellman-Ford                    | $O(VE)$                |
| Directed Acyclic Graph (DAG) | Topological Sort + Bellman-Ford | $O(V + E)$             |

### All Pairs Shortest Path

Ignoring DAGs, the first two algorithms for APSP are merely SSSP algorithms run from every vertex of the graph, whilst there are different algorithms for general graphs depending on the number of edges. It is also important to note that as the number of edges increases to $O(V^2)$ in a graph, Johnson's algorithm approaches $O(V^2 \text{lg} V + V^3)$, making it slower than Floyd-Warshall.

| Graph Type                | Algorithm                 | Runtime                   |
| ------------------------- | ------------------------- | ------------------------- |
| Unweighted                | $\|V\| \times$ BFS        | $O(VE)$                   |
| Non Negative Edge Weights | $\|V\| \times$ Dijkstra's | $O(V^2 \text{lg} V + VE)$ |
| General (Dense)           | Floyd-Warshall            | $O(V^3)$                  |
| General (Sparse)          | Johnson's                 | $O(V^2 \text{lg} V + VE)$ |

## Floyd-Warshall and Matrix Multiplication

That being said a special shout out goes to Floyd-Warshall for it's simplicity - elegantly composed of a few lines of code.

```py
# Implementation of the Floyd-Warshall algorithm in Python

def floyd_warshall(graph: list[list[int]]):
    n = len(graph)
    for k in range(n):
        for j in range(n):
            for i in range(n):
                graph[i][j] = min(graph[i][j], graph[i][k] + graph[k][j])
```

It shares a structure very similar to Matrix Multiplication (MM) if the two given matrices are square matrices (which is always the case for Floyd-Warshall as all adjacency matrices are square).

```py
# Implementation of matrix multiplication for square matrices

def matrix_mul(A: list[list[int]], B: list[list[int]]) -> list[list[int]]:
    n = len(A)
    C = [[0] * n for _ in range(n)]
    for i in range(n):
        for j in range(n):
            for k in range(n):
                C[i][j] = sum(C[i][j], A[i][k] * B[k][j])
    return C
```

With the exception of creating result matrix and storing the calculations there, the algorithms both run in $O(V^3)$, except for two differences in operation

1. Instead of taking the cumulative `sum`, Floyd-Warshall takes the cumulative `min`
2. Instead of applying `*` between two values, Floyd-Warshall applies `+`

## Strassen's Algorithm

That being said there exists a sub $O(V^3)$ time for calculating the MM of two matrices, known as Strassen's algorithm. It is important to note that naive MM runs in $O(V^3)$, whilst matrix addition / subtraction runs in $O(V^2)$, and so Strassen's algorithm aims to reuse computations and lower runtime by using less multiplications, but more addition / subtraction.

### Explanation

The naive method of MM is as follows

Given the $n \times n$ matrices $A$ and $B$, to calculate their product, we can split each matrix into smaller block matrices in each quadrant (i.e. $A_{1, 1}$ is the smaller $\frac{n}{2} \times \frac{n}{2}$ matrix in the top left of A).

$$
A =
\begin{bmatrix}
A_{1, 1} A_{1, 2} \\\\
A_{2, 1} A_{2, 2}
\end{bmatrix}
,
B =
\begin{bmatrix}
B_{1, 1} B_{1, 2} \\\\
B_{2, 1} B_{2, 2}
\end{bmatrix}
,
C =
\begin{bmatrix}
C_{1, 1} C_{1, 2} \\\\
C_{2, 1} C_{2, 2}
\end{bmatrix}
$$

As a result the naive algorithm requires 8 multiplications (and 4 additions).

1. $C_{1, 1} = A_{1, 1}B_{1, 1} + A_{1, 2}B_{2, 1}$
2. $C_{1, 2} = A_{1, 1}B_{1, 2} + A_{1, 2}B_{2, 2}$
3. $C_{2, 1} = A_{2, 1}B_{1, 1} + A_{2, 2}B_{2, 1}$
4. $C_{2, 2} = A_{2, 1}B_{1, 2} + A_{2, 2}B_{2, 2}$

In comparison, Strassen's algorithm requires 7 multiplications (and 18 additions / subtractions) by creating temporary matrices

1. $M_1 = (A_{1, 1} + A_{2, 2})(B_{1 ,1} + B_{2, 2})$
2. $M_2 = (A_{2, 1} + A_{2, 2})(B_{1, 1})$
3. $M_3 = (A_{1 ,1})(B_{1, 2} - B_{2, 2})$
4. $M_4 = (A_{2, 2})(B_{1, 2} - B_{2, 2})$
5. $M_5 = (A_{1, 1} + A_{1, 2})(B_{2, 2})$
6. $M_6 = (A_{2, 1} - A_{1, 1})(B_{1, 1} + B_{1, 2})$
7. $M_7 = (A_{1, 2} - A_{2, 2]})(B_{2, 1} + B_{2, 2})$

which can then be added and subtracted with each other to produce the final result

1. $C_{1, 1} = M_1 + M_4 - M_5 + M_7$
2. $C_{1, 2} = M_3 + M_5$
3. $C_{2, 1} = M_2 + M_4$
4. $C_{2, 2} = M_1 - M_2 + M_3 + M_6$

### Time Complexity

The final runtime of the aforementioned algorithms can be determined through the master's theorem
$$T(n) = aT \left( \frac{n}{b} \right) + f(n)$$
Where

- $a$ = Number of subproblems (no.# of smaller MMs required per subproblem)
- $b$ = Reduction in size per problem (2, as the smaller matrixes have size $\frac{n}{2}$)
- $f(n)$ = Cost of work per problem ($O(n^2)$, as we do matrix addition / subtraction)

|                     | Naive Solution                                  | Strassen's Algorithm                            |
| ------------------- | ----------------------------------------------- | ----------------------------------------------- |
| Recurrence Relation | $T(n) = 8T \left( \frac{n}{2} \right) + O(n^2)$ | $T(n) = 7T \left( \frac{n}{2} \right) + O(n^2)$ |
| Time Complexity     | $O(n^3)$                                        | $O(n^{\text{lg}7}) \approx O(n^{2.807})$        |

Not bad! We have managed to reduce the runtime to $O(n^{2.807})$, and there are algorithms that can achieve even lower asymptotic runtime (i.e. the Vassilevska Williams has a runtime of $O(n^{2.373}$), though their hidden constant times make them impractical. Now would it be possible to apply this to Floyd-Warshall?

Note: $n = V$, as $V$ is the number of vertices, which is the number of rows and columns in an adjacency matrix.

## Rings and Semi-Rings

The issue is that fast matrix multiplication can only be applied to any ring.

In mathematics, a ring is a set that

- has addition which must be commutative and associative
- has multiplication that must be associative
- has a zero (aka the identity element)
- has negatives (i.e. adding an element and it's negative produces the ring's zero element)
- has two distributive laws relating addition and multiplication

However, for APSP, it is usually defined in a semi-ring. The definition of a semi-ring is the same as a ring, but **without the requirement for a negative**.

This becomes relevant as the Strassen's reduces matrix multiplications by taking advantage of matrix subtractions as the **negative** of the `sum` to reuse calculations.
Looking back at our comparison between Floyd-Warshall and MM, we must therefore need a negative to `min` in order to apply Strassen's to APSP, which unfortunately does not exist, so we cannot apply fast MMs to APSP.

But what if we could define APSP in the domain of a ring?

## Transitive Closure

Don't worry, I didn't drag you through all that to learn 0 applications.
The Transitive Closure for an adjacency matrix $G$ is an adjacency matrix $G'$, where $G'_{i, j} = 1$ if there is a path from $v_i$ to $v_j$ in $G$.

Due to the simplicity of this problem, this can be implemented through an algorithm using boolean operations as shown below.

### Comparison with MM

```py
# Implementation of a function to "square" a boolean matrix

def boolean_matrix_squaring(graph: list[list[bool]]):
    n = len(graph)
    for k in range(n):
        for j in range(n):
            for i in range(n):
                graph[i][j] = graph[i][j] or (graph[i][k] and graph[k][j])
```

We can compare this to our MM implementation and note two differences

1. Instead of taking the cumulative `sum`, we take the cumulative `or`
2. Instead of applying `*` between two values, we apply `and`

The issue earlier was that fast MM is only defined for a ring, however TC is also [defined under a ring with boolean operations](http://math.mit.edu/~jerison/103/handouts/rings.pdf), and as a result we can apply Strassen's algorithm to the TC problem.

### Application

Now we can attempt to apply Strassen's algorithm to quickly find the TC of a graph.

#### Applying with Strassen's

To find the TC, we can more or less represent `true` with `1`, `false` with `0`, and then apply Strassen's algorithm to multiply the adjacency matrix with itself, always taking `mod 2` of the results in $O(V^{2.807})$.

The result of adjacency matrix multiplication produces the a matrix where $A_{i, j}$ represents if there is a path of length 2 from $v_i$ to $v_j$.

#### Reduction from TC to MM

Now it is possible to reduce (apply an algorithm to convert from one problem to another) the TC problem to MM by following the following steps.

1. Replace all strongly connected components with a single vertex in $O(V^2)$.
2. Topological sort the graph so edges move from lowered numbered vertices go to higher numbered ones in $O(V + E)$.
3. As a result, the adjacency matrix is an upper triangular matrix (as vertices only have a path to higher numbered vertices) $G$, which is can be composed of 4 quadrants:

   $$
   G =
   \left[ \begin{array}{c | c}
   A & C \\\\
   \hline
   0 & B \\\\
   \end{array} \right],
   $$

   where $A$ is the adjacency matrix for vertices $v_1, ... v_{n/2}$, and $B$ for $v_{(n/2) + 1}, ..., v_{n - 1}$.
   Meanwhile C represents the edges from the vertices in $A$ to $B$.

Now, we claim that the TC of $G$:

$$
G' =
\left[ \begin{array}{c | c}
A' & A' C B' \\\\
\hline
0 & B' \\\\
\end{array} \right].
$$

This is because the TC of $A$ is solely dependent on connections in $A$, and the same applies for $B$.

As for the upper right matrix, $C_{i, j}$ will be `true` if there is a path from $v_i$ to $v_j$, which is true if there is a path from $v_i$ to some vertex in $A'$, which has a path to some other vertex in $B'$ which is connected to $v_j$.

This path can be determined by taking the multiplication $A' \times C \times B'$ as $A'$, hence $C' = A' C B'$.

Since $G'$ can be calculated recursively by finding the TC of 2 half sized graphs ($A'$, $B'$) and 2 MMs ($A' C B'$), the final time complexity is represented by the recurrence relation

$$
T(n) = 2T \left( \frac{n}{2} \right) + O \left( n^{2.807} \right)
$$

which can be calculated to be $O(n^{2.803})$, the same runtime as the MM algorithm used! In fact, [TC is reducible to boolean MM](https://www.cs.bgu.ac.il/~dinitz/Course/SS-12/transitiveClosure.pdf).

### Repeated DFS

That being said, it is possible to apply DFS on every vertex of the graph to find the TC. However a single DFS of a graph has a runtime of $O(V + E)$ (which can be $O(V)$ for sparse graphs and $O(V^2)$ for dense graphs), and so the total runtime would be $O(V^2)$ for sparse graphs and $O(V^3)$ for dense graphs.

As a result the suitable algos for finding the Transitive Closure of a graph is listed below.

|           | Dense Graphs   | Sparse Graphs |
| --------- | -------------- | ------------- |
| Algorithm | Strassen's     | Repeated DFS  |
| Runtime   | $O(V^{2.807})$ | $O(V^2)$      |

## Conclusion

In conclusion, we've now learn that we can apply fast MMs to other problems defined in a ring, one of which was allowing us to find the TC of a dense graph quickly by reducing the problem to MM.
