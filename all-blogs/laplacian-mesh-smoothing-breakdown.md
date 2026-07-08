# This is my code implementation+breakdown+documentation of Laplacian (Taubin) Mesh Smoothing Using VEX in Houdini

In the below code, I implement Laplacian Mesh Smoothing using the actual maths formulas behind the Smooth and Attrib Blur Nodes in Houdini. I do this for the position attribute of the points ('vertices' are called points in Houdini). This can be done for other data too, like colour.

## Some additional implementation-related information apart from the theoretical discussion on the algo in my [previous blog↗](laplacian-mesh-smoother.md)

- There are lots of ways to get something to work in Houdini, but also a lot of incorrect ways that leave you wondering what's wrong even if your code is fully sound. So it's important to first create a clean setup, using the nodes/wrangles that work for the type of problem we wish to code. This algorithm is an iterative algorithm, meaning it runs operations for one point at a time. Of course, in Houdini we have multithreading/parallel processing. If we write it in a pointwrangle, this executes at the exact same time for all the points. The issue here is that all the threads will be reading an empty cotangent matrix. The correct approach is using a solver. Solvers are used for temporal algorithms, like iterative ones.
  
- Optimizing memory space usage. As discussed in the theory, the cotangent matrix (which i will refer to as cot_mat here on) formed is very sparse because a point generally only has a few neighbours (too many neighbours may mean a very tangled mesh). This will slow down the program a lot if the geometry is even a tiny bit more complex than the absolute basic shapes like cubes. **THE PRODUCTION PARADIGM**: In DCC tools, matrix-free appraoches are preferred (and practical). Houdini already has very highly optimized techniques of storing geometry and accessing its attributes. Each point is a row and each edge is a column in the cot_mat.

- Maths used (BEST PRACTICES/**INDUSTRY STANDARDS**):    
a. Cotangent formula used is $$\cot(\theta) = \frac{A \cdot B}{|A \times B|}$$, where A and B are vectors, $$A \cdot B = AB\cos(\theta)$$ and $$|A \times B| = AB\sin(\theta)$$. The product $AB$ cancels out and we are left with the formula for cotangent. Trig functions are notoriously heavy to compute and can easily break if values input are not within the correct range, so we avoid any cosine and sine calculations at all.

b. No actual matrix is created for the weights. This would in fact be equivalent to taking many steps backward efficiency-wise because Houdini a highly optimized method to store geometry meshes. Each point already has its 1-ring neighbours stored within structures that allow O(1) (that is constant time) access. They are retrieved using the simple 'neighbours(node_num, point_num)' function.   
What I used instead is a dictionary for every point. And each dictionary contains _(key: neighbour_num; value: cotangent weight of this neighbour with current point)_. This seemed like one of the best options because 1. this allows constant time access to weight for any point-neighbour pair, and 2. Each point has a variable number of neighbours and dictionaries can easily adapt to this, since the value can be any data type in a dictionary in VEX Houdini.

c. When to normalize vectors? I have normalized some vectors in the custom cotangent weight calculations fucntion. We need to keep in mind the values a function may return and how they can change when operations are done them. When working with vectors is the magnitude is not really necessary for the computation or may be cancelled out by some operations later on, it is good to normalize them. This way, you preent bugs that may explode or collapse your geometry in case you experiment with very large or very small scale of geometry. An example will be shown when I comee to the cotangent weight calculations function.
  
## The Node Setup 

<img width="1470" height="956" alt="Screenshot 2026-07-08 at 2 03 38 PM" src="https://github.com/user-attachments/assets/7f230d40-df3d-4b1a-a89d-10d6bfbfba2c" />

1. Choose any geometry and remesh it so that our smoothing is easier to see.
2. Insert a detail wrangle (init_multipliers). This will control values of $λ$ and $μ$ (for Taubin Smoothing). These values will be same throughout the algorithm runtime. Generally, $abs(μ)<abs(λ)$. You can experiment with different combinations; I chose $μ$ 0.2 and $λ$ 0.6.
``` 
// positive lambda for smoothing - range 0 to 1
float lambda=chf("lambda");
// save as attribute
f@lambda=lambda;
// negative mu (to prevent excessive shrinkage) -  range -1 to 0
float mu=chf("mu");
// save as attribute
f@mu=mu;
```

4. Then, insert a point wrangle (init_point_attributes). This contains the attributes that are unique for each point, unlike those in the detail wrangle. As mentioned above, I will be using this dict to implement a more practical version of the cotangent weight matrix.
```
// create a dictionary attribute for each point to store dict["p"]={q, cotangent_weight_pq}
dict cot_wts={};
// save as attribute
d@cot_wts=cot_wts;
```

5. Insert a solver. We need the algorithm to run for each point, iteratively (NOT in parallel). You may also use a for-each-number loop if you want to control the number of iterations but you will only be able to see the final result and not the intermediate stages using that. Here, I used a solver.

## Coding The Algorithm (+Breakdown of code)

1. Go into the solver and insert a point wrangle. Let's name it 'laplacian_mesh_smoother'.

2. The main body of the algorithm is below:

Remember to get the attributes from upstream first!
```
// get all attributes from detailwrangle
dict cot_wts=point(0,"cot_wts",@ptnum);
float lambda=detail(0,"lambda");
float mu=detail(0,"mu");
```

Main code block
```
// this code runs for each point in the mesh
// loop over all 1-ring neighbours of current point
foreach(int nbr; neighbours(0,@ptnum)) {
    // calculate cotangent_weight of this point using our own function get_cot_wts
    float curr_wt=-1.0;
    // remember, the formula in the screenshot on the theory blog page says that the weight with the point's own self has a different formula
    if(@ptnum!=nbr) curr_wt=get_cot_wt(@ptnum,nbr);
    // store the weight as value with nbr as dictionary
    cot_wts[itoa(nbr)]=curr_wt;
}
// get summation of the weights of all its neighbours
float sum_wts=0;
foreach(string key; keys(cot_wts)) sum_wts+=cot_wts[key];
// update the dictionary attribute for current point
setpointattrib(0,"cot_wts",@ptnum,cot_wts,"set");
// now we have all the cotangent weights
// the position of current point can be updated. find the delta_p
// initialize vector
vector delta_p=set(0);
// iterate over all 1-ring neighbours and add them
foreach(int nbr; neighbours(0,@ptnum)) {
    // get weight for current neighbour from dictionary
    float wt=cot_wts[itoa(nbr)];
    // get the world coordinate for this neighbour
    vector pos_nbr=point(0,"P",nbr);
    // use the formula for updating p: weight multiplied by the difference vector of (neighbour and current point).
    delta_p+=wt*(pos_nbr-@P);
}
// this step was not mentioned in the maths that i read online, but it is important when implementing. without this, it is the mesh exploded! when doing things practically, it is important to consider real ratios.      
vector normalized_delta_p=delta_p/sum_wts;

// finally, we update the position
// this code just makes the inflation and smoothing to alternate. at even frames, we do regular smoothing. at odd frames, we use the taubin smoothing factor (inflation).
if(@Frame%2==0) @P+=lambda*normalized_delta_p;
else @P+=mu*normalized_delta_p;
```

3. Writing the cotangent weights calculation function.

<img width="919" height="256" alt="Screenshot 2026-07-08 at 2 59 16 PM" src="https://github.com/user-attachments/assets/29c873da-968b-4f40-af94-e83647f44e12" />


```
// write function to calculate cotangent weight of a point 
function float get_cot_wt(int pt; int nbr) {
    // we have the point pt and its neighbor nbr
    // get the shared gedge
    int hedge1=pointedge(0, pt, nbr);
    // get the next equivalent hedge (which will be used to get the adjacent triangle)
    int hedge2=hedge_nextequiv(0,hedge1);
    // get the first triangle primitive from the first hedge
    int tri_num1=hedge_prim(0,hedge1);
    // get the adjacent traignle using the next equivalent hedge
    int tri_num2=hedge_prim(0,hedge2);
    // get the vectors needed to calculate the angles needed to find weight
    int h1=hedge_next(0,hedge1);
    int t1=hedge_dstpoint(0,h1);
    vector v1_a=point(0,"P",pt)-point(0,"P",t1);
    vector v1_b=point(0,"P",nbr)-point(0,"P",t1);
    int h2=hedge_prev(0,hedge2);
    int t2=hedge_srcpoint(0,h2);
    vector v2_a=point(0,"P",t2)-point(0,"P",pt);
    vector v2_b=point(0,"P",t2)-point(0,"P",nbr);
    // finding cotangent of the angles
    float cot_theta1 = dot(v1_a,v1_b) / max(length(cross(v1_a,v1_b)), 0.00001);
    float cot_theta2 = dot(v2_a,v2_b) / max(length(cross(v2_a,v2_b)), 0.00001);
    // finally average of both values is the weight
    float cot_wt=(cot_theta1+cot_theta2)/2;
    // make sure to return this value because i specified float type as return type
    return cot_wt;
}
```
