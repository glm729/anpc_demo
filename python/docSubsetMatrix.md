# Python -- Subsetting a matrix
#### Mr. George L. Malone
#### 15<sup>th</sup> of January, 2021

NOTE:  These operations use base Python (3), and I'm not particularly
experienced with Python, so any recommendations or alternative approaches that
could be provided are welcome.

Assuming your matrix is an array of arrays, e.g.:

```python
m = [
    [ 1,  2,  3,  4,  5],
    [ 6,  7,  8,  9, 10],
    [11, 12, 13, 14, 15],
    [16, 17, 18, 19, 20]
]
```

Then, to subset the matrix, you could use the following function:

```python
def subset(matrix, rows, cols):
    result = []
    for i in rows:
        rowi = []
        for j in cols:
            rowi.append(matrix[i][j])
        result.append(rowi)
    return result
```

Or alternatively:

```python
def subset(matrix, rows, cols):
    result = []
    for i in rows:
        result.append([])
        for j in cols:
            result[i].append(matrix[i][j])
    return result
```

This should allow the use of any iterable (if appropriate).  For example, if
you wanted all of the rows of the matrix but only columns 1 and 2 (recalling
that Python is zero-indexed), then you could try this:

```python
subset(m, range(0, len(m)), [1, 2])
```

This should return the following object:

```python
[
    [ 2,  3],
    [ 7,  8],
    [12, 13],
    [17, 18]
]
```

If you wanted all columns in one row, this has a caveat.  The assumption for
this is that the array of arrays passed in has an identical number of items per
array -- that is, the rows are all of identical length.  Given that this is so,
you could use the length of the array in index 0 to get the number of columns:

```python
subset(m, [1, 2], range(0, len(m[0])))
```

This should return the following:

```python
[
    [ 6,  7,  8,  9, 10],
    [11, 12, 13, 14, 15]
]
```

Ranges work well as an iterator because you can even specify the step size, but
that's another application.  In the case of "normal" use, they're used as such:

```python
for i in range(0, 5):
    print(i)
```

This produces the output:

```
>>> for i in range(0, 5):
...     print(i)
...
0
1
2
3
4
>>>
```

Thus, to iterate from row 0 to the maximum row, start the range at 0 and
iterate to the maximum length (exclusive, because of zero-indexing):

```python
range(0, len(matrix))
```

Or, to iterate over the columns, assuming that the rows are all of identical
length:

```python
range(0, len(matrix[0]))
```
