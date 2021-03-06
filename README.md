# libLBFGS

[![Build Status](https://travis-ci.org/lostella/libLBFGS.svg)](https://travis-ci.org/lostella/libLBFGS)

`libLBFGS` is a small piece of C code computing
limited-memory BFGS (also known as L-BFGS) directions for minimizing smooth functions. Contains a MATLAB interface [briefly documented here](#matlab-interface).

The library implements the necessary functions to allocate a FIFO buffer for some pairs of vectors to be stored;
to cyclically push and pull pairs `(s, y)` of vectors into the buffer;
to compute matrix vector products with the resulting approximate inverse Hessian matrix.

For more information about the L-BFGS method, see:

* J. Nocedal, "Updating quasi-Newton matrices with limited storage," in *Mathematics of Computation*, vol. 35, issue 151, pp. 773-782 (1980). [Available here](http://www.ams.org/journals/mcom/1980-35-151/S0025-5718-1980-0572855-7/)
* D. Liu, J. Nocedal, "On the limited memory BFGS method for large scale optimization," in *Mathematical Programming*, vol. 45, pp. 503-528 (1989). [Available here](http://link.springer.com/article/10.1007%2FBF01589116)

## MATLAB interface

To setup `libLBFGS` for MATLAB, navigate to `libLBFGS/matlab` from your MATLAB command line and

```matlab
>> setup_libLBFGS
>> test_libLBFGS
```

Now you can use `libLBFGS`. The easiest way is through the `LBFGS` class:

```matlab
H = LBFGS(n, mem); % create an operator of dimension n and memory mem
H.push(s, y);      % push a pair (s, y) into the buffer and update the operator
d = -(H*grad);     % multiply H by (minus) grad, to obtain a search direction
```

## C interface (Linux/Unix/MacOS X)

To compile the library simply navigate to its directory and type `make`.
This will produce `libLBFGS.o`, that you will need to statically link to your project.
The following routines are exported in `libLBFGS.h`:

* `libLBFGS_buffer * libLBFGS_init (int n, int mem);`
    Initialize a libLBFGS_buffer structure.

    * Parameters

     `int n`: dimension of the vectors to handle

     `int mem`: memory of the buffer

    * Return value

     `libLBFGS_buffer * b`: pointer to the buffer structure

* `int libLBFGS_push (libLBFGS_buffer * b, double * s, double * y);`
    Pushes a new pair (s, y) into the buffer, and discards the oldest one if necessary.

    * Parameters

     `libLBFGS_buffer * b`: pointer to the buffer structure

     `double * s`: double vector of length n, containing the difference between to points x and x'

     `double * y`: double vector of length n, containing the difference between the two gradients at x and x'

    * Return value

     `int status`: some integer status code

* `void libLBFGS_matvec (libLBFGS_buffer * b, double H0, double * g, double * d);`

    Performs a MATVEC operation between the inverse Hessian approximation stored so far and a given vector.

    * Parameters

     `libLBFGS_buffer * b`: pointer to the buffer structure

     `double H0`: value of the diagonal initial Hessian approximation to use; can be any positive value, a good choice for this is suggested in Nocedal, Wright, "Numerical Optimization", eq. (7.20)

     `double * g`: double vector of length n, containing the vector to multiply with the inverse Hessian approximation

     `double * d`: double vector of length n, where to store the result of the matrix-vector product

* `void libLBFGS_reset (libLBFGS_buffer * b);`

    Resets the memory of the given buffer structure.

    * Parameters

     `libLBFGS_buffer * b`: pointer to the buffer structure

* `void libLBFGS_clear (libLBFGS_buffer * b);`

    Deallocates the given buffer structure: all the referenced memory locations will be cleared, including the buffer itself, and the pointer will no longer be usable.

    * Parameters

     `libLBFGS_buffer * b`: pointer to the buffer structure
