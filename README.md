# FiniteDifferenceFormula

This Julia package provides a general finite difference formula generator and a tool
for teaching/learning the finite difference method. It generates finite difference
formulas for derivatives of various orders by using Taylor series expansions of a
function at evenly spaced points. It also gives the truncation error of a formula
in the big-O notation. We can use it to generate new formulas in addition to
verification of known ones.

We may play with this package when teaching/learning numerical computing, especially
the finite difference method, and explore the distribution, symmetry, and beauty in
the coefficients of the formulas. By changing decimal places, we can also see how
rounding errors affect a result.

Beware, though formulas are mathematically correct, they may not be numerically useful.
This is true especially when we derive formulas for a derivative of higher order. For
example, run compute(9,-5:5), provided by this package, to generate a 10-point
central formula for the 9-th derivative. The formula is mathematically correct, but it
can hardly be put into use for numerical computing without, if possible, rewriting it
in a special way. Similarly, the more points are used, the more precise a formula
is mathematically. However, due to rounding errors, this may not be true numerically.

To run the code, you need the Julia programming language (https://julialang.org/), a
wonderful and amazing computing platform.

## How to install the package

In Julia REPL, execute the following two commands in order.

1. import Pkg
1. Pkg.add("FiniteDifferenceFormula")

## The package exports twelve functions

- compute, search, searchforward, searchbackward, formula, truncationerror
- decimalplaces, activatejuliafunction, verifyformula, taylor, printtaylor
- _set_default_max_num_of_taylor_terms

### functions compute, search, searchforward, and searchbackward take the same arguments (n, points, printformulaq = false)

#### Input

```
            n:  the n-th order derivative to be found
       points: in the format of a range, start : stop, or a vector
printformulaq: print the computed formula or not
```

|   points     |   The points/nodes to be used                  |
|   ---------- | ---------------------------------------------- |
|    0:2       |   x[i], x[i+1], x[i+2]                         |
|   -3:2       |   x[i-3], x[i-2], x[i-1], x[i], x[i+1], x[i+2] |
|   [1 0 1 -1] |   x[i-1], x[i], x[i+1]                         |

A vector can be like [1, 0, 2] or [1 0 2]. It will be rearranged so that elements are ordered
from lowest to highest with duplicate ones removed.

#### Output

Each function returns a tuple, (n, points, [k[1], k[2], ..., k[stop-start+1]], m) where n,
points, k[:] and m are described below. With the information, you may generate formulas for
any programming language of your choice.

While 'compute' may fail to find a formula using the points, others try to find one, if possible,
by using fewer points in different ways. (See the docstring of each function.)

The algorithm uses the linear combination of f(x[i+j]), j ∈ points, a given list of points,
to eliminate f(x[i]), f'(x[i]), f''(x[i]), ..., so that the first nonzero term of the Taylor
series of the linear combination is f^(n)(x[i]):

```Julia
    k[1]*f(x[i+points[1]]) + k[2]*f(x[i+points[2]]) + ... + k[len]*f(x[i+points[len]]) = m*f^(n)(x[i]) + ..., m > 0
```

where len = length(points). It is this equation that gives the formula for computing f^(n)(x[i])
and the truncation error in the big-O notation as well.

### function formula( )

The function generates and lists

1. k[1]*f(x[i+points[1]]) + k[2]*f(x[i+points[2]]) + ... + k[len]*f(x[i+points[len]])
       = m*f^(n)(x[i]) + ..., m > 0

1. The formula for f^(n)(x[i]), including estimation of accuracy in the big-O notation.

1. Julia function for f^(n)(x[i]).

### function truncationerror( )

The function returns a tuple, (n, "O(h^n)"), the truncation error of the newly computed finite
difference formula in the big-O notation.

### function decimalplaces( ) or decimalplaces(n)

Without an argument, the function returns current decimal places. With argument n, it sets the
decimal places to be n for generating Julia function(s) for formulas if n is a nonnegative
integer. It returns the (new) default decimal places. Without/before calling the function, 16
decimal places are used by default.

This function can only affect Julia functions with the suffix "d" such as f1stderiv2ptcentrald.
See function activatejuliafunction().

### function activatejuliafunction( )

Call this function to activate the Julia function(s) for the newly computed finite
difference formula. For example, after compute(1, -1:1), it activates the
following Julia functions.

```Julia
f1stderiv2ptcentrale(f, x, i, h)  = ( -f(x[i-1]) + f(x[i+1]) ) / (2 * h)
f1stderiv2ptcentrale1(f, x, i, h) = ( -1/2 * f(x[i-1]) + 1/2 * f(x[i+1]) ) / h
f1stderiv2ptcentrald(f, x, i, h)  = ( -0.5000 * f(x[i-1]) + 0.5000 * f(x[i+1]) ) / h
```
The suffixes 'e' and 'd' stand for 'exact' and 'decimal', respectively. No suffix? It is "exact".
After activating the function(s), we can evaluate right away in the present Julia REPL session. E.g.,

```Julia
FiniteDifferenceFormula.f1stderiv2ptcentrale(sin, 0:0.01:pi, 3, 0.01)
```
Below is the output of activatejuliafunction(). It gives us the first chance to examine the usability
of the computed or tested formula.

```Julia
import FiniteDifferenceFormula as fd
f, x, i, h = sin, 0:0.01:10, 501, 0.01
fd.f1stderiv2ptcentrale(f, x, i, h)   # result: 0.2836574577837647, relative error = 0.00166666%
fd.f1stderiv2ptcentrale1(f, x, i, h)  # result: 0.2836574577837647, relative error = 0.00166666%
fd.f1stderiv2ptcentrald(f, x, i, h)   # result: 0.2836574577837647, relative error = 0.00166666%
                                      # cp:     0.2836621854632262
```

### function activatejuliafunction(n::Int, points, k, m::Int)

It allows users to load a formula from some source to test and see if it is correct. If it is valid,
its truncation error in the big-O notation can be determined. Furthermore, if the input data is not
for a valid formula, it tries also to find one, if possible, using n and points.

Here, n is the order of a derivative, points are a list, k is a list of the corresponding
coefficients of a formula, and m is the coefficient of the term f^(n)(x[i]) in the linear
combination of f(x[i+j]), where j ∈ points. In general, m is the coefficient of h^n in the
denominator of a formula. For example,

```Julia
import FiniteDifferenceFormula as fd
fd.activatejuliafunction(2, [-1 0 2 3 6], [12 21 2 -3 -9], -12)
fd.truncationerror()
fd.activatejuliafunction(4, 0:4, [2//5 -8//5 12//5 -8//3 2//5], 5)
fd.activatejuliafunction(4, [0, 1, 2, 3, 4], [2/5 -8/5 12/5 -8/3 2/5], 5)
fd.activatejuliafunction(2, [-1 2 0 2 3 6], [1.257 21.16 2.01 -3.123 -9.5], -12)
``` 
### verifyformula(n::Int, points, k, m::Int)

It is exactly the same as function activatejuliafunction(n::Int, points, k, m::Int). The name
is self-explanatory.

### function taylor(j, n = 10)

The function returns the coefficients of the first n terms of the Taylor series of f(x[i+j])
about x[i].

### function printtaylor(j, n = 10)

The function prints the first n terms of the Taylor series of f(x[i+j]) about x[i].

### function printtaylor(coefficients_of_taylor_series, n = 10)

The function prints the first n nonzero terms of a Taylor series of which the coefficients are
provided.

### function _set_default_max_num_of_taylor_terms(n) or _set_default_max_num_of_taylor_terms()

The function sets to n the default maximum number of terms of Taylor series. Usually, users
never need to know its existence. (If no change is made, the default value is 30.) The value
affects the behaviors of functions 'taylor' and 'printtaylor' only. When you need more than
30 terms of a Taylor series, the value should be set. To have m nonzero terms, n should
certainly be larger than or equal to m, say, n = m + 8. The function returns the present default value.

## Examples

```Julia
import FiniteDifferenceFormula as fd
fd.compute(1, 0:2, true)             # find, generate, and print "3"-point forward formula for f'(x[i])
fd.compute(2, -3:0, true)            # find, generate, and print "4"-point backward formula for f''(x[i])
fd.compute(3, -9:9)                  # find "19"-point central formula for f'''(x[i])
fd.decimalplaces(6)                  # use 6 decimal places to generate Julia functions of computed formulas
fd.compute(2, [-3 -2 1 2 7])         # find formula for f''(x[i]) using points x[i+j], j = -3, -2, 1, 2, and 7
fd.compute(1,-230:230)               # find "461"-point central formula for f'(x[i]). does it exist? run the code!
fd.formula()                         # generate and print the formula computed last time you called compute(...)
fd.truncationerror()                 # print and return the truncation error of the newly computed formula
fd.printtaylor(-2, 5)                # print the first 5 terms of the Taylor series of f(x[i-2]) about x[i]
coefs = 2*fd.taylor(0) - 5*fd.taylor(1) + 4*fd.taylor(2) - fd.taylor(5);
fd.printtaylor(coefs, 7)             # print the 1st 7 nonzero terms of the Taylor series of
                                     # 2f(x[i]) - 5f(x[i+1]) + 4f(x[i+2]) - f(x[i+5])
fd.activatejuliafunction()           # activate Julia function(s) of the newly computed formula in present REPL session
fd.verifyformula(1, 2:3, [-4, 5], 6) # verify if f'(x[i]) = (-4f(x[i+2] + 5f(x[i+3)) / (6h) is a valid formula
```
