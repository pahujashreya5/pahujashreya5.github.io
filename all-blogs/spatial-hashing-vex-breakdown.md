# This is my implementation + breakdown + documentation of spatial hashing using VEX

My work is adapted from the algorithm and C++ code explainde by Ten Minute Physics  
[https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf](https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf)  
[https://github.com/matthias-research/pages/blob/master/tenMinutePhysics/11-hashing.html](https://github.com/matthias-research/pages/blob/master/tenMinutePhysics/11-hashing.html)

## The Node Setup

I started by inserting a grid and scattering points on it. The use copy to points so that we have spheres at each of the points. This is because we need a non-zero radius to work with.

<img width="538" height="339" alt="Screenshot 2026-07-05 at 6 01 46 PM" src="https://github.com/user-attachments/assets/942fbf72-0215-4ec4-963f-7583f7385b19" />

Make sure we have spacing (which is side length of each cell in the grid) equal to twice the sphere radius. This is the assumption made in our algorithm that enables us to run it for a bounded region of 9 cells (or 27 in 3D). We use 'paste relative reference' to control the size of grid cells and spheres from the radius slider easily.

## Coding The Algorithm + Breakdown Of Code

### _Step 1:_ Defining Variables, Attributes & Data Structures

I declared the variables that are common across all points in a detail wrangle. Let's name this wrangle gloabl_vars.


```
// radius of each point
float radius=chf("radius");
f@radius=radius;
// size of each cell
float spacing=2*radius;
f@spacing=spacing;
// number of rows
int numx=chi("numx");
i@numx=numx;
// number of columns
int numy=chi("numy");
i@numy=numy;
```

Operations that are specific to each point should be done within a point wrangle. This way, these operations run for each point at the same time (multithreading) without having to explicitly define a loop. This also maintains easy to understand code and easy to access attributes.
I always try and maintain this separation. Otherwise, I get confused when coding the main algorithm because accessing upstream detail attributes and point attributes have different syntaxes.

```
// creating coordinates (xi,yi) and name it relative_pos
vector relative_pos;
// index of the one D array that we will store point numbers into
int array_idx;
// retrieve from upstream
// subtract 1 because we need number of columns and rows
int nx=detail(0,"numx")-1;
int ny=detail(0,"numy")-1;
float spc=detail(0,"spacing");
```
### _Step 2:_ Store the points into a 1D array

/// DESCRIPTION

```
relative_pos=floor(@P/spc); // this is set(xi,yi)
// v@relative_pos=relative_pos;
// find array_idx to which this point belongs
// remember we are in the XZ plane so we use relative_pos.z
// modulo with number of grid cells-1 because our vector (dict) indices are 0 to numx*numy-1
array_idx=int(relative_pos.x*nx+relative_pos.z)%(nx*ny); 
i@array_idx=array_idx; // save as point attribute
```

### _Step 3:_ 

Here, the algorithm begins. I will first store the points into an array. Then, I will create a dense array which saves memory in case of sparse data like the kind we have. Then, I will move on to the unbounded grid case (we are not limited to 9 cells anymore). Here is where spatial hashing comes into the picture. 
// 1. Store the points into the oned_array[]
To create the intermediary array, we need to have the dict keys in sorted order. VEX does not have an option to do this natively, so I used an 'ordering' array to access the elements of the dictionary in a sorted order, even though the dictionary itself is by default unsorted.

_Step 3:_ Create and intermediary array

_Step 4:_ Create the dense array (trick to save memory space)

_Step 5:_ 
