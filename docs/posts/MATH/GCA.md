# Geometry: Computation and Algorithm

## Exercises

### Ex 10.10

> Show that every ridge is incident to exactly two facets.

This can be easily proved with dual polytope, where ridge is mapped to an edge and facet is mapped to vertex. Clearly an edge will be incident to two vertices (this can be shown by considering walking along the edge in both directions and it will get somewhere that it can no longer move, which is exactly the two vertices, corresponding to two directions along the edge).

If we want to directly prove this, we can consider ridge together with some other vertex, say $S = A \cup \left\{ v \right\}$. If $S$ is not a facet, then there must be two other vertices such that the line segment between the two vertices intersect with $\text{conv}(S)$. So $\text{conv}(S)$ naturally divide the polytope into two parts, so we can further divide and conquer until find two vertex finally together with $A$ gives a facet.