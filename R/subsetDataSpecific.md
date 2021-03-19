# Subset a dataframe in R
## Specific example
#### Mr. George L. Malone
#### 19<sup>th</sup> of March, 2021


### Background

You have a dataframe that contains four main data -- patient ID, time to
diagnosis (in days), time of measurement since diagnosis (in days), and value
of a measurement (such as metabolite concentration in blood plasma).  You want
to look at two continuous variables, the time to diagnosis and the value of the
measurement, but only for time-points 0 and 7.  Both values must be present for
each patient under consideration.


### Tools

_R_ is used for this demo, using `base` operations (rather than Tidy).  The
main functionality used will be the `apply` family of functions.  Of course,
you will also need the dataframe, which might be read in to _R_ as in the
following code block.  I've used certain names for certain variables which will
likely differ in your dataset.  The patient ID, time to diagnosis, time of
sample collection after diagnosis, and value of the sample measurement are
represented by `idPatient`, `timeDiagnosis`, `timeCollection`, and `measure`,
respectively.

```R
data <- read.delim(
  "./data.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE
)

head(data)
```

```
idPatient  timeDiagnosis  timeCollection  measure
Pt00001    13             0               3.1416
Pt00002    76             28              1.2274
Pt00001    13             7               2.7231
Pt00003    19             7               1.9970
Pt00003    19             14              2.0213
Pt00002    76             21              1.3208
...        ...            ...             ...
```


### Approach

The first thing that might be done is reducing the data to the patients with
the right number of entries.  If you want each patient with time-points 0 and
7, this patient must have at least two entries in the dataframe.  So, you could
check how many rows the dataframe features for each unique patient ID.

Firstly, get the unique patient IDs.  If you need to remove `NA` values or
other undesirable values, a `which` to select the rows of the original dataset
matching a certain Boolean condition could be applied prior to this operation.
This particular operation is fairly straightforward thanks to the "built-in"
`unique` function, which (naturally) returns all unique elements of an array.

```R
idUnique <- unique(data$idPatient)
```

Then, an operation needs to be performed for each unique patient ID.  A `for`
loop could be used, but `for` loops in _R_ are quite slow.  An alternative is
using an `apply` function -- in this case, `sapply`.  This function receives an
array to iterate over and performs a certain operation for each element in the
array.  The operation to perform is defined by a function.

There are two approaches to take here.  One approach is to define an anonymous
function inside the call to `sapply` -- e.g. `sapply(arr, function(a) { ... })`
-- or to define a named function which is then used inside the call -- e.g.
`sapply(arr, someFunction)`.  The latter approach will be taken here, because
this may make it easier to debug the operations should something go awry, but
also because this separates certain concerns within the code, such as which
operations take in a single item and which take in an array or list.

But what do we need?  Well, we need all rows in the dataframe for certain
patients.  Those patients must meet the condition that, for each patient, there
are at least two measurements taken, and these measurements must be at
time-points 0 and 7.


### `sapply`

The function takes three arguments -- the array (or other iterable) to loop
over, the function to use for each value of the array, and any additional
(named) arguments to use in the looped function call.  The last component is
not strictly necessary in my personal experience, but my experiences with code
in general have been somewhat unorthodox.  However, it will be utilised here.

A twee example of this is now demonstrated.  The `sapply` takes an array of
numbers and, for each value, returns the multiplication of the value.  The
function definition is as follows:

```R
testFunction <- function(input, number = 2) {
  return(input * number)
}
```

So, when this is run with the array `c(1, 2, 3, 4, 5)`, the values 2, 4, 6, 8,
and 10 should be returned:

```R
testArray <- c(1, 2, 3, 4, 5)

sapply(testArray, testFunction)  #=>  2  4  6  8 10
```

But what about other numbers?  `testFunction` can take `number` as an argument,
so we can pass this in to the `sapply` operation:

```R
sapply(testArray, testFunction, number = 3)  #=>  3  6  9 12 15
```

**But now, a caveat:**
This value is static throughout all iterations.  If you want to use a different
value for each iteration, this gets a little more complex.  Such a case will
not be covered here, but an example may be given if desired.

Anyway, that's a quick overview of `sapply`.


### Building the function

For now, ignore the fact that the input to `sapply` is an array of names.
Taking a single name under consideration, there is a specific set of checks
that must be performed to include the patient in the final set.  So, the
function to use inside the `sapply` will be constructed _with only one entry in
mind_, but should also be able to accommodate the potential problems of all,
such as `NULL` values.

Here's a start:

```R
useFunc <- function(id, data) {
  subset <- data[which(data$idPatient == id), ]
  return(nrow(subset))
}
```

This will tell you the number of rows in the data where the variable
`idPatient` is exactly equal to `id`.  That's not quite what is needed, but
it's on the right track.  The target patient IDs must have at least two entries
in the data, and the return value should be the patient IDs matching such a
criterion.  So, the number of rows can be used within the function to decide
whether the ID should be included or not:

```R
useFunc <- function(id, data) {
  subset <- data[which(data$idPatient === id), ]
  # If there aren't at least two rows, the ID is not included
  if (nrow(subset) < 2) return(NA)
  # At this point, there must be enough rows, so the ID is returned
  return(id)
}
```

Good, but now there can be `NA` values in the data.  This will need to be
accounted for in the output of the `sapply`.  But there's also a little more
that needs to be done -- the time-points.  This could be achieved by further
subsetting the data according to the values required:

```R
useFunc <- function(id, data) {
  subset0 <- data[which(data$idPatient === id), ]
  # Subset the subset to rows with time-point 0 or 7 only
  subset1 <- subset0[which(subset0$timeCollection %in% c(0, 7)), ]
  # Check if there are enough rows
  if (nrow(subset1) < 2) return(NA)
  return(id)
}
```

So, this will reduce the data to the entries relating to the current ID, then
reduce this further to the entries in the subset with the time of collection
being 0 days or 7 days.  Thus, if the number of rows is less than two, then the
times of collection are not 0 or 7, so the ID is not applicable.  Otherwise,
the ID is returned.  There are two problems here.

Firstly, the `timeCollection` variable may be a string, in which case checking
the value of `timeCollection` against an integer will fail.  The variable could
be coerced to an integer prior to using this function, or the function could
coerce the value.  It is much more efficient to coerce the values in the data,
rather than within the `sapply`, so I would recommend this.  The alternative is
to change the values in the check to reflect the expected data type.

The second problem is uniqueness -- the input data must contain _exactly one
entry each_ for time-points 0 and 7.  If this is not so, then there could be
two entries for 0 days and one for 7 days for a specific ID, which is
technically erroneous.  You must be certain that the function is correctly
working with the actual structure of your data.

Also, there are still `NA` values, but this could be handled after running the
`sapply`.


### Operate

Now, run the `sapply` and see what happens.  Without a specific example, it is
difficult for me to progress beyond this point, as beyond here requires
contextual interpretation of results and, most likely, debugging.  But using
the function looks like this:

```R
# Apply over all unique IDs
result <- sapply(idUnique, useFunc, data = data)

# Remove NA values
result <- result[which(!is.na(result))]
```

What is returned by this is an array of unique patient IDs for which there are
at least two rows in the data, and `timeCollection` is either 0 or 7.  Thus, to
get the rows of the data for these IDs only, subset the original dataframe
based on whether the value of `idPatient` is in `result`:

```R
newData <- data[which(data$idPatient %in% result), ]
```
