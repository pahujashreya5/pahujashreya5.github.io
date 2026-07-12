# This is implementation+breakdown+documentation of VEX code for Horn's Quaternions Method For Absolute Orientation Problem 
My code is adapted from [this C++ code by Albert Huang](https://docs.ros.org/en/indigo/api/libfovis/html/absolute__orientation__horn_8hpp_source.html) 💜

[Paper: https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf](https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf) 

## Introduction

I went over the theory and intuition for the algorithm in [my previous blog](optimal-shape-matching-two-methods.md). In this one, I will implement the algorithm in VEX Houdini. For this, I studied the C++ code linked above. It was very helpful to understand usage of the Eigen library. I will also be explaining how I translated it to VEX and why I made certain choices. Let's start!

## Understanding The C++ Code

## Code in VEX Houdini

## The Node Setup

## My VEX Code
absolute orientation using quaternions  
<img width="600" height="576" alt="Screenshot 2026-07-10 at 10 26 19 PM" src="https://github.com/user-attachments/assets/9ad51259-ec60-490e-b20d-bfae472882da" />


```
// retrieve data from upstream node
int npts1=npoints(0);
int npts2=npoints(1);
vector p1=detail(0,"centroid1");
vector p2=detail(1,"centroid2");
vector point_cloud1[]=detail(0,"point_cloud1");
vector point_cloud2[]=detail(1,"point_cloud2");

// subtract the point cloud matrix containing all the points with the position of 
// ->centroid of each of the clouds.
// create matrix r by subtracting the centroid. 
vector r1[]=point_cloud1; // firsrt copy to r
// subtract centroid columnwise
for(int col=0; col<npts1; col++) {
    r1[col]-=p1; // 3x1 - 3xn -> 3xn matrix (vector of arrays in vex)
}
v[]@r1=r1; // 3xn

vector r2[]=point_cloud2;
for(int col=0; col<npts2; col++) {
    r2[col]-=p2; 
}
v[]@r2=r2; // 3xn

// compute matrix m=dot(r1,transpose(r2))
// iterate over columns
float row_x1[]; 
for(int col=0; col<npts1; col++) {
    float ele_x1=r1[col].x;
    append(row_x1,ele_x1);
}
float row_y1[];
for(int col=0; col<npts1; col++) {
    float ele_y1=r1[col].y;
    append(row_y1,ele_y1);
}
float row_z1[];
for(int col=0; col<npts1; col++) {
    float ele_z1=r1[col].z;
    append(row_z1,ele_z1);
}

float row_x2[]; 
for(int col=0; col<npts2; col++) {
    float ele_x2=r2[col].x;
    append(row_x2,ele_x2);
}
float row_y2[];
for(int col=0; col<npts2; col++) {
    float ele_y2=r2[col].y;
    append(row_y2,ele_y2);
}
float row_z2[];
for(int col=0; col<npts2; col++) {
    float ele_z2=r2[col].z;
    append(row_z2,ele_z2);
}

// elements of matrix m
float S_xx=dot(row_x1,row_x2);
float S_xy=dot(row_x1,row_y2);
float S_xz=dot(row_x1,row_z2);
float S_yx=dot(row_y1,row_x2);
float S_yy=dot(row_y1,row_y2);
float S_yz=dot(row_y1,row_z2);
float S_zx=dot(row_z1,row_x2);
float S_zy=dot(row_z1,row_y2);
float S_zz=dot(row_z1,row_z2);

// creating elements of matrix mat_n size 3x3
float a_00=S_xx+S_yy+S_zz;
float a_01=S_yz-S_zy;
float a_02=S_zx-S_xz;
float a_03=S_xy-S_yx;
float a_11=S_xx-S_yy-S_zz;
float a_12=S_xy+S_yx;
float a_13=S_zx+S_xz;
float a_22=-S_xx+S_yy-S_zz;
float a_23=S_yz+S_zy;
float a_33=-S_xx-S_yy+S_zz;

matrix mat_n=set(a_00,a_01,a_02,a_03,a_01,a_11,a_12,a_13,a_02,a_12,a_22,a_23,a_03,a_13,a_23,a_33);
4@mat_n=mat_n;
// get eigenvalues and eigenvectors
// since matrix mat_n is real (it is symmetric) so we can use selfadjoineigensolver



```

# Sources ♥️

1. [https://libeigen.gitlab.io/eigen/docs-nightly/group__TopicLinearAlgebraDecompositions.html](https://libeigen.gitlab.io/eigen/docs-nightly/group__TopicLinearAlgebraDecompositions.html)
2. [https://docs.ros.org/en/indigo/api/libfovis/html/absolute__orientation__horn_8hpp_source.html](https://docs.ros.org/en/indigo/api/libfovis/html/absolute__orientation__horn_8hpp_source.html)
3. [https://www.sidefx.com/forum/topic/85882/#post-405489](https://www.sidefx.com/forum/topic/85882/#post-405489)




