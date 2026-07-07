# Coding a Laplacian Mesh Smoother using VEX

## Introduction To Laplacian Smoothing

Mesh smoothing refers to removing high frequency noise from a mesh geometry. Houdini does this natively using either [Smooth 2.0 Geometry Node](https://www.sidefx.com/docs/houdini/nodes/sop/smooth.html) or [Attribute Blur Node](https://www.sidefx.com/docs/houdini/nodes/sop/attribblur.html).    
Laplacian smoothing means repositioning every vertex to the average position of its first-ring (meaning directly connected/adjacent) neighbours. [These slides I found online](https://graphics.stanford.edu/courses/cs468-12-spring/LectureSlides/06_smoothing.pdf) have some very simple diagrams to understand how this works.

### There can be 2 types of Laplacian smoothing:

#### a. Uniform Laplacian smoothing

The 'uniform' refers to uniform weights for all neighbours. This makes it not very accurate. It works well when triangulation in uniform (triangles of mesh are of equal size). The mesh is simply drawn inward to the geometric center (center of the shape of mesh) which is hardly the case for most geometries unless they are intended to be simple.

$$$$ b. Cotan Laplacian smoothing*

This is more accurate. Uses the cotangent matrix for weighting neighbours. This matrix keeps the information about shape and size of triangles intact. It is made of the cotangents of the angles opposite the shared angles of the vertices. So as you can see, the average of cotangents of angles $\alpha_{ij}$ and $\beta_{ij}$. We will come to the rest of the terms in the below formula used to geenrate the weight matrix later.

<img width="412" height="261" alt="image" src="https://github.com/user-attachments/assets/3de9d808-a7fd-4b2d-8b07-d42a6b25a119" />

In Houdini, the [Laplacian geometry node](https://www.sidefx.com/docs/houdini//nodes/sop/laplacian.html) calculates the Laplacian matrix for a geometry. You can use the Cotan Laplacian option to compute the cotangent laplacian matrix.

In this article, I'll be diving into the code and algorithm under the hood of some of the nodes mentioned above. I'll be showing the cotangent laplacian smoothing. Creating the uniform matrix is trivial. Below, I attempt to explain this in a simple manner. If you have some background in maths, do try to follow along!

## How To Implement Algorithm (Intuition for pseudocode) - Simplified

1. Create a mesh geometry.
2. Create cotangent matrix. (Formula from [https://ddg.math.uni-goettingen.de/pub/convergence_cotan.pdf](https://ddg.math.uni-goettingen.de/pub/convergence_cotan.pdf) and images drawn by me using [https://excalidraw.com/](https://excalidraw.com/))

_A snapshot from the above paper_
<img width="905" height="494" alt="Screenshot 2026-07-07 at 1 15 03 PM" src="https://github.com/user-attachments/assets/ed08dd88-4a9f-4a35-a782-4767f8d2ee12" />

<img width="496" height="431" alt="Screenshot 2026-07-07 at 1 30 55 PM" src="https://github.com/user-attachments/assets/37cd7635-7614-4c33-af51-ded136435a1c" />

Non-diagonal entry is $\delta_{pq}=\frac{1}{2}\left(\cot(\alpha_{pq}) + \cot(\beta_{pq})\right)$, where p is the shared vertex of the 2 adjoining triangles, and q is the 2nd endpoint of the line primitive that these 2 triangles share. In the diagram I drew above, $q1$ is q. 
This will give you the weight for the red vertex in the diagram. The matrix is defined for the data at each of the vertices. So, the size of the cotangent matrix will be nxn (where n is number of vertices). 1 row and 1 column for each vertex.

**Note:** This means that the amtrix is quite sparse. Each vertex has 1 row and 1 column, but the cotangent weight is only calculated for its immediate neighbours. Each vertex only has a few fixed number of neighbours. So it is beneficial to store only the non-zero connections when we impleemnt this using code.

Diagonal entry $\delta_{pp}=-\sum_i\delta_{pq_i}
=-\left[\frac{1}{2}\left(\cot(\alpha_{pq_1})+\cot(\beta_{pq_1})\right)
+\frac{1}{2}\left(\cot(\alpha_{pq_2})+\cot(\beta_{pq_2})\right)
+\cdots+
\frac{1}{2}\left(\cot(\alpha_{pq_n})+\cot(\beta_{pq_n})\right)\right].$

3. Update vertex data using $$p_{\text{new}} = p_{\text{old}} + \lambda \* C \* (q_i - p)$$

4. Now, increasing iterations will result in your geometry shrinking, maybe even into a single point eventually. To prevent this, a technique that inflates the geometry back up between iterations of smoothing is used. This is called Taubin smoothing. It is as simple as using a negative coefficient in place of $\lambda$. Shrink with $\lambda>0$ and inflate with $\mu<0$.
5.  Important additional considerations (eg: normalization, parallel execution) to actually run this algorithm practically are detailed in the code breakdown blog, as this current blog is aimed at giving a big picture view of laplacian mesh smoothing.

Done!

## Coding In VEX Houdini - With Breakdown+Documentation of Code
[See my code files here↗]()

# Sources ♥️
1. [https://graphics.stanford.edu/courses/cs468-12-spring/LectureSlides/06_smoothing.pdf](https://graphics.stanford.edu/courses/cs468-12-spring/LectureSlides/06_smoothing.pdf)
2. [https://www.sidefx.com/docs/houdini/nodes/sop/smooth.html](https://www.sidefx.com/docs/houdini/nodes/sop/smooth.html)
3. [https://www.sidefx.com/docs/houdini/nodes/sop/attribblur.html](https://www.sidefx.com/docs/houdini/nodes/sop/attribblur.html)
4. [https://igl.ethz.ch/projects/Laplacian-mesh-processing/Laplacian-mesh-optimization/lmo.pdf](https://igl.ethz.ch/projects/Laplacian-mesh-processing/Laplacian-mesh-optimization/lmo.pdf)
5. [https://rodolphe-vaillant.fr/entry/70/laplacian-smoothing-c-code-to-smooth-a-mesh](https://rodolphe-vaillant.fr/entry/70/laplacian-smoothing-c-code-to-smooth-a-mesh)
6. [https://houdinigubbins.wordpress.com/tag/laplacian/](https://houdinigubbins.wordpress.com/tag/laplacian/)
7. [https://discourse.mcneel.com/t/cotangent-matrix/178421](https://discourse.mcneel.com/t/cotangent-matrix/178421)
8. [https://ddg.math.uni-goettingen.de/pub/convergence_cotan.pdf](https://ddg.math.uni-goettingen.de/pub/convergence_cotan.pdf)
