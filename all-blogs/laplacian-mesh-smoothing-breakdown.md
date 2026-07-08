# This is my code implementation+breakdown+documentation of Laplacian Mesh Smoothing Using VEX in Houdini

In the below code, I implement Laplacian Mesh Smoothing using the actual maths formulas behind the Smooth and Attrib Blur Nodes in Houdini. I do this for the position attribute of the points ('vertices' are called points in Houdini). This can be done for other data too, like colour.

## Some additional implementation-related information apart from the theoretical discussion on the algo in my [previous blog↗](laplacian-mesh-smoother.md)

- There are lots of ways to get something to work in Houdini, but also a lot of incorrect ways that leave you wondering what's wrong even if your code is fully sound. So it's important to first create a clean setup, using the nodes/wrangles that work for the type of problem we wish to code. This algorithm is an iterative algorithm, meaning it runs operations for one point at a time. Of course, in Houdini we have multithreading/parallel processing. If we write it in a pointwrangle, this executes at the exact same time for all the points. The issue here is that all the threads will be reading an empty cotangent matrix. The correct approach is using a solver. Solvers are used for temporal algorithms, like iterative ones.
  
- Optimizing memory space usage. As discussed in the theory, the cotangent matrix (which i will refer to as cot_mat here on) formed is very sparse because a point generally only has a few neighbours (too many neighbours may mean a very tangled mesh). This will slow down the program a lot if the geometry is even a tiny bit more complex than the absolute basic shapes like cubes. **THE PRODUCTION PARADIGM**: In DCC tools, matrix-free appraoches are preferred (and practical). Houdini already has very highly optimized techniques of storing geometry and accessing its attributes. Each point is a row and each edge is a column in the cot_mat.

- Maths used (BEST PRACTICES/**INDUSTRY STANDARDS**):    
a. Cotangent formula used is $$\cot(\theta) = \frac{A \cdot B}{|A \times B|}$$, where A and B are vectors, $$A \cdot B = AB\cos(\theta)$$ and $$|A \times B| = AB\sin(\theta)$$. The product $AB$ cancels out and we are left with the formula for cotangent. Trig functions are notoriously heavy to compute and can easily break if values input are not within the correct range, so we avoid any cosine and sine calculations at all.

b. No actual matrix is created for the weights. This would in fact be equivalent to taking many steps backward efficiency-wise because Houdini a highly optimized method to store geometry meshes. Each point already has its 1-ring neighbours stored within structures that allow O(1) (that is constant time) access. They are retrieved using the simple 'neighbours(node_num, point_num)' function.   
What I used instead is a dictionary for every point. And each dictionary contains _(key: neighbour_num; value: cotangent weight of this neighbour with current point)_. This seemed like one of the best options because 1. this allows constant time access to weight for any point-neighbour pair, and 2. Each point has a variable number of neighbours and dictionaries can easily adapt to this, since the value can be any data type in a dictionary in VEX Houdini.

  
## The Node Setup 

1. Iterate over all neighbours of each point and store them in an array.

```
// write function to calculate cotangent weight of a point 
function float get_cot_wt(int pt; int nbr) {
    // we have the point pt and its neighbor nbr
    // get the shared edge
    int hedge1=pointedge(0, pt, nbr);
    // get the adjacent triangles (their primitive numbers) from this half edge
    int hedge2=hedge_nextequiv(0,hedge1);
    int tri_num1=hedge_prim(0,hedge1);
    int tri_num2=hedge_prim(0,hedge2);
    int h1=hedge_next(0,hedge1);
    int t1=hedge_dstpoint(0,h1);
    vector v1_a=point(0,"P",pt)-point(0,"P",t1);
    vector v1_b=point(0,"P",nbr)-point(0,"P",t1);
    int h2=hedge_prev(0,hedge2);
    int t2=hedge_srcpoint(0,h2);
    vector v2_a=point(0,"P",t2)-point(0,"P",pt);
    vector v2_b=point(0,"P",t2)-point(0,"P",nbr);
    float cot_theta1 = dot(v1_a,v1_b) / max(length(cross(v1_a,v1_b)), 0.00001);
    float cot_theta2 = dot(v2_a,v2_b) / max(length(cross(v2_a,v2_b)), 0.00001);
    float cot_wt=(cot_theta1+cot_theta2)/2;
    return cot_wt;
}

// get all attributes from detailwrangle
dict cot_wts=point(0,"cot_wts",@ptnum);
float lambda=detail(0,"lambda");
float mu=detail(0,"mu");

// this code runs for each point in the mesh
// loop over all 1-ring neighbours of current point
foreach(int nbr; neighbours(0,@ptnum)) {
    // calculate cotangent_weight of this point
    float curr_wt=-1.0;
    if(@ptnum!=nbr) curr_wt=get_cot_wt(@ptnum,nbr);
    // store the weight as value with nbr as dictionary
    cot_wts[itoa(nbr)]=curr_wt;
}
float sum_wts=0;
foreach(string key; keys(cot_wts)) sum_wts+=cot_wts[key];
cot_wts[itoa(@ptnum)]=sum_wts;
// update the dictionary attribute for current point
setpointattrib(0,"cot_wts",@ptnum,cot_wts,"set");
// now we have all the cotangent weights
// the position of current point can be updated. find the delta_p
vector delta_p=set(0);
foreach(int nbr; neighbours(0,@ptnum)) {
    float wt=cot_wts[itoa(nbr)];
    vector pos_nbr=point(0,"P",nbr);
    delta_p+=wt*(pos_nbr-@P);
}

vector normalized_delta_p=delta_p/sum_wts;

// update the position
if(@Frame%2==0) @P+=lambda*normalized_delta_p;
else @P+=mu*normalized_delta_p;
```

