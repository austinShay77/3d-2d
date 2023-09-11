My submission is programmed in python 3.8.10 on the drexel tux linux servers.

My source code is in CG_hw4.py, but I put my code in an executable file call CG_hw4 that can be called
as needed in the requirements.

For this assignment, I decided to split my code up into different files for less scrolling.

My first change is in fileIO.py.

    lines 46 - 59
    read_smf() -    Reads in smf format and sets it into a data structure my code can handle.

The next few major changes were in transforms.py.

    lines 31 - 199
    In here is where I create the matrices needed to create both the Npar and Nper matrices using the
    methods shown in class. 

    lines 60 - 124
    Here I apply the normalized matrix to all of the points in the image. After that, depending on which 
    type of normalization, I modify how the homogenous coordinates get projected into 2d. I return them in 
    a fashion that was similar to previous assignments  so there is less code to change when clipping and
    mapping to the viewport.

I also created a helper function on lines 4 - 12 to normalize the matrices needed to get the rotated matrix.