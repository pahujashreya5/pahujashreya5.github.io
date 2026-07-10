# Optimal Shape Matching (1. Using Absolute Rigid Alignment (Quaternions) 2. As-Rigid-As-Possible (ARAP) Deformation)
1. Closed-form solution of absolute orientation using unit quaternions [https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf](https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf)
2. 

**Note:** This blog has some pre-requisties. Make sure you have at least some prior knowledge of linear algebra (matrices & differentiation mainly) and quaternions.

## Introduction 

### What Is Shape Matching (Basic definition)?

Say we have a point cloud representing some geometry. This cloud is shifted by some translation vector $t$ and rotated by some rotation matrix $R$. We get a new point cloud after these operations The problem statement is that we need to find these matrices to be able to get the original point cloud from the transformed one. 

_A helpful diagram from [https://graphicscomputing.fr/course/2025_2026/epita_ani3d/lecture_slides/04_simulation_rigids/html/02_pbd/06_shape_matching_use/index.html](https://graphicscomputing.fr/course/2025_2026/epita_ani3d/lecture_slides/04_simulation_rigids/html/02_pbd/06_shape_matching_use/index.html)_
<img width="1373" height="703" alt="Screenshot 2026-07-09 at 12 46 37 PM" src="https://github.com/user-attachments/assets/37844d73-e3ed-45da-abc9-6486e005438c" />

### Methods For Shape Matching

### Singular Value Decomposition (SVD)

Singular Value Decomposition is the generally most robust way to break a transformation matrix into translation and rotation. [https://nghiaho.com/?page_id=671&unapproved=1249516&moderation-hash=6d999e119bcd2fc25f1dfff5e03362c5#comment-1249516](https://nghiaho.com/?page_id=671&unapproved=1249516&moderation-hash=6d999e119bcd2fc25f1dfff5e03362c5#comment-1249516) is a lovely explanation of the algorithm.   
Their are 2 problems with SVD: it is computationally expensive and it often givess you a matrix that may be a reflection of the true original matrix. This then needs an additional check to confirm which one we have ended up with.

#### Practical Algos In Computer Graphics

The three major algorithms for shape matching in computer graphics are: 

1. Iteractive Closest Point: This is used when mapping from original point cloud to transformed point cloud is unkown. It guesses mapping based on proximity in each iteration. 
Under the hood: Uses gradient descent.

3. Principal Component Analysis Alignment: This is often used as a coarse step before ICP. It tries to align the cloud to guessed axes based on overall volume.
Under the hood: Uses eigen-decomposition.

5. As-Rigid-As-Possible Surface Modelling: Divides mesh into smaller parts and runs shape matching for each.
Under the hood: Also uses gradient descent.

**Note:** Apart from these, there are many new and upcoming methods which involve methods like dynamic programming or machine learning.

## Methods I Implemented

### 1. Absolute Rigid Alignment (Quaternions)
[See my VEX code here♥️]()

**Statement:** Implementing [Horn's solution](https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf) to find the optimal translation and rotation between two corresponding point sets. **(Closed-form solution of absolute orientation using unit quaternions)**

#### The Maths

The original solution to align 2 sets of 3D points did not use SVD; it used quaternons! From the paper:

> The transformation between two Cartesian coordinate systems can be thought of as the result of a rigid-body motion and can thus be decomposed into a rotation and a translation.

It is necessary to have at least 3 points for the number of variables we are required to solve for (translation DOF=3 + rotation DOF=3 + scaling DOF=1), ie, total 7 variables. 3 points will give us 9 equations in total (3 per point). This works for an approximate solution. In practice, this will result in an approximate solution, since solving such a system uses the least squares **approximation**. Therefore, it is wise to use more than 3 points.

This paper offers a closed-form solution. This means that it is not an iterative solution- it gives you an equation that will result in an answer upon solving it once. Moreover, there is no need to assume any values for a cold start, as would otherwise be the case in an iterative solution.

> I give the solution in a form in which unit quaternions are used to represent rotations. The solution for the desired quaternion is shown to be the eigenvector of a symmetric 4X4 matrix associated with the most positive eigenvalue.

I do not delve into the derivations of each step in this blog, but drop me an email if you would like one on that too :)

### Algorithm 
_Note: $r_i$ are vectors._

INTRO: Horn uses the idea of finding an error term with respect to each of the operations (rotation, translation and scaling) and then solves each of them by the error (by differentiation).

1. Horn starts with creating another coordinate system, with axes called 'left' and 'right. It uses the vectors created by coordinates of the points.
   
3. Matrix $M_l$ (l subscript for 'left') contains the x, y and z axes of the new coordinate system left, as columns. Similarly, we create a matrix $M_r$.
   
5. ROTATION: Now, $r_r = M_rM_lr_r$, where $r_l$ and $r_r$ are the coordinates of the points in left and right systems. This can be simply verified using matrix multiplication. From this, we get $R=M_rM_l$<sup>T</sup>, where R is the rotation matrix we will be solving for.
  
7. TRANSLATION: Here, Horn says that using the centroids of each of the 3 points will be equivalent to using their coordinates. It will also make our equations simpler. So using this replacement, we come to the simplified equation $r_0 = r_r-sRr_l$, where $r_0$ is the translation vector, s is the scale factor and $r_r$ and $r_l$ are now representing centroids of $r_{r_1},r_{r_2},r_{r_3}$  and $r_{l_1},r_{l_2},r_{l_3}$ respectively.

The total error to be minimized becomes: <img width="211" height="83" alt="Screenshot 2026-07-09 at 6 18 45 PM" src="https://github.com/user-attachments/assets/c8843eb6-cce4-4656-bd17-0fd85c172eac" />

> ...the translation is just the difference of the right centroid and the scaled and rotated left centroid.

8. SCALE: Using a. the error formula above and 2. the fact that the magnitude of the centroid vector remains the same after applying rotation, Horn derives a scale factor $s$. There is a problem of asymmetry when using the least squares formula: we subtract the calculated value from the ground truth (accurate value). But in shape matching, no point cloud is the ground truth! They both have noise. So as a solution, Horn has given a symmetric scale formula.
<img width="281" height="79" alt="Screenshot 2026-07-09 at 8 48 47 PM" src="https://github.com/user-attachments/assets/8f7ebffc-60f4-411f-87ae-6d9ac4a79ea8" />

Now we don't even need to calculate the rotation before solving for scale.

9. Lastly, regardless of which espression for $s$ we choose (asymmetrical or symmetrical), the minimum error is when $D=sum (i to n) dot(r_l_iR(r_l_i))$ is as large as possible. (because this is a term that is being subtracted in each of the expressions). This was derived from the assumption that the magnitude/length of vector remains same even after rotation.

10. On quaternions:

> Here I solve the problem of finding the rotation that maximizes D by using unit quaternions.

**SOLUTION to find the best rotation**

11. Usin quaternions, the expression for rotation of a 3D vector $r$ is $r_{rotated}=qrq^*$, where $q^*$ is the compelx conjugate of q. A simple explanation: think of $q$ as a complex number a+$i$b. The complex conjugate wuold be $q^*$=a-$i$b. Therefore, the mimum error expression in step 9 can now be written as

<img width="195" height="78" alt="Screenshot 2026-07-09 at 9 17 25 PM" src="https://github.com/user-attachments/assets/aa05c92c-0f5a-443f-ab31-5b0018ca6a5f" />

He mentions the benefits of using quaternions over matrices (unlike in SVD) because:
> it takes fewer arithmetic operations to multiply two quaternions than it does to multiply two 3 x 3 matrices.
> also, calculations are not carried out with infinite precision.
And lastly, it is not easier to find the unit quaternion given a quaternion than it is to find the orthogonal matrix given a matrix.

12. The formula in the image can be substituted with the appropriate variables and be written as $q^TNq$



### 2. As-Rigid-As-Possible Deformation
[See my VEX code here♥️]()

**Statement:** Implement a local-global iterative solver for non-rigid mesh deformation using non-linear least squares.


## Algorithm (Maths & Explanation)
Adapted from  and [https://igl.ethz.ch/projects/ARAP/svd_rot.pdf](https://igl.ethz.ch/projects/ARAP/svd_rot.pdf)

1. 

Problem with SVD
Because solving SVD implies some sort of randomness(there can be different number of correct solutions) it sometimes make rotation matrix to be sign reflected.
We can fix this by checking determinant of R matrix and if it negative. If so then multiply 3rd column of R by -1



## Sources ♥️
1. [https://nghiaho.com/?page_id=671](https://nghiaho.com/?page_id=671)
2. [https://igl.ethz.ch/projects/ARAP/svd_rot.pdf](https://igl.ethz.ch/projects/ARAP/svd_rot.pdf)
3. [https://cedarcantab.wordpress.com/2021/02/06/physics-based-animation-rigid-body-simulation-with-shape-matching/](https://cedarcantab.wordpress.com/2021/02/06/physics-based-animation-rigid-body-simulation-with-shape-matching/)
4. [https://webspace.science.uu.nl/~veltk101/publications/art/smi2001.pdf](https://webspace.science.uu.nl/~veltk101/publications/art/smi2001.pdf)
5. [https://graphicscomputing.fr/course/2025_2026/epita_ani3d/lecture_slides/04_simulation_rigids/html/02_pbd/06_shape_matching_use/index.html](https://graphicscomputing.fr/course/2025_2026/epita_ani3d/lecture_slides/04_simulation_rigids/html/02_pbd/06_shape_matching_use/index.html)
6. [https://medium.com/machine-learning-world/linear-algebra-points-matching-with-svd-in-3d-space-2553173e8fed](https://medium.com/machine-learning-world/linear-algebra-points-matching-with-svd-in-3d-space-2553173e8fed)
7. [https://docs.ros.org/en/indigo/api/libfovis/html/absolute__orientation__horn_8hpp_source.html](https://docs.ros.org/en/indigo/api/libfovis/html/absolute__orientation__horn_8hpp_source.html)
