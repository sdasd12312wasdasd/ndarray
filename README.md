# Introduction


This project is an experimental, header-only implementation of a numpy-like `ndarray` template for pure C++14. It should be comparable to (but smaller and more modern than) [Boost.MultiArray](https://www.boost.org/doc/libs/1_68_0/libs/multi_array/doc/index.html).


If you are interested in a featureful and professionally maintained numerical package for C++, you should check out [xtensor](https://github.com/QuantStack/xtensor).


If you just wanted a lightweight `ndarray` container and not much else, this project might interest you.


The code should be transparent enough that you can modify it without much trouble. Pull requests welcome!


# Overview

`ndarray` objects use the same memory model as the numpy's `ndarray`. The array itself is a lightweight stack object containing a `std::shared_ptr` to the allocated memory block, which may be in use by multiple arrays. Const-correctness is ensured by having slicing operations on const arrays return a deep-copy of the data.


```C++
  // Basic usage:

  ndarray<3> A(100, 200, 10);
  ndarray<2> B = A[0]; // B.shape() == {200, 10}; B.shares(A);
  ndarray<1> C = B[0]; // C.shape() == {10}; C.shares(B);
  ndarray<0> D = C[0]; // D.shape() == {}; D.shares(C);
  double d = D; // rank-0 arrays cast to underlying scalar type
  double e = A[0][0][0]; // d == e (slow)
  double f = A(0, 0, 0); // e == f (fast)
```


```C++
  // Creating a 1D array from an initializer list

  auto A = ndarray<1>{0, 1, 2, 3};
  A(0) = 3.0;
  A(1) = 2.0;
```


```C++
  // Multi-dimensional selections

  auto A = ndarray<3>(100, 200, 10);
  auto B = A.select(0, std::make_tuple(100, 150), 0); // A.rank == 1; A.shares(B);
```


```C++
  // STL-compatible iteration

  auto x = 0.0;
  auto A = ndarray<3>(100, 200, 10);

  for (auto &a : A[50])
  {
    a = x += 1.0;
  }
  auto vector_data = std::vector<double>(A[50].begin(), A[50].end());
```


```C++
  // Respects const-correctness

  // If A is non-const, then
  {
    ndarray<1> A(100); // A[0].shares(A);
    A(0) = 1.0; // OK
    ndarray<1> B = A; // B.is(A);
  }

  // whereas if A is const,
  {
    const ndarray<1> A(100); // ! A[0].shares(A);
    // A(0) = 1.0; // compile error
    ndarray<1> B = A; // ! B.is(A);
  }
```


```C++
  // Basic arithmetic expressions

  auto A = ndarray<1>::arange(10);
  auto B = ndarray<1>::ones(10);
  auto C = (A + B) / 2.0;
```


# Priority To-Do items:
- [x] Generalize scalar data type from double
- [x] Basic arithmetic operations
- [x] Allow for skips along ndarray axes
- [ ] Support for boolean mask-arrays, comparison operators >=, <=, etc.
- [ ] Relative indexing (negative counts backwards from end)
- [ ] Array transpose (and general axis permutation)
- [x] Factories: zeros, ones, arange
- [ ] Custom allocators (allow e.g. numpy interoperability or user memory pool)
- [x] Binary serialization
- [x] Enable/disable use of C++ exceptions at compile time
- [ ] Enable/disable bounds-checking at compile time
