+++
title = 'Accelerating Python with sprinkles of C++'
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
There are many ways to do this. The most common way is to use the python's
`ctypes` library. But, this is not the best way to do this. The best way to do
this is to use the `pybind11` library. This library is very easy to use and
provides a very good interface to write C++ code and use it in python.

pybind11 is also used in popular libraries like `pytorch` and `pandas`.




## How to use pybind11?
Lets start with torch extensions library. This library is used to write custom
layers and functions in pytorch. This library is written in C++ and uses
pybind11 to use it in python.


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


```python
merge_sort_cpp = load_inline(
    name = 'merge_sort_cpp',
    cpp_sources = [cpp_source],
    functions = ['merge_sort_cpp'],
    verbose= True,
    build_directory='.'
)
```


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
    

