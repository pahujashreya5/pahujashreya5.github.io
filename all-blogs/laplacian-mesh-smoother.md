# Coding a Laplacian Mesh Smoother using VEX

## Introduction To Laplacian Smoothing

Mesh smoothing is the process of removing high-frequency noise from mesh geometry. While Houdini handles this natively via the [Smooth 2.0 Node](https://www.sidefx.com/docs/houdini/nodes/sop/smooth.html) or [Attribute Blur Node](https://www.sidefx.com/docs/houdini/nodes/sop/attribblur.html), understanding the underlying mathematics is crucial for pipeline development.

Laplacian smoothing repositions every point (or *vertex*, in standard geometry terminology) to the average position of its first-ring (directly adjacent) neighbors. 

### Smoothing Variations

#### 1. Uniform Laplacian Smoothing
"Uniform" assigns equal weights to all neighbors. This is computationally simple but mathematically inaccurate for complex meshes. It assumes uniform triangulation; otherwise, the mesh is simply drawn inward toward its geometric center. 

#### 2. Cotangent Laplacian Smoothing
This is the industry-standard, mathematically accurate approach. It utilizes a cotangent matrix for weighting neighbors, preserving the topological information regarding the shape and size of the triangles. 

The weight is calculated via the average of the cotangents of angles $\alpha_{ij}$ and $\beta_{ij}$.

<img width="412" height="261" alt="image" src="https://github.com/user-attachments/assets/3de9d808-a7fd-4b2d-8b07-d42a6b25a119" />

In this article, I dive into the code and algorithmic structure under the hood of Houdini's Laplacian geometry nodes, specifically focusing on generating and applying the cotangent Laplacian matrix.

## Algorithmic Intuition & Pseudocode

**Step 1:** Define the mesh geometry.

**Step 2:** Construct the cotangent matrix.

<img width="905" height="494" alt="Screenshot 2026-07-07 at 1 15 03 PM" src="https://github.com/user-attachments/assets/ed08dd88-4a9f-4a35-a782-4767f8d2ee12" />

The non-diagonal entry is $\delta_{pq}=\frac{1}{2}\left(\cot(\alpha_{pq}) + \cot(\beta_{pq})\right)$, where $p$ is the shared point of the two adjoining triangles, and $q$ is the second endpoint of the shared line primitive. 

**Optimization Note:** This matrix is incredibly sparse. Each point has its own row and column, but the cotangent weight is only calculated for immediate neighbors. When implementing this in VEX, it is critical to store *only* the non-zero connections to preserve memory.

The diagonal entry is defined as:
$\delta_{pp}=-\sum_i\delta_{pq_i}
=-\left[\frac{1}{2}\left(\cot(\alpha_{pq_1})+\cot(\beta_{pq_1})\right)
+\frac{1}{2}\left(\cot(\alpha_{pq_2})+\cot(\beta_{pq_2})\right)
+\cdots+
\frac{1}{2}\left(\cot(\alpha_{pq_n})+\cot(\beta_{pq_n})\right)\right].$

**Step 3:** Update the point data using:
$$p_{\text{new}} = p_{\text{old}} + \lambda \cdot C \cdot (q_i - p)$$

**Step 4:** Implement Taubin Smoothing. 
Increasing iterations naturally results in geometry shrinkage (volume loss). To prevent this, we utilize Taubin smoothing, which inflates the geometry between smoothing iterations by alternating a positive coefficient ($\lambda > 0$) with a negative coefficient ($\mu < 0$).

---
**References:**
1. [Stanford CS468: Smoothing](https://graphics.stanford.edu/courses/cs468-12-spring/LectureSlides/06_smoothing.pdf)
2. [Houdini Smooth Node](https://www.sidefx.com/docs/houdini/nodes/sop/smooth.html)
3. [Laplacian Mesh Optimization](https://igl.ethz.ch/projects/Laplacian-mesh-processing/Laplacian-mesh-optimization/lmo.pdf)
4. [Convergence of Cotangent Weights](https://ddg.math.uni-goettingen.de/pub/convergence_cotan.pdf)
