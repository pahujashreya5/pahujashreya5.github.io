# KD-Tree Implementation in VEX (Breakdown & Documentation)

This post outlines my implementation and documentation of a KD-Tree using VEX in Houdini. 

[See my code files]()

The KD-Tree is inherently a recursive algorithm, which typically utilizes memory overhead (a recursion stack) to store the results of previous function calls. 

## A Note On Recursion in VEX

Implementing a KD-Tree using pure recursion in VEX presents a challenge. Because VEX is designed to compile code inline (it pastes all code into a single main block for extremely fast execution), it is not structurally designed for the overhead memory usage required by standard recursion. 

The industry-standard, memory-optimized method for traversing trees in environments like this is to utilize an explicit heap/stack data structure rather than relying on call-stack recursion. 

*(For a foundational explanation of the KD-Tree algorithm itself, I recommend [this Baeldung resource](https://www.baeldung.com/cs/k-d-trees).)*

## The Node Setup


## Code Breakdown


---
**References:**
1. [Baeldung: KD Trees](https://www.baeldung.com/cs/k-d-trees)
2. [Recursion in VEX](https://sergeneren.com/2018/09/23/recursion-in-vex/)
3. [VEX Video Tutorial](https://www.youtube.com/watch?v=V5i6KM_-8X0)
