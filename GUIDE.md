***How to construct arrays?***
*For example, let's construct a 3-dimensional array with 2x4x3 entries...*
 - *...initialized as int():*
    > ```cpp
    > size_t shape[] = {3, 4, 2}; // 3x4x2 array
    > libnd::MDArray<int> i(shape, shape + 3); // 3x4x2 array of 0s
    > ```
 - *...initialized as 1:*
    > ```cpp
    > libnd::MDArray<int> j(shape, shape + 3, 1); // 3x4x2 array of 1s
    > ```
 - *...which remain un-initialized:*
    > ```cpp
    > libnd::MDArray<int> k(libnd::SkipInitialization, shape, shape + 3); // 3x4x2 array of uninitialized entries
    > ```
 - *...indexed in first coordinate major order:*
    > ```cpp
    > libnd::MDArray<int> l(shape, shape + 3, libnd::FirstMajorOrder); // 3x4x2 array of 0s indexed in first coordinate major order
    > ```

***How to access the entries of an array?***
*There are three ways to access the entries of an array:*
 - *By coordinates:*
    > ```cpp
    > size_t shape[] = {3, 4, 2}; // 3x4x2 array
    > libnd::MDArray<int> a(shape, shape + 3); // 3x4x2 array of 0s
    > a(0, 0, 0) = 1; // set entry at (0, 0, 0) to 1
    > a(2, 3, 1) = 2; // set entry at (2, 3, 1) to 2
    > ```
 - *By scalar indices:*
    > ```cpp
    > for (size_t j = 0; j < a.size(); ++j) // set all entries to their index
    > {
    >   a(j) = static_cast<int>(j); // set entry at j to j
    > }
    > ```
 - *By STL-compliant random access iterators:*
    > ```cpp
    > for (libnd::MDArray<int>::const_iterator it = a.begin(); it != a.end(); ++it) // iterate over all entries
    > {
    >   std::cout << *it << ' '; // print entry
    > }
    > std::cout << std::endl; // print newline
    > ```

***How to do arithmetic operations on arrays?***

*Simple arithmetics work just as you would expect.*
> ```cpp
> size_t shape[] = {3, 4, 2}; // 3x4x2 array
> libnd::MDArray<int> a(shape, shape + 3, 2); // 2x2x2 array of 2s
> libnd::MDArray<int> b(shape, shape + 3, 2); // 2x2x2 array of 2s
> libnd::MDArray<int> c; // uninitialized array
> ++a; // increment all entries by 1
> --a; // decrement all entries by 1
> a += 2; // add 2 to all entries
> a -= 2; // subtract 2 from all entries
> a /= 2; // divide all entries by 2
> a *= 2; // multiply all entries by 2
> c = a + b; // add all entries of a and b
> c = a - b; // subtract all entries of b from a
> c = a * b; // multiply all entries of a and b
> c = a / b; // divide all entries of a by b
> c = a * a + 2 * a * b + b * b; // polynomial
> ```

*Different types (e.g. int and float) can be combined in one arithmetic expression; the lib will promote the types according to the standard rules.*
> ```cpp
> size_t shape[] = {4}; // 4x1 array
> libnd::MDArray<int> a(shape, shape + 1, 1); // 4x1 array of 1s
> libnd::MDArray<float> b(shape, shape + 1, 1.1f); // 4x1 array of 1.1s
> libnd::MDArray<float> c = a + b; // 4x1 array of 2.1s
> std::cout << c.asString(); // print array
> ```

*The lib implements expression templates. The following expression is thus evaluated only for one entry at coordinate (2, 0, 0):*
> ```cpp
> size_t shape[] = {3, 4, 2}; // 3x4x2 array
> libnd::MDArray<int> a(shape, shape + 3, 2); // 2x2x2 array of 2s
> libnd::MDArray<float> b(shape, shape + 3, 3.1f); // 2x2x2 array of 3.1s
> float c = (a * a * a + 2 * a * a * b + 2 * a * b * b + b * b * b)(2, 0, 0); // the expression is evaluated only for one entry at coordinate (2, 0, 0)
> std::cout << c << std::endl; // print entry
> ```

***How to construct and transform views?***
*A view makes a contiguous interval in memory look as if it was an multi-dimensional array. In contrast to a multi-dimensional array which allocates and de-allocates memory via an STL-compliant allocator, a view provides an interface to a contiguous interval in memory that needs to be allocated by other means. One important use of view is to access sub-arrays.*
 1. *For example, let's start with a simple matrix:*
    > ```cpp
    > size_t shape[] = {15, 5}; // 15x5 array
    > libnd::MDArray<int> a(shape, shape + 2); // 15x5 array of 0s
    > for (size_t j = 0; j < a.size(); ++j) // set all entries to their index
    > {
    >   a(j) = 10 + static_cast<int>(j); // set entry at j to 10 + j
    > }
    > ```
 2. *...let us define a view to a 7x3 sub-matrix that starts at coordinates (5, 1):*
    > ```cpp
    > size_t base[] = {5, 1}; // base coordinates
    > shape[0] = 7; // set the first dimension of the shape to 7
    > shape[1] = 3; // set the second dimension of the shape to 3
    > libnd::View<int> b = a.view(base, shape); // construct a view
    > std::cout << a.asString(); // print array
    > std::cout << b.asString(); // print view
    > ```
 3. *Now, let us manipulate the sub-matrix (i.e. the data under the view) and print the original matrix.*
    > ```cpp
    > b = 10; // set all entries of the sub-matrix to 10
    > std::cout << a.asString(); // print array
    > ```

*A View can also be used to make data look differently. Here are two examples:*
 - *Transposition:*
    > ```cpp
    > libnd::View<int, true> c = a; // Construct a View. 'true' makes the data under the view constant, i.e., this particular view cannot be used to change the data.
    > std::cout << c.asString(); // print view
    > c.transpose(); // transpose the view
    > std::cout << c.asString(); // print view
    > ```
 - *Reshaping:*
    > ```cpp
    > libnd::View<int, true> d = a; // Construct a View. 'true' makes the data under the view constant, i.e., this particular view cannot be used to change the data.
    > size_t newShape[] = {5, 3, 5}; // new shape
    > d.reshape(newShape, newShape + 3); // reshape the view
    > std::cout << d.asString(); // print view
    > ```

*A more advanced use of views is to provide a convenient interface for the multi-dimensional arrays that are native to C:*
> ```cpp
> int e[3][4][2];
> for (size_t j = 0; j < 24; ++j)
> {
>   (**e)[j] = static_cast<int>(j);
> }
> size_t shape[] = {3, 4, 2};
> libnd::View<int> f(shape, shape + 3, **e, libnd::FirstMajorOrder, libnd::FirstMajorOrder);
> // the first libnd::FirstMajorOrder determines how coordinates
> // are mapped to memory. The second libnd::FirstMajorOrder
> // determines how scalar indices into the view are mapped to
> // coordinates in the view.
> // let's compare the element access by scalar indices...
> for (size_t j = 0; j < 24; ++j)
> {
>   std::cout << (**e)[j] << ", " << f(j) << std::endl;
> }
> // ...to the element access by coordinates
> for (size_t x = 0; x < shape[0]; ++x)
> {
>     for (size_t y = 0; y < shape[1]; ++y)
>     {
>         for (size_t z = 0; z < shape[2]; ++z)
>         {
>           std::cout << "e[" << x << "][" << y << "][" << z << "] = "
>                     << e[x][y][z] << ", "
>                     << "f(" << x << ", " << y << ", " << z << ") = "
>                     << f(x, y, z) << std::endl;
>         }
>     }
> }
> ```