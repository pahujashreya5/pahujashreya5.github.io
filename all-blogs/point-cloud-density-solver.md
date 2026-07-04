# Implementing Point Cloud Density Solver In VEX
[See VEX code files↗]

## Introduction

Point cloud density means how many points occupy a speicifc volume or area. A point cloud density solver in Houdini is meant to calculate and analyze (even adjust) the point cloud density of a point cloud geometry.
The Houdini nodes for this purpose are: Point Cloud Measure and Point Cloud Reduce. I will be implementing the maths (2 ways) behind this functionality.

## Let's Understand The Maths

### A Note

In recent years, neural networks are being used to calculate point cloud density more effectively. [https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/ipr2.13189](https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/ipr2.13189) and [https://www.researchgate.net/publication/381261077_A_Density-aware_Point_Cloud_Geometry_Compression_Leveraging_Cluster-centric_Processing](https://www.researchgate.net/publication/381261077_A_Density-aware_Point_Cloud_Geometry_Compression_Leveraging_Cluster-centric_Processing) are 2 wonderful works that detail these methods. In this blog, I will not touch upon these fields.

### The Brute Approach

One way to estimate density is to select a radius (r) and count the number of points within this radius. the number of points divided by the volume of this sphere with radius r is the point cloud density for this sphere! 
Another (faster but less precise) way is to measure the distance from our point to its nearest neighbour. This distance is then assumed to be radius r. Again, the point cloud density is now  number of points (equal to 1, which is our source point) divided by total volume (of sphere with radius r). 

Houdini manages thousands, even millions, of points at once. These methods are either to computationally-heavy, or too imprecise for real purposes. Hence, a data structure called KD-tree along with an algorithm called Spatial Hashing are used. Let's dive in!

### Method 1: (Fast) Spatial Hashing Algorithm

Regular hashing means using a function, say hash(), which combines some mathematical operations, say 2x+5, to output a 'hash value'. This value is then stored in an array. This is treated as an ID or a key to retrieve data from the array in constant time O(1).
Spatial hashing is when hashing is done for multi-dimensional values, like coordinates. This makes it useful for point clouds which could be 2D or 3D. The below video is a beautifully simple explanation of the algorithm of spatial hashing.   
This approach uses an additonal step of converting the sparse array to a dense array, which reduces memory usage greatly.
[https://www.youtube.com/watch?v=D2M8jTtKi44](https://www.youtube.com/watch?v=D2M8jTtKi44)

#### Code Breakdown [See VEX code file↗]()
This code is adapted from the [C++ code by Ten Minute Physics](https://github.com/matthias-research/pages/blob/master/tenMinutePhysics/11-hashing.html)

##### VEX Code


```

```

##### Visual Demo Showing Working Of Above Code In Houdini

### Method 2: KD Tree Data Structure

K-Dimensional tree is analogous to a binary search tree for more than one dimension. The median of a set of points is the root node, and the points less than the root node are recursively input for the left subtree while the points greater than or equal to the root node are recursively input for the right subtree. There are some optimizations and best practices to keep in mind when using KD tree for different purposes:

1. It is highly recommended to keep a check either on the algorithm's runtime or recursion stack, and terminate the algorithm if either of these two exceeds a certain limit. Else, the system might crash. Alternatively, a maximum distance or maximum points parameter may be maintained. These are some approximate methods that do well for certain purposes.
2. On high dimensional data, this algorithm may be barely any better than an exhaustive linear search. Commonly, it is advised to use this algorithm if n (the number of points) exceeds 2 raised to power k (where k is dimension of data) by a large difference, i.e., n>>2**k.

#### Code Breakdown [See VEX code file↗]()

### Comparison Between Method 1 & Method 2

Of course, there is no one '_better_' method. The choice may vary depending on purpose and type of data. Here's an [interesting thread](https://www.reddit.com/r/gamedev/comments/3jrtpc/quadtree_vs_spacial_hashing_which_to_use/) that disucsses this. 

Now, let's actually run both algorithms on different number of points and toggle various parameters to see the results! 

## Sources ♥️
1. [https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/ipr2.13189](https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/ipr2.13189)
2. [https://in.mathworks.com/matlabcentral/answers/563603-how-to-compute-the-density-of-a-3d-point-cloud](https://in.mathworks.com/matlabcentral/answers/563603-how-to-compute-the-density-of-a-3d-point-cloud)
3. [https://www.researchgate.net/publication/367149994_DENSITY-BASED_METHOD_FOR_BUILDING_DETECTION_FROM_LIDAR_POINT_CLOUD](https://www.researchgate.net/publication/367149994_DENSITY-BASED_METHOD_FOR_BUILDING_DETECTION_FROM_LIDAR_POINT_CLOUD)
4. [https://www.researchgate.net/publication/381261077_A_Density-aware_Point_Cloud_Geometry_Compression_Leveraging_Cluster-centric_Processing](https://www.researchgate.net/publication/381261077_A_Density-aware_Point_Cloud_Geometry_Compression_Leveraging_Cluster-centric_Processing)
5. [https://www.open3d.org/docs/0.19.0/python_api/open3d.geometry.PointCloud.html](https://www.open3d.org/docs/0.19.0/python_api/open3d.geometry.PointCloud.html)
6. [https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf](https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf)
7. [https://history.siggraph.org/wp-content/uploads/2022/08/2020-Talks-Gautron_Real-Time-Ray-Traced-Ambient-Occlusion-of-Complex-Scenes.pdf](https://history.siggraph.org/wp-content/uploads/2022/08/2020-Talks-Gautron_Real-Time-Ray-Traced-Ambient-Occlusion-of-Complex-Scenes.pdf)
8. [https://yasenh.github.io/post/kd-tree/](https://yasenh.github.io/post/kd-tree/)
9. [https://www.youtube.com/watch?v=D2M8jTtKi44](https://www.youtube.com/watch?v=D2M8jTtKi44)
10. [https://en.wikipedia.org/wiki/K-d_tree](https://en.wikipedia.org/wiki/K-d_tree)
11. [https://www.reddit.com/r/gamedev/comments/3jrtpc/quadtree_vs_spacial_hashing_which_to_use/](https://www.reddit.com/r/gamedev/comments/3jrtpc/quadtree_vs_spacial_hashing_which_to_use/)

