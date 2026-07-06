This is my implementation + breakdown + documentation of KD Tree using VEX
[See my code files]()

This is a recursive algorithm, which means it uses memory overhead (recursion stack) to store the result of previous function calls. 

## A Note On Recursion in VEX

Implementing this using recursion in VEX would be pointlessly exhausting. Since VEX is designed to compile code inline (it just pastes all your code into one main block in its compiler and runs it for fast execution), it is inherently designed for no overhead memory usage-whithout which recursion can't happen.
The common memory-optimized method for trees is using a heap/stack. This data structure 

BTW, you can find the KD tree algorithm itself nicely explained here: [https://www.baeldung.com/cs/k-d-trees](https://www.baeldung.com/cs/k-d-trees)

## The Node Setup

## Coding the algorithm + Breakdown of Code


Sources ♥️
1. [https://www.baeldung.com/cs/k-d-trees](https://www.baeldung.com/cs/k-d-trees)
2. [https://sergeneren.com/2018/09/23/recursion-in-vex/](https://sergeneren.com/2018/09/23/recursion-in-vex/)
3. [https://www.youtube.com/watch?v=V5i6KM_-8X0](https://www.youtube.com/watch?v=V5i6KM_-8X0)
