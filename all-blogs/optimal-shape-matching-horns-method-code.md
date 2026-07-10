# This is implementation+breakdown+documentation of VEX code for Horn's Quaternions Method For Absolute Orientation Problem 
My code is adapted from [this C++ code by Albert Huang](https://docs.ros.org/en/indigo/api/libfovis/html/absolute__orientation__horn_8hpp_source.html) 💜

[Paper: https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf](https://web.stanford.edu/class/cs273/refs/Absolute-OPT.pdf) 

## Introduction

I went over the theory and intuition for the algorithm in [my previous blog](optimal-shape-matching-two-methods.md). In this one, I will implement the algorithm in VEX Houdini. For this, I studied the C++ code linked above. It was very helpful to understand usage of the Eigen library. I will also be explaining how I translated it to VEX and why I made certain choices. Let's start!

## Understanding The C++ Code





