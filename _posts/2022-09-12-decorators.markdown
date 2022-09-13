---
layout: post
title:  "First Class Functions, Closures, and Decorators"
date:   2022-09-12 23:28:05 -0700
categories: jekyll update
---

# Higher Order Functions and Decorators

## First-Class Functions

In some languages, functions are limited compared to other data types. For example, in C, you can put an array inside an array, or a struct inside a struct, but you can't define a function inside a function.

```c
int outer_square(int x) {
    int square(int y) {
        return y * y;
    }
    return square(x);
}

int main() {
    return outer_square(3);
}
```

Compiled with clang, this reports:

```
demo.c:2:23: error: function definition is not allowed here
    int square(int y) {
                      ^
```

However, other languages, like Python and Javascript, have what are called "First Class Functions". This means that functions are not "second class" citizens like they would be in C, but instead get "first class" treatment; they have all the powers of other data types like integers and lists.

For example, in Python, the above code is perfectly legal:

```python
def outer_square(x):
    def square(y):
        return y * y

    return square(x)

print(outer_square(3))  # Prints 9
```

Anything you could do with an integer or list, you can also do with a function. For example, you can put a function in a list! (Note that `print` without the parentheses is a reference to the print function, while `print()` *with* parentheses *calls* the function)

```python
my_list = [0,1,2,print]  # Note that we don't include the parentheses here
my_list[3]("hello")      # This is the same as print("hello")
```

You can return a function from another function:

```python
def get_the_square_function():
    def square(y):
        return y * y

    return square  # Again, note the lack of parentheses. I'm referring to the function, not calling it.

i = get_the_square_function()(3)  # get_the_square_function returns a function, which we immediately call
print(i)  # Prints 9
```

And you can pass a function as an argument to another function. Functions that take other functions as arguments, or return functions themselves, are sometimes called "Higher Order Functions":

```python
def square(x):
    return x * x

def apply(function, argument):
    return function(argument)

print(apply(square, 3))  # Prints 9
```

This comes up all the time, especially in Javascript. It's very common to pass one function to another as a "callback"; when the first function finishes running, it executes the callback instead of returning a result. This allows you to coordinate tasks that can run asynchronously and can finish in any order, or it can turn your code into a nightmarish spaghetti mess, affectionately dubbed Callback Hell. 

## Closures

When you define a function inside another function, scopes can get a bit confusing. Code in a function can access variables from a higher scope, like the global scope:

```python
x = 5                  # x is in global scope
def do_some_math(y):   # y is in local scope
    return x + y       # But x can be pulled in from the scope above

print(do_some_math(3)) # Prints 8
```

This means that code in a nested function can access variables in any enclosing scopes, including any outer functions:

```python
x = 5                  
def do_some_math(y):
    z = 1
    def inner_function():
        return x + y + z  # x is in global scope, y in local scope, and z in "nonlocal" scope.
    
    return inner_function()

print(do_some_math(3))  # Prints 5+3+1 == 9
```

This might not seem too noteworthy when the outer scope just holds a constant, but this has some very interesting implications. What if the inner function gets returned, then called from a different scope from where it was defined? Surprisingly, this actually works!

```python
def add_by_a_number(x):  # x is defined outside the scope of "add_x"
    def add_x(y):
        return x + y  # x comes from a nonlocal scope, but gets "enclosed" in this inner scope

    return add_x

add_three = add_by_a_number(3)  # In the "add_three" function, 'x' has a value of 3.
print(add_three(5))  # Prints 8. 

add_five = add_by_a_number(5)  # In the "add_five" function, 'x' has a value of 5.
print(add_five(1))   # Prints 6. 

print(add_three(1))  # This still prints 4. This value of 'x' was NOT overriden when we created add_five!

print(x)  # This throws "NameError: name 'x' is not defined"
```

The first time I saw this, it kind of broke my brain. This isn't an inevitable property of the universe; it's a design decision. But it's a design decision made by every mainstream language that supports higher order functions, so it's worth coming to terms with, even if it seems impossible or absurd.

What we've just done is called a Closure, because we "enclose" a variable from one scope into another (although I prefer the term "embed", personally). Once we've created our `add_three` function, it "remembers" that `x` has a value of 3, even though that information was defined in an outer scope that no longer exists!

This can get very, very weird:

```python
def entangled_functions():
    count = 0

    def get_count():
        return count
    
    def increment_count():
        nonlocal count     # This is a weird Python specific hack for this situation.
                           # By default, count = ... would define a new variable called count,
                           # rather than modify the existing variable in a higher scope.
                           # This is a consequence of Python having the same syntax for declaring
                           # a new variable and modifying one that already exists.
                           # "nonlocal count" conveys "When I say count, I mean the nonlocal version"
        count = count + 1
    
    return get_count, increment_count

get_count, increment_count = entangled_functions()

print(get_count())  # 0
print(get_count())  # 0
increment_count()
print(get_count())  # 1
print(get_count())  # 1
increment_count()
print(get_count())  # 2

get_count_2, increment_count_2 = entangled_functions()

print(get_count())    # 2
print(get_count_2())  # 0
increment_count_2()
print(get_count())    # 2
print(get_count_2())  # 1
```

In this case, `increment_count()` affects the behavior of `get_count()` by changing the variable `count` in the enclosing scope. But once the `get_count` and `increment_count` functions have been created, that enclosing scope no longer exists! The `count` variable has been enclosed/embedded inside *both* functions! If you create different `get_count_2` and `increment_count_2` functions by calling the higher order `entangled_functions()` again, these new ones refer to their own *completely independent* version of `count`, from a completely separate enclosing scope, which *also* no longer exists! 🤯

Computers are crazy, man.

## Decorators

To reiterate, since Javascript and Python have First Class Functions, you can treat functions like any other datatype. One of the less interesting consequences of this is that you can assign a function to a variable.

```python
x = print
x("hello")  # prints "hello"
```

If you have a Higher Order Function that takes and returns a function, you can "wrap" one function in another. This wrapping function is called a Decorator. Here's an example of a Decorator that does nothing:

```python
def dummy_wrapper(function):
    return function

x = dummy_wrapper(print)
x("hello")  # prints "hello"
```

More realistically, you can use this technique to "augment" a function, injecting new capabilities after the function has been defined. For example, this wrapper takes an original_function and creates a new, augmented version which always casts its input to uppercase.

```python
def uppercase_wrapper(original_function):
    def new_function(x):
        original_function(x.upper())  # Even though original_function is defined in a higher scope,
                                      # it will be "enclosed" or "embedded" in the new_function.
    return new_function

new_print = uppercase_wrapper(print)
new_print("hello")  # prints "HELLO"
```

That said, this version has a problem. If you try to call `new_print("first", "second")`, it will fail, because we've defined the new_function to only take one argument! In general, if you plan to wrap functions, you usually want to define the arguments as broadly as possible. In Javascript, you would do this with `...args`. In Python, you would do this with `*args` for positional arguments and `**kwargs` for keyword arguments.

There are a few canonical examples where you might want to wrap a function. If you think a function is slow, you might wrap it with a timer to see how long it takes:

```python
import time

def time_me(function):
    def inner_function(*args, **kwargs):  # This allows maximum flexibility for any arguments
        start = time.time()
        function(*args, **kwargs)         # All arguments are passed to the original function
        end = time.time()
        print("Time elapsed: ", end-start)
    
    return inner_function

def slow_square(x):
    time.sleep(3)
    return x * x

square = time_me(slow_square)
square(3)  # Returns 9, and prints something like "Time elapsed: 3.005"
```

You might want to add some logging:

```python
def log_me(function):
    def inner_function(*args, **kwargs):  
        log_something()
        function(*args, **kwargs)         
    
    return inner_function
```

And, maybe as part of that logging, you might want to track how often a function is called:

```python
def count_me(function):
    count = 0
    def inner_function(*args, **kwargs):  
        nonlocal count
        count = count + 1
        print("Function call #", count)
        function(*args, **kwargs)         
    
    return inner_function

def square(x):
    return x * x

square = count_me(square)
prints = count_me(print)

square(3)  # prints "Function call # 1"
square(4)  # prints "Function call # 2"

prints("hello")  # prints "Function call #1" and "hello"

square(5)  # Function call # 3
```

In Python, this is extremely common. It's the idiomatic way to add functionality to a function, and it's so heavily encouraged that it gets a special syntax. Instead of `square = count_me(square)`, you can write this:

```python
def count_me(function):
    counts = 0
    def inner_function(*args, **kwargs):  
        nonlocal counts
        counts = counts + 1
        print("Function call #", counts)
        function(*args, **kwargs)         
    
    return inner_function

@count_me
def square(x):
    return x * x

square(1)  # prints "Function Call #1"
```

That `@count_me` syntax means "Replace `square` with `count_me(square)`". It's a small bit of syntactic sugar meant to subtly nudge you toward this pattern, which is built into the language on a pretty deep level. For example, if you've ever defined a class method that *doesn't* refer to an instance of the class, you've probably used the built-in decorator `@staticmethod`.

Javascript doesn't have the same syntactic sugar, but it still offers robust support for this technique:

```javascript
function loggingDecorator(function_to_wrap) {
    let count = 0;

    let innerFunction = function(...args) {
        count = count + 1;  // Unlike Python, Javascript knows what you mean, since it has
                            // a different syntax for declaring a variable.
        console.log("Function call #", count);
        function_to_wrap(...args);
    }

    return innerFunction
}

function square(x) {
    return x * x;
}

function doNothing() {
    return;
}

square = loggingDecorator(square);
doNothing = loggingDecorator(doNothing);

square(5);    # Logs "Function call # 1"
square(5);    # Logs "Function call # 2"

doNothing();  # Logs "Function Call #1"

square(5);    # Logs "Function call # 3"
```

## Conclusion

In conclusion, Computer Science is a land of contrasts, and Laura should use this knowledge to get a better job 🙂
