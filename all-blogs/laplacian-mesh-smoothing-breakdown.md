# This is my code implementation+breakdown+documentation of Laplacian Mesh Smoothing Using VEX in Houdini

In the below code, I implement Laplacian Mesh Smoothing using the actual maths formulas behind the Smooth and Attrib Blur Nodes in Houdini. I do this for the position attribute of the points ('vertices' are called points in Houdini). This can be done for other data too, like colour.

## Some additional implementation-related information apart from the theoretical discussion on the algo in my [previous blog↗](laplacian-mesh-smoother.md)

- There are lots of ways to get something to work in Houdini, but also a lot of incorrect ways that leave you wondering what's wrong even if your code is fully sound. So it's important to first create a clean setup, using the nodes/wrangles that work for the type of problem we wish to code. This algorithm is an iterative algorithm, meaning it runs operations for one point at a time. Of course, in Houdini we have multithreading/parallel processing. If we write it in a pointwrangle, this executes at the exact same time for all the points. The issue here is that all the threads will be reading an empty cotangent matrix. The correct approach is using a solver. Solvers are used for temporal algorithms, like iterative ones.
  
- Optimizing memory space usage. As discussed in the theory, the cotangent matrix (which i will refer to as cot_mat here on) formed is very sparse because a point generally only has a few neighbours (too many neighbours may mean a very tangled mesh). This will slow down the program a lot if the geometry is even a tiny bit more complex than the absolute basic shapes like cubes. THE PRODUCTION PARADIGM: In DCC tools, matrix-free appraoches are preferred (and practical). Houdini already has very highly optimized techniques of storing geometry and accessing its attributes. Each point is a row and each edge is a column in the cot_mat.
- 
## The Node Setup 

1. Iterate over all neighbours of each point and store them in an array. 

