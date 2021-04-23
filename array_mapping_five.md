# Array mapping in five scripting languages
#### Mr. George L. Malone
#### 23<sup>rd</sup> of April, 2021


## Setup and Background

Let's say you have a set of data stored in an array, such as this:

```
[1.6, 3.9, 4.2, 8.0, 0.8, 2.7]
```

and you want to perform a certain operation for every element of the array and
return the elementwise result.  For a simple operation such as multiplication,
the result can fairly easily be achieved using a map.  More complex operations
can also use a map, but other methods, such as reducers, might be more
applicable in such a circumstance.  In this case, the example operation is to
multiply each element by 3.

Firstly, the scripting languages that will be demonstrated are *R*, Python,
Ruby, JavaScript, and Julia.  It is likely that *R* and Python are known to
many who may be reading this, whereas Ruby and Julia are perhaps less
well-known (which is a shame, really!).  Of JavaScript, I do not know, but it
is also unlikely that many here (at work) know it.

Secondly, both compact and "long-format" approaches will be demonstrated.
Compact approaches include in-line, lambda, or anonymous functions.  Long
format is an external, full function definition which is then provided as a
callback in the map operation.


## *R*

Mapping operations in *R* are achieved using the `sapply` function, specifying
the object to iterate over, and the function to call at each iteration.  The
resulting object, in this case, is a vector, as the input is a vector and the
input has no name attribute.

In compact form, this can be achieved using an in-line anonymous function
within the `sapply` call, where the operative function is specified as the
second parameter.

```r
test_array <- c(1.6, 3.9, 4.2, 8.0, 0.8, 2.7)

result <- sapply(test_array, function(x) x * 3)
```

In long form, the function is defined prior to being given as a variable
reference in the `sapply` call.  The function definition itself can be an
in-line definition (as inside the above `sapply` call), but the full format is
as follows.  Note that the full definition requires an explicit return
statement.

```r
multiply_by_3 <- function(x) {
  return(x * 3)
}

test_array <- c(1.6, 3.9, 4.2, 8.0, 0.8, 2.7)

result <- sapply(test_array, multiply_by_3)
```


## Python

In Python, this operation looks a little more complex, because the result from
calling map is a map object, which then needs to be coerced to a list.

The function definition for the compact version is a lambda expression, rather
than a clear function definition.

```python
test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

result = list(map(lambda x : x * 3, test_array))
```

In full format, the function is defined prior to the map call, and, according
to [PEP8][1], the indent level is four spaces and two blank lines are left
after the function definition.

```python
def multiply_by_3(x):
    return x * 3


test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

result = list(map(multiply_by_3, test_array))
```


## Ruby

In Ruby, these operations are also quite straightforward, but there are at
least two different notation styles, and Ruby is not as fond of return
statements inside a block.  Ruby returns the value of the last operation seen.
So, the compact form itself has two formats and technically no in-line function
definition at all, because of the block-based code structure.  I don't
understand the concept enough to provide appropriate guidance.

```ruby
test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

# Notation 1:  Multiline block
result = test_array.map do |x|
  x * 3
end

# Notation 2:  Inline block
result = test_array.map{|x| x * 3}
```

Long form makes it a bit easier to see where things fit in, but again, it is
not the common convention to use return statements in Ruby, including within a
function definition, so the following precludes the return statement.  You will
notice, however, that it would become a normal function definition by simply
prefixing `return ` to the operative line.

```ruby
def multiply_by_3(x)
  x * 3
end

test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

test_array.map{|x| multiply_by_3(x)}
```

But that hasn't really changed anything, has it?  The class for floating-point
numbers could be extended to include the desired functionality, thus the map
could simply call upon a functional attribute of a floating-point number.
Modifying the inherent methods associated with a type is the more canonical
Ruby approach.  Note the peculiar syntax -- the map call is now in round
brackets, rather than curly brackets (which denotes a block), and the ampersand
is used to denote the current element.  The method `multiply_by_3` is called
for each current element of the array in the map operation.

```ruby
class Float
  def multiply_by_3
    self * 3
  end
end

test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

test_array.map(&:multiply_by_3)
```

The result is identical, but achieved in a more proper manner, considering the
conventions of Ruby.


## JavaScript

Using JavaScript, anonymous functions or "full" function definitions can be
used, but this is otherwise much the same as previously.  Note that, if a code
block is used rather than an in-line definition, the return statement must be
explicit.  Note also that operation scope termination is noted by the affixed
semi-colon.

So, these are both short-form approaches.  Variables must be initially declared
using `let`, `const`, or `var` -- considering the limited scope, `let` is used
here for the test array, and `var` is used for the result such that the
variable definition can be overwritten.

```javascript
let test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7];

// Anonymous function with code block
var result = test_array.map(x => {
  return x * 3;
});

// Anonymous function with inline operation
var result = test_array.map(x => x * 3);
```

Long format is similar, but this time there are two little tricks.  Firstly,
JavaScript features something called hoisting, which means that the function
can be defined after its initial used, as long as the definition is fully
present within the scope of the script.  Secondly, because the variable is not
being overwritten this time, `let` is used to declare the results variable.

```javascript
let test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7];

let result = test_array.map(multiply_by_3)

function multiply_by_3(x) {
  return x * 3;
};
```


## Julia

Julia is the interesting one, as it can be deliberately made more complex in
its type declaration such that compilation and execution time is minimised.
However, it can also be written in quite a simple manner.  The outcome of both
simple and complex examples is identical, but the complex example will compile
(thus execute) faster than the simpler example.

To emphasise the considerable potential differences in the simple and complex
forms, the simple form will be represented by the compact form, and will not
include type declaration, whereas the complex form is shown in the long form
will make use of a high degree of type declaration.  Non-local variables cannot
be type-declared, so are left "bare" -- these are `test_array` and `result`, in
this case.

Compact first:

```julia
test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

result = map(x -> x * 3, test_array)
```

And long form.  The statement beginning with the double-colon is the type
declaration.  The `Union` statement restricts the scope of possible types while
keeping it open to multiple types, and here the input value `x` and the output
are both declared to be either `Int64` or `Float64`, that is, 64-bit integer or
64-bit floating-point number.

```julia
function multiply_by_3(x::Union{Int64,Float64})::Union{Int64,Float64}
  return x * 3
end

test_array = [1.6, 3.9, 4.2, 8.0, 0.8, 2.7]

result = map(multiply_by_3, test_array)
```


[1]:https://www.python.org/dev/peps/pep-0008/
