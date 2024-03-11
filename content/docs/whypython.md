+++
title = 'Need for Speed: Accelerating Python with sprinkles of C++'
date = 2024-03-11T04:42:21+05:30
draft = false
+++

## Get out of the python
For building performant system python is not really a good language. There are
many lacking factors in python which makes it difficult to use for production
level systems.


## Drawbacks of python
1. Python is not a compiled language.
2. Python is not a statically typed language.
3. Python is not very parallelisable cause of GIL (Global Interpreter Lock).

## Pros of python
1. Python is very easy to learn.
2. Python is good for prototyping and small scale projects.
3. Python has really good libraries for data science and machine learning like 
numpy, pandas, scikit-learn, tensorflow, keras etc.

## How does performant libraries like numpy, pandas, scikit-learn etc. work?
These libraries are written in C/C++ and wrapped with python. So, the core
computation is done in C/C++ and the python is used for the high level
abstraction and user interface.
This gives the best of both worlds. The performance of C/C++ and the ease of
python. :)

## How to write code in C/C++ and use it in python?
One of the most popular library for this is `pybind11`. Pybind11 is a lightweight
header only library which makes it easy to expose c++ code to python.
pybind11 is also used in popular libraries like `pytorch` and `pandas`.




## How to use pybind11?
We will use torch extension library to use c++ code in python. The torch extension
library is a part of pytorch and it makes it easy to use c++ code in python.
Internally it uses pybind11 to expose c++ code to python. We will use this library
to make it simpler to get started as we don't have to write a setup.py file and
makefile to compile the c++ code into shared library. All of this process is abstracted
away by the torch extension library.


### Lets import the required libraries


```python
import torch
import random
### load inline function makes it easier to import c++ code into python. 
### It abstracts away the need to write a setup.py file and 
### the need to compile the code separately.
from torch.utils.cpp_extension import load_inline
```

### Lets implement naive solution for merge sort in python.



```python
# Implement merge sort in python    

def merge_sort_py(arr):
    if len(arr) > 1:
        mid = len(arr) // 2
        L = arr[:mid]
        R = arr[mid:]

        merge_sort_py(L)
        merge_sort_py(R)

        i = j = k = 0

        while i < len(L) and j < len(R):
            if L[i] < R[j]:
                arr[k] = L[i]
                i += 1
            else:
                arr[k] = R[j]
                j += 1
            k += 1

        while i < len(L):
            arr[k] = L[i]
            i += 1
            k += 1

        while j < len(R):
            arr[k] = R[j]
            j += 1
            k += 1

    return arr
```

### Lets write the c++ code for merge sort and use it in python using torch extension library.



```python
cpp_source = '''

#include <vector>

std::vector<int> merge_sort_cpp(std::vector<int> arr) {
    if (arr.size() > 1) {
        int mid = arr.size() / 2;
        std::vector<int> L(arr.begin(), arr.begin() + mid);
        std::vector<int> R(arr.begin() + mid, arr.end());

        L = merge_sort_cpp(L);
        R = merge_sort_cpp(R);

        int i = 0;
        int j = 0;
        int k = 0;

        while (i < L.size() && j < R.size()) {
            if (L[i] < R[j]) {
                arr[k] = L[i];
                i++;
            } else {
                arr[k] = R[j];
                j++;
            }
            k++;
        }

        while (i < L.size()) {
            arr[k] = L[i];
            i++;
            k++;
        }

        while (j < R.size()) {
            arr[k] = R[j];
            j++;
            k++;
        }
    }
    
    return arr;
}


'''
```

### Lets use the `load_inline` function to use the c++ code in python. This will compile the c++ code and use it in python as a module. In this case the module name is `merge_sort_cpp`.

```python
merge_sort_cpp = load_inline(
    name = 'merge_sort_cpp',
    cpp_sources = [cpp_source],
    functions = ['merge_sort_cpp'],
    verbose= True,
    build_directory='.'
)
```

### Lets test the c++ code with the python code. Notice how List[int] got converted to std::vector<int> and the return type got converted to List[int].

```python
test_data = [5,1,3,2,4]
result = merge_sort_cpp.merge_sort_cpp(test_data)
assert result == sorted(merge_sort_py(test_data)) == [1,2,3,4,5]
```

Lets do performance comparison between the python and c++ code.
For x axis we will use the size of the array and for y axis we will use the time
taken to sort the array.


```python
import matplotlib.pyplot as plt
import numpy as np
import time


test_data_sizes = [10**i for i in range(2, 9)]
test_data_legend = [i for i in range(2, 9)]
python_runtimes = []
for size in test_data_sizes:
    test_input = random.sample(range(10**9), size)
    start_time = time.time()
    merge_sort_py(test_input)
    python_runtimes.append(time.time() - start_time)

    

cpp_runtimes = []
for size in test_data_sizes:
    test_input = random.sample(range(10**9), size)
    start_time = time.time()
    merge_sort_cpp.merge_sort_cpp(test_input)
    cpp_runtimes.append(time.time() - start_time)


plt.plot(test_data_legend, python_runtimes, marker='o', label='Python')
plt.plot(test_data_legend, cpp_runtimes, marker='o', label='C++')
plt.xlabel('Input Size 10^')
plt.ylabel('Runtime (seconds)')
plt.title('Merge Sort Runtime')
plt.legend()
plt.show()

```

![plot](/docs/f00816e7cc168cd244791548e80bae416cba803b.png "### Title")
    

As we can see from the plot, the c++ code is much faster than the python code.
In the next blog post we will see how to use pybind11 to write c++ code and use
it in python. We will also see how to circumvent the GIL and write parallel
code in python using c++.