# VEX Implementation: Laplacian (Taubin) Mesh Smoothing 

Below is my code implementation and documentation for Laplacian Mesh Smoothing using VEX in Houdini, specifically mirroring the mathematics behind Houdini's native Smooth and Attribute Blur nodes. 

## Architectural Considerations

1. **Solver Logic vs. Parallel Processing:**
Houdini executes point wrangles in parallel across all threads. Because this is an iterative algorithm (requiring temporal state updates), running this directly in a standard point wrangle would result in all threads attempting to read an empty cotangent matrix simultaneously. The correct architectural approach is to encapsulate the logic within a Solver SOP to manage the temporal iteration.

2. **Memory Optimization (The Production Paradigm):**
The mathematical cotangent matrix is incredibly sparse. In DCC tools, matrix-free approaches are preferred. Houdini already possesses highly optimized techniques for storing and accessing geometry attributes in $O(1)$ time (e.g., `neighbours(0, @ptnum)`). Creating a literal $N \times N$ matrix in memory is highly inefficient. Instead, I utilize a VEX dictionary for each point, storing key-value pairs of `[neighbor_num : cotangent_weight]`.

3. **Mathematical Best Practices:**
The standard formula $\cot(\theta) = \frac{A \cdot B}{|A \times B|}$ is used. By avoiding direct cosine and sine calculations, we bypass the computational weight and potential boundary-value errors associated with trigonometric functions. Normalizing vectors at specific stages is also critical to prevent floating-point explosions when scaling geometry.

## Node Setup & VEX Logic

*For visual node network, see original documentation.*

### Initialization (Detail Wrangle)
Establish the $\lambda$ (smoothing) and $\mu$ (inflation) values for Taubin Smoothing.
```c
// positive lambda for smoothing - range 0 to 1
float lambda = chf("lambda");
f@lambda = lambda;

// negative mu (to prevent excessive shrinkage) -  range -1 to 0
float mu = chf("mu");
f@mu = mu;
// Retrieve attributes
dict cot_wts = point(0, "cot_wts", @ptnum);
float lambda = detail(0, "lambda");
float mu = detail(0, "mu");

// Calculate weights for 1-ring neighbors
foreach(int nbr; neighbours(0, @ptnum)) {
    float curr_wt = -1.0;
    if(@ptnum != nbr) curr_wt = get_cot_wt(@ptnum, nbr);
    cot_wts[itoa(nbr)] = curr_wt;
}

// Calculate sum of weights
float sum_wts = 0;
foreach(string key; keys(cot_wts)) sum_wts += cot_wts[key];
setpointattrib(0, "cot_wts", @ptnum, cot_wts, "set");

// Update position delta
vector delta_p = set(0);
foreach(int nbr; neighbours(0, @ptnum)) {
    float wt = cot_wts[itoa(nbr)];
    vector pos_nbr = point(0, "P", nbr);
    delta_p += wt * (pos_nbr - @P);
}

// Normalize delta to maintain scale ratios
vector normalized_delta_p = delta_p / sum_wts;

// Taubin Smoothing application (alternate smoothing and inflation)
if(@Frame % 2 == 0) @P += lambda * normalized_delta_p;
else @P += mu * normalized_delta_p;
function float get_cot_wt(int pt; int nbr) {
    int hedge1 = pointedge(0, pt, nbr);
    int hedge2 = hedge_nextequiv(0, hedge1);
    
    int h1 = hedge_next(0, hedge1);
    int t1 = hedge_dstpoint(0, h1);
    vector v1_a = point(0, "P", pt) - point(0, "P", t1);
    vector v1_b = point(0, "P", nbr) - point(0, "P", t1);
    
    int h2 = hedge_prev(0, hedge2);
    int t2 = hedge_srcpoint(0, h2);
    vector v2_a = point(0, "P", t2) - point(0, "P", pt);
    vector v2_b = point(0, "P", t2) - point(0, "P", nbr);
    
    // Calculate cotangents
    float cot_theta1 = dot(v1_a, v1_b) / max(length(cross(v1_a, v1_b)), 0.00001);
    float cot_theta2 = dot(v2_a, v2_b) / max(length(cross(v2_a, v2_b)), 0.00001);
    
    return (cot_theta1 + cot_theta2) / 2.0;
}
```
