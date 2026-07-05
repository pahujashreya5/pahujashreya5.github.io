# This is my implementation + breakdown + documentation of spatial hashing using VEX
[See my code files↗]()

My work is adapted from the algorithm and C++ code explained by Ten Minute Physics  
[https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf](https://matthias-research.github.io/pages/tenMinutePhysics/11-hashing.pdf)  
[https://github.com/matthias-research/pages/blob/master/tenMinutePhysics/11-hashing.html](https://github.com/matthias-research/pages/blob/master/tenMinutePhysics/11-hashing.html)

## The Node Setup

<img width="538" height="339" alt="Screenshot 2026-07-05 at 6 01 46 PM" src="https://github.com/user-attachments/assets/942fbf72-0215-4ec4-963f-7583f7385b19" />

I started by inserting a grid and scattering points on it using scatter node. 

Make sure we have spacing (which is side length of each cell in the grid) equal to twice the sphere radius. This is the assumption made in our algorithm that enables us to run it for a bounded region of 9 cells (or 27 in 3D). We use 'paste relative reference' to control the size of grid cells and spheres from the radius slider easily (shown below).

## Coding The Algorithm + Breakdown Of Code

### _Step 1:_ Defining Variables, Attributes & Data Structures

I declared the variables that are common across all points in a detail wrangle. Let's name this wrangle global_vars.


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

Copy the Radius parameter and paste it into the grid Size channel. If you wish, you can align the corner of the grid with the world origin so that it is easier to follow along with a dry run on paper, which I did to make sure the code was not breaking (for this, just add half the size of the grid to both x and y coordinates of the center).

<img width="896" height="601" alt="Screenshot 2026-07-05 at 6 04 29 PM" src="https://github.com/user-attachments/assets/52030a45-6e1d-451f-95ca-15cea228eba4" />

Operations that are specific to each point should be done within a point wrangle. This way, these operations run for each point at the same time (multithreading) without having to explicitly define a loop. This also maintains easy to understand code and easy to access attributes.
I always try and maintain this separation. Otherwise, I get confused when coding the main algorithm because accessing upstream detail attributes and point attributes have different syntaxes.

_Without Hashing (bounded grid)_
This is the code in the 'bounded_grid' point wrangle

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

// 1. Store the points into the oned_array[]

relative_pos=floor(@P/spc); // this is set(xi,yi)
// v@relative_pos=relative_pos;
// find array_idx to which this point belongs
// remember we are in the XZ plane so we use relative_pos.z
// modulo with number of grid cells-1 because our vector (dict) indices are 0 to numx*numy-1
array_idx=int(relative_pos.x*nx+relative_pos.z)%(nx*ny); 
i@array_idx=array_idx; // save as point attribute




```
_With Hashing (Which Runs Faster)_
This is the code in the 'unbounded_grid' point wrangle

```
// attributes that are point specific are in pointwrangle
vector relative_pos;
int array_idx;
// retrieve from upstream
int nx=detail(0,"numx")-1;
int ny=detail(0,"numy")-1;
int num_cells=nx*ny;
float spc=detail(0,"spacing");

// 1. define hash function
int hash(int var1,var2, num_cells) {
    int hash_val=(var1*92837111)^(var2*283923481);
    return abs(hash_val)%num_cells;
}

// 2. Store the points into the oned_array[]

relative_pos=floor(@P/spc); // this is set(xi,yi)
v@relative_pos=relative_pos;
// find array_idx to which this point belongs
// we are in the XZ plane
array_idx=hash(relative_pos.x,relative_pos.z,num_cells); 
// put this point into the array_index we just calculated
i@array_idx=array_idx; // save as point attribute
```

### _Step 2:_ Store the point numbers into a 1D array & create dense array

Now, let's write the actual body of the algo within a detail wrangle called 'spatial_hashing'. We use this because we need to run over each of the points on by one using for or foreach loops. The comments are self-explanatory, but I am using a diagram to explain the flow of the code too.

```
// get variables declared upstream
// Maintaining global attributes-same for all points
// the 1d array
dict oned_array;
// an intermediary array to build the dense array
int intermediary_array[];
// the dense array. size of this is exactly equal to number of points so we manage to 
// ->save a lot of space by not working with a sparse array directly
int dense_array[];
// total number of cells
int num_cells=(i@numx-1)*(i@numy-1);

// 1. store point number into the array index calculated in point wrangle
for(int i=0; i<npoints(0); i++) { // loop over all points
    int pt_idx=point(0,"array_idx",i); // retreive from its array_idx point wrangle
    int curr[]=oned_array[itoa(pt_idx)]; // get the corresponding array from dictionary of arrays
    append(curr,i); // store this point number in this array
    oned_array[itoa(pt_idx)]=curr; // put the array back into the dictionary against the correct key
}
// set the 1d array as detail attribute (it is stored only once, not for each point) 
setdetailattrib(0,"oned_array",oned_array);

// 2. create intermediary_array

int acc_sum=0; // store a running/cumulative sum
for(int i=0; i<num_cells; i++) { // loop over the size of the 1D array
    string key=itoa(i); // convert the index to string because we need to use it as key for dict
    int curr[]=oned_array[key]; // get the array of this key into a temporary array
    acc_sum+=len(curr); // update the running sum with number of elements within the temporary array
    append(intermediary_array,acc_sum); // put the updated array back into the dict against its key
}
// store the intermediary_array as a detail attribute for further use
setdetailattrib(0,"intermediary_array",intermediary_array);

// 3. create dense array -> size=number of points
int j=0; // pointer/index for dense array
for(int i=0; i<num_cells; i++) { // loop over intermediary array
    int num_pts;// initialize a variable for number of points until current index of the intermediary_array
    if(i>0) num_pts=intermediary_array[i]-intermediary_array[max(i-1,0)]; // current number of 
    // ->points is the increment in the number of points at current index compared to 
    // ->previous index 
    else num_pts=intermediary_array[i]; // in case i-1 is negative we handle it separately
    if(num_pts==0) continue; // if there are no new points to put in the dense array yet, 
    // ->we don't need to run the rest of the operations
    string sidx=itoa(i); // convert index to string to use as key
    int curr[]=oned_array[sidx]; // explained above
    int curr_len=len(curr); // get number of points inside current array
    for(int idx=0; idx<curr_len; idx++) { // iterate over these points
        dense_array[j]=curr[idx]; // put these points next to each other in sequential order in the dense_Array
        j++; // increment the dense array index for the next loop iterations
    }
}
// store as detail attribute
setdetailattrib(0,"dense_array",dense_array);
```

### Dry Run

Now let's try a dry run using the values in my geometry spreadsheet. You can do this for your own values also!

#### Geometry Spreadsheet snapshots:
<img width="1353" height="405" alt="Screenshot 2026-07-05 at 9 06 25 PM" src="https://github.com/user-attachments/assets/807d1eef-0a0c-4b9e-bcce-1d7cec515ff0" />
<img width="1353" height="405" alt="Screenshot 2026-07-05 at 9 06 16 PM" src="https://github.com/user-attachments/assets/30dd47df-7a77-4290-95ee-859545362d61" />

#### Dry run diagram/notes:

Let's take point number 2. Its position is (1.10552,0,2.87059). The relative position comes out to be (1,0,3) according to the bounded_grid algo first.
Thus, its array_idx is 8.
Calculating similarly for all points, the 1D array looks like this:

<img width="848" height="104" alt="Screenshot 2026-07-05 at 9 16 37 PM" src="https://github.com/user-attachments/assets/7284741c-0fbf-4ff9-80b0-c4bbbfcb64e1" />

Then we create the intermediary array: {0, 1, 1, 1, 3, 3, 3, 4, 5, 5, 6, 6, 6, 7, 7, 9, 9, 10, 10, 10}
- At index 0 in the oned_array, there are 0 points. So, intermediary_array[0]=0.
- At index 1 in the oned_array, there is 1 point (@ptnum==8). To check this we see the length of the array at key "1". So, intermediary_array[1]=1.
- At index 2 in the oned_array, there are 0 points. To check this we see the length of the array at key "2". It is empty. So, intermediary_array[2]=1 again.
- The value is 1 until intermediary_array[3].
- At index 4 in the oned_array, there are 2 points. The length of array at key "4" of oned_array is 2 that's why 2 points. So intermediary_array[4]=3.
- Same until last intermediary_array is filled.

Now for the dense array: {8, 1, 9, 4, 2, 3, 5, 0, 7, 6}
- At index 0 in intermediary array, the value is 0. This means there are 0 points until now. So, dense_array[0]=0;
- At index 1 on intermediary_array, value is 1. 1-0=1, that means we have just now stored 1 point. The point's value can be retrieved from our oned_array dict. It is at key "1". Point number 8.
- At index 2 there are no new points stored since the value is still 1. We can skip until we find new points.
- At index 4, we see 3-1>0. There are 3-1=2 new points here. Check dict oned_array[4]. It is {1, 9}. So, store these adjacent to each other in the dense array. dense_array[4]=1, dense_array[5]=9. We need to make sure we are incrementing j correctly.
- Continue until end of intermediary_array.
