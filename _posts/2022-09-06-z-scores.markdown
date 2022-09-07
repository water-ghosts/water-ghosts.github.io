---
layout: post
title:  "1) Mean, Standard Deviation, and Z Scores"
date:   2022-09-05 18:14:05 -0700
categories: jekyll update
---

# Prerequisites

This series assumes a basic familiarity with programming. I'll avoid doing anything complicated, but you should at least be familiar with functions and loops, and be able to follow a simple Python program [Note: I'll include some examples and maybe a refresher]. The target audience of this series will ideally find basic programming syntax at least as easy to follow as formal math syntax, and preferably easier. Over these chapters, we're going to build up a Python statistics library. This will not be production quality by any measure, and we're going to strongly focus on clarity over performance.

This series also assumes a rough working knowledge of calculus. Don't worry if you don't know the formulas or formal definitions, but I will assume you're comfortable with the general idea of derivatives and integrals and why they're useful [Note: I'll write a refresher/overview chapter].

# Chapter 1: Mean, Standard Deviation, and Z Scores

## Averages

Let's say you've collected some data and you'd like to summarize it. Sometimes you can do this losslessly; `[1, 1, 1, ..., 1]` could be described as "N ones" and `[1, 1, 1,... , 1, 2]` is "N ones and 1 two". But more often, the data has enough complexity (or entropy) that you can't capture all its complexity without reproducing the whole dataset itself. In these cases, we need some standard, reproducible ways to compress our data into something easy to understand.

In most cases, we manage this compression with two numbers: One which represents the average (or central) data point, and one which describes how far away most data points are from that average, on average. Together, these tell us the shape and location of our data. 

There are multiple ways to measure the average, but in practice "average" almost always means... Mean. The Mean is almost certainly a familiar concept, but it's worth briefly revisiting its definition and emphasizing why it's preferred over other measures of centrality, specifically the median and mode.

### Mode

The mode of a dataset is the number in the dataset which appears the most often. This doesn't have a clean mathematical notation, but in Python we'd write it like this:

```python
def mode(data):
    values = {}
    most_common_value = None
    
    for d in data:
        if d not in values:
            values[d] = 0
        values[d] += 1
        if most_common_value is None:
            most_common_value = d
        elif values[d] > values[most_common_value]:
            most_common_value = d
    
    return most_common_value
```

There's an intuitive logic to Mode, since it answers the question "If we picked a data point at random, what's the most likely value we'll get?", but that's the only thing to recommend it. It's surprisingly awkward to calculate (although this could be streamlined with Python's `collections.counter`, which feels tailor-made for this exact situation), and it's often extremely misleading. For example, the series `[1, 1, 996, 997, 998, 999...]` has a mode of 1, perhaps its least useful or interesting feature.

Mode can also be extremely sensitive to small variations, since it's an all-or-nothing measurement. The series `[1, 1, 999.999, 1000]` can see its Mode increase three orders of magnitude if you nudge the third data point. If you're working with imprecise floating point math, the Mode might depend on whether this rounds to 1000 or 1000.0000000000001. Alternately, the Mode might not change at all, since it's unclear how this definition breaks ties. Meanwhile, if your dataset is `[1, 1, 2]`, then nothing you do to that third data point will ever change the Mode. Whether it's 2, 2 million, or negative 2 trillion, the Mode will remain fixed at 1.

In the real world, Mode mostly offers countless ways to lie with statistics. If you hear someone say "the modal outcome is X", exercise the same caution you would when a salesman extols a used car. Clearly, we need something better.

### Median

If you sort your data from smallest to largest, the Median is the value in the middle, or the value halfway between the middle two.

```python
def median(data):
    data = sorted(data)
    n = len(data)
    i = int(n / 2)
    if n % 2 == 0:
        return (data[i-1] + data[i]) / 2
    else:
        return data[i]
```

Median is generally more useful than Mode, as it has a clear, unambiguous, unique definition that reliably reflects a dataset's "center". However, it still doesn't solve one of Mode's larger problems; it's still unresponsive to changes. Take the dataset `[0, 4, 4, x]`. If x is 4 or higher, the median will always be 4, whether x is 4, 4.1, or 4 quintillion. Likewise, the median will be 2 `((0 + 4) / 2)` if x is anywhere from 0 to negative infinity.

In fairness, this is sometimes this is what we want. The median is especially useful for measuring income and wealth statistics. Elon Musk and Jeff Bezos gain and lose billions daily just from random variation in stock prices, and your town's wealth will change drastically if one of them moves in. If we want to look at average wealth among all Americans, it's better to choose a metric by which these changes have no effect.

But in most cases, this isn't a feature; it's a bug. We *want* to detect changes in our inputs. If we're solving a constrained optimization problem, we'd like to find a minimum or maximum point. This is typically done by finding where a function's derivative is 0, but this median function doesn't have a clean derivative; it's non-continuous and relies on a hard-to-compose sort. If we want to target a particular median, we can't even tell if we're on the right track. Going from `[0, 4, 4, 100]` to `[0, 4, 4, 10]` brings us closer to a median of 2, but we'd never be able to tell that from trends or intermediate results.

[Note: I'll probably include some graphs of derivatives to make this clearer]

### Mean

We want a measure of centrality that's easy to calculate, optimize, and incorporate into other formulas, and this brings us to the familiar, ubiquitous Mean. The Mean is simply the sum of all data points divided by the number of data points. 

```python
def mean(data):
    return sum(data) / len(data)
```

Statisticians love to make things seem more complicated than they are, so your introductory textbook will probably juggle hairline distinctions between µ (the population mean), x-bar (the sample mean), and E(x) (the expected value; technically a slightly broader concept, but in practice almost always interchangeable), but the core concept really is just the version you learned in fourth grade. In general, µ (mu) seems to be the most common representation, and Python is conveniently flexible enough to accept that as a variable name, so I'll mostly stick to that.

If our dataset is `[a, b, c, d]`, then increasing `d` by some value (call it `x`) simply increases the mean by `x/4`. You can think of this as a partial derivative, or bypass the calculus and simply work through the arithmetic.

`(a+b+c+d+x)/4 - (a+b+c+d)/4`

`(a+b+c+d)/4 + x/4 - (a+b+c+d)/4`

`(a+b+c+d)/4 - (a+b+c+d)/4 + x/4`

`0 + x/4`

`x/4`

[Note: I wasn't able to get LaTeX notation to work in Github Pages. I will eventually just make my own blog and do this properly]

Since we don't need to worry about counting or sorting, we can do this kind of analysis without knowing anything about the individual data points. The relationship between inputs and output is always linear, and it's trivial to check our work or compose this with other formulas. These properties make Mean by far the most useful of our three averages. 

## Variance

Once we know how our data is centered, we also want to describe how spread out it is. `[-1, 0, 1]` and `[-1500, 0, 1500]` have the same mean (0), but the second is much more spread out than the first.

As with centrality, there are multiple ways to approach this. We could report on the total range between the smallest and largest value, or break the data into quartiles and focus on the range covered by the middle two. But both of these approaches, like Median and Mode, are insensitive to changes and extremes. What if we want another measure that reflects every input and possible change?

We could measure the total deviation of all data points from the mean, but this wouldn't mean much on its own. A total temperature deviation of 30 degrees C means something very different if your time horizon is three hours vs three years! And we can't solve this by taking the average, since the average deviation from the mean will always be 0 (don't worry if this isn't immediately obvious; we'll demonstrate why it is in a few minutes). 

The simplest solution is to take the absolute value of each deviation, so `[-1500, 0, 1500]` would have a total deviation of 3000 and an average absolute deviation of 1000 `(3000 / 3)`. This is by no means a bad approach, but the absolute value function is still a bit messier than we'd like; it's mostly continuous, but it doesn't have a well-defined derivative at its minimum point. We could probably make things work despite that problem, but instead we've traditionally chosen to *square* each deviation from the mean. This is called variance, and it gives our `[-1500, 0, 1500]` data an average squared deviation of 1,500,000 `(4,500,000 / 3)` .

```python
def variance(data):
    µ = mean(data)
    return mean([(d - µ)**2 for d in data])
```

Note that Variance is never less than 0; since every deviation from the mean is squared, no component will ever be negative, so the average must be nonnegative as well (nonreal datasets are far outside our scope).
Also note that this squaring causes outliers (extremely atypical data points) to have extreme effects. This turns out to be a feature, not a bug, and might be intuitive than you expect. For example, 99.9% of lottery players have the same outcome (they lose), but playing the lottery is still an extremely high variance activity; a few players make millions!

### Wait, shouldn't the denominator be...?

Some introductory texts emphasize a distinction between two types of variance, one that divides by `n` and another by `n-1`. Personally, I think this is a bad approach. These textbooks rarely give you full context for this often-irrelevant distinction, and off-by-one errors are infamously one of the ~~two~~ three hardest problems there are. Forcing you to juggle this distinction before you've even mastered the basics is just asking for anxiety, confusion, and failure.

If you need to pass a stats exam, want to understand the distinction between VAR.P and VAR.S in Excel, or just want to read a rant, I go into more detail in this appendix [Note: This hasn't made it to a sharable draft yet].

But if you just want a good mental model for Variance, it's the average squared distance from the mean. And because it's an average, the denominator is `n` :)

## Standard Deviation

Variance still has one major flaw: its units are a nightmare to interpret. Let's say you measured a few people's heights in centimeters to get `[175, 175, 185, 185]`. The mean is 180 centimeters, but the variance is... 25 cm^2? Is the distribution of heights somehow an area? You might think back to calculus and try to interpret this as the area under a curve, but imagine if we measured temperatures, not heights. If our temperatures were `[10, 20, 30, 40, 50, 60, 70]`, how would you interpret the variance of 400 Degrees Celsius Squared?

To make this more tractable, we usually take the square root of variance, which gives us the units we started with. This square root is called Standard Deviation (often shortened to stddev, stdev, or s.d. and represented by σ (sigma), which somewhat confusingly leaves variance as σ^2). This gives us a sensible 5 cm for the heights and 20 Degrees Celsius for the temperatures.

```python
def sqrt(x):  # In reality you'd import sqrt from the math library, but let's define it ourselves just for fun
    return x ** 0.5

def stdev(data):
    return sqrt(variance(data))
```

Just like Variance, Standard Deviation is never negative, although it can be 0 if all data points are the same. For example, every number in the dataset `[1, 1, 1]` is 0 away from the mean, and `sqrt(0^2 + 0^2 + 0^2) = sqrt(0) = 0`. Taking the square root somewhat softens the impact of outliers, but standard deviation will still typically be larger than average absolute value. `[0, 0, 0, 200]` has an average absolute deviation of 75, but a standard deviation of 86.6.

Why yes, it *is* confusing that Variance and Standard Deviation have completely different names despite measuring the same thing. Statistics is shrouded in nonsensical terminology, making straightforward concepts seem endlessly opaque. Don't be surprised if you hear people get confused and use "variance" and "standard deviation" interchangeably; unless you're actively crunching numbers, it's rarely worth being pedantic.

## Z Scores - A Universal Unit Translator

We now have a measure of centrality and a measure of spread. Together, these do a good job summarizing our data, and we've made sure the units aren't a mess. Let's see how they work together in practice.

Suppose we take two measures of people's heights: `[175, 175, 185, 185]` and `[1.75, 1.75, 1.85, 1.85]`. These represent the same data in centimeters and meters respectively. The first dataset has a mean of 180 cm and a standard deviation of 5 cm, while the second has a mean of 1.8 m and a standard deviation of 0.05 m. In both cases, we can divide the standard deviation by the mean to get the uninspiringly named Coefficient of Variation, which works out to around 0.278 either way. Note that this number is unitless, since the units in the numerator and denominator cancel out.

Unfortunately, this doesn't generalize as well as we'd like. Suppose a European measures the temperatures `[10, 20, 30, 40, 50, 60, 70]`, while an American measures `[50, 68, 86, 104, 122, 140, 158]`. Like the height examples, these represent the same observations in Celsius and Fahrenheit respectively, with the conversion factor `F = C * 1.8 + 32`. But while the Celsius temperatures have a mean of 40 and a standard deviation of 20, the Fahrenheit temperatures have a mean of 104 and a standard deviation of 36. Despite measuring the exact same temperatures, these datasets have different Coefficients (0.5 vs 0.346); even our unitless ratio depends on what units we use. The real solution is to settle on one temperature scale, but while we try to convince the Americans to change their dumb units, they're busy making first contact. The aliens beam down their own corresponding measurements in Degrees Quarblax, with a mean of 8.1 * 10^700 and a standard deviation of 3. Is the situation hopeless? Can we ever bridge the gap between our kinds?

Thankfully, we have a universal translator. We can convert these datasets to Z-Scores, a term that's ironically lost in translation itself; the Z stands for Zentrum, German for Center, as in the Central (or Normal) Distribution. I'd like to call the process of converting numbers to their respective Z Scores "normalizing", but that name is overloaded, and typically means scaling numbers to between 0 and 1. Instead, since these scores are sometimes called Standard Scores, we can call this process Standardizing. Alternately, if it's a better mnemonic, we can call it Centralizing (Zentralizing?)

Whatever we call it, the idea is to express each data point as the number of standard deviations it is from the mean. The Celsius temperatures `[10, 20, 30, 40, 50, 60, 70]` have a mean of 40 and a standard deviation of 20, which means the first temperature of `10` is -30 degrees from the mean. 30 is equal to 1.5 standard deviations, so this data point's Z Score is -1.5. The fully standardized form of these temperatures works out to `[-1.5, -1.0, -0.5, 0.0, 0.5, 1.0, 1.5]`. 

Meanwhile, since the Fahrenheit temperatures `[50, 68, 86, 104, 122, 140, 158]` have a mean of 104 and a standard deviation of 36, the first temperature of `50` is -54 from the mean, and 54 is... 1.5 standard deviations. The fully standardized form of these temperatures is *also* `[-1.5, -1.0, -0.5, 0.0, 0.5, 1.0, 1.5]`! And without even knowing the conversion ratio between Celsius, Fahrenheit, and Quarblax, I can rest assured that the standardized Quarblax measurements are *also* `[-1.5, -1.0, -0.5, 0.0, 0.5, 1.0, 1.5]`.

```python
def z_scores(data):
    µ = mean(data)
    σ = stdev(data)
    return [(d - µ) / σ for d in data]
```

This process not only simplify conversions, it lets us compare wildly different measurements by expressing everything as unitless, relative distances from means. Is an 80 degree temperature in Antartica more unusual than a -10 degree temperature in LA? If we know the mean and standard deviation of those distributions, we can standardize the scales, pick the relevant Z scores, and find out. If that sounds too straightforward, try this: is Elizabeth II older than Yao Ming is tall? This might seem unanswerable or even absurd but it's surprisingly just as straightforward! Pull some data on typical heights and ages, compute the means and standard deviations, convert Elizabeth's age and Ming's height to Z scores, and see whether Ming diverges more from a typical height than Elizabeth from a typical age! Even Zen koans tremble before the power we've unleashed.

This makes for a good sales pitch, but it's probably not immediately clear why this works. How can we be sure Quarblax will cooperate if it uses numbers we've never even seen? For the rest of this chapter, I'd like to build up some useful properties of these metrics, with the hope of making these properties seem obvious and intuitive. 

The traditional way to present this would be LaTeX-formatted algebra, but this formal presentation has some downsides. It's hard to check for mistakes, and unless you live and breath math, the formal syntax can be difficult to quickly parse and work with. Instead, with a bit of Python, we can make these demonstrations interactive, translating the mathematical notation into a form that, for a lot of us, is more comfortable. 

This won't be nearly as robust as formal proofs, but if you're just trying to get comfortable with an idea and explore, I find it an invaluable way to build up intuition and confidence. In lieu of the term "Proof", I'll call these code examples "Demonstrations".

### Demonstration 1: mean(data + c) = mean(data) + c

This first demonstration may barely seem worth it, but it's a good way to get our feet wet with the technique. Let's start by doing our algebraic manipulation in Python, saving the intermediary steps to variables, and then validating that we didn't make a mistake by checking if all the variables stored the same result.

(We could use a library like Sympy to do symbolic algebra directly, but as soon as we go that route, we need to worry about setting up a dependency manager, downloading libraries, learning their APIs, and managing permissions and virtual environments and Python versions so we don't end up with this: https://xkcd.com/1987/. All this could become a huge barrier to entry, so let's keep things simple. Instead of manipulating symbols directly, we'll pick sample values to plug in and confirm these manipulations work.)

```python
# Demo 1: mean(data + c) = mean(data) + c

# First, we pick some arbitrary values [x, y, z] for our dataset
x = 1.1
y = 2.2
z = 3.3

# n is the length of our data
n = 3

# c is the constant we're adding to every data point (also arbitrary)
c = 4.4

# Now we work through the algebra, saving each step to a variable.
# I'm not sure why I started using s (maybe it's short for step?)
# Feel free to use any other notation you find helpful

s0 = (x+c + y+c + z+c) / n   # Add a constant (c) to all values
s1 = ((x+y+z) + (n*c)) / n   # Rearranging terms, this is the same as adding (c * n)
s2 = (x+y+z)/n + (n*c)/n     # Note that (c * n) / n == c, since the n's cancel out
s3 = mean([x,y,z]) + c       # So this simplifies to (x+y+z)/n + c , which is Mean + c

# And now we print out the variables. 
# If our steps were valid, this will be (almost) equal 
s0,s1,s2,s3 
```

This is a starting point, but it leaves a lot to be desired. 

Notice that I didn't end this snippet with something like `assert s0 == s1 and s2 == s3`. Unless you're reading this on a Burroughs Adding Machine, your phone/computer uses floating point math, which has inherently limited precision (there are a million resources on the internet that explain how floating point works with varying degrees of formality. This is one of the most accessible: https://www.youtube.com/watch?v=dQhj5RGtag0). The short version is that subtle changes in how you calculate the "same" value can lead to slightly different outcomes. The canonical example that usually trips up students is:

```python
assert 0.1 + 0.2 == 0.3  # False! The left side will equal something like 0.30000000000000004
```

It's generally a bad idea to compare floating point values for equality, and some languages like Rust won't even let you. Instead, you should check that they're sufficiently close to each other. Let's write some helper functions so we don't have to print the numbers out and eyeball this:

```python
def assert_equal(values):
    base_value = values[0]
    for i, value in enumerate(values):
        difference = abs(value - base_value)
        if difference > .000001:
            raise ValueError(f"Values are not equal, starting at position {i}: {value} != {base_value}.")
    print(f"All values are equal: ~{values[0]}")
```

This function will raise an error if any of the values are more than 0.00001 away from the first, then tell us which value failed. This isn't a general solution; if we're working with very small orders of magnitude, this will be trivially true, and if we're working across many orders of magnitude, it gets much trickier to define what precision we expect. There's a whole field of study around how to handle calculations like this with maximum precision, and I don't pretend to know the first thing about it. 

So, how can we tell the difference between a harmless consequence of floating point precision and a debunking of our whole approach? We can't, and that's a very good reason not to use this approach to prove Fermat's Last Theorem.  That said, I think the benefits of this methodology in making the algebra feel visceral and immediate more than outweighs its weaknesses. We're not trying to formally prove statistics from scratch; we're trying to build intuition around what these formulas mean. 

We can somewhat mitigate the risk by randomly selecting values instead of hardcoding them ourselves. We can constrain these values to limited orders of magnitude so our equality comparison works out, while adding enough noise in the form of negative numbers, decimal places, dataset length, etc to be sure we're not gerrymandering ourselves into success.

```python
import random

def random_value():
    r = (random.random() - 0.5) * 200 # Generate a random number from -100 to 100
    if abs(r) < .000001:              # Avoid numbers extremely close to 0
        r = r + 1
    return r

def random_values(n=None):  # We can request a specific number of values if we want
    if n is None:
        n = random.randint(3, 50)
    return [random_value() for _ in range(n)]
```

Putting this together, we get this cleaned-up demonstration:

```python
# Demo 1: mean(data + c) == mean(data) + c

n = 3
c, x, y, z = random_values(n+1) # 3 data points and 1 constant value

s0 = (x+c + y+c + z+c) / n      # Add a constant (c) to all values
s1 = ((x+y+z) + (c*n)) / n      # Rearranging terms, this is the same as adding (c * n)
s2 = (x+y+z)/n + (c*n)/n        # Note that (c * n) / n == c, since the n's cancel out
s3 = mean([x,y,z]) + c          # So this simplifies to (x+y+z)/n + c , which is Mean + c

assert_equal([s0, s1, s2, s3])  # Success!
```

Alternately, if you don't like the `s` variables, you can do this, but I find it a bit harder to read:

```python
assert_equal([
    (x+c + y+c + z+c) / n,
    ((x+y+z) + (n*c)) / n,
    (x+y+z)/n + (n*c)/n,
    mean([x,y,z]) + c,
])
```

Skeptical? Try changing something. This approach is very good at catching math mistakes!

```python
s0 = (x+c + y+c + z+c) / n  # Add a constant to all values in the dataset 
s1 = ((x+y+z) + c) / n      # Factor out the addition. This is how math works, right?
s2 = mean([x,y,z]) + c/n    # I am extremely confident this is how math works.

assert_equal([s0, s1, s2])  # Whoops! This throws an error!
```

We can also make this even more general by operating on data of any size, rather than hardcoding this to three values.

```python
c = random_value()
data = random_values()               # Use a random number of data points
n = len(data)

s0 = sum([d + c for d in data]) / n  # Add c to all data points, without writing them explicitly
s1 = (sum(data) + c*n) / n           # sum(data) is the same as sum([d for d in data])
s2 = mean(data) + c                  # n cancels out, so this works with any nonzero data size

assert_equal([s0, s1, s2])
```

Hopefully this is persuasive. I'll be using this approach throughout, and it will let us build up a series of functions, demonstrations, and tests to convince ourselves of properties we'd otherwise have to painstakingly verify or merely take on faith. [Note: I'll also present the LaTex-ified math notation once I can get it to render]. 

We've now demonstrated our first important property: Adding a constant to all data points will increase the mean by the same amount. Note that a special case of this is particularly useful: if you subtract the mean itself, then the new mean is µ - µ, or 0. This is what I alluded to earlier when I said "the average deviation from the mean will always be 0". We'll use this property again before we're done.

### Demonstration 2: mean(data * m) = mean(data) * m

Multiplying every data point by a constant will also multiply the mean by that constant. We'll demonstrate this in the same way:

```python
# First, let's use this standard boilerplate to get our values

c = random_value()
data = random_values()
n = len(data)

# Now we're ready to walk through the algebra steps

s0 = sum([d * c for d in data]) * 1/n  # Multiply each term by a constant (c)
s1 = sum(data) * c * 1/n               # Factor out the constant
s2 = mean(data) * c                    # The result is sum(data) * 1/n * c, which is Mean * c

assert_equal([s0, s1, s2])
```

### Demonstration 3: stdev(data + c) = stdev(data)

The mean changes when you add a constant to every data point, but the standard deviation does not.

```python
c = random_value()
data = random_values()
n = len(data)
µ = mean(data)  # This is a legal name, but you can call it mu instead of µ if you prefer       

# First, let's show that the sum of squared distances from the mean has this property

s0 = sum([((d+c) - (µ+c))**2 for d in data])  # Adding c to all terms also adds c to the Mean (see Demo 1)
s1 = sum([(d - µ + c - c)**2 for d in data])  # The terms and the Mean both shift by c, so relative distances are unchanged
s2 = sum([(d-µ)**2 for d in data])            # This means we can ignore the constant. 

# The sum of squared errors is unaffected by adding c
assert_equal([s0, s1, s2])                    

# Which means the standard deviation is also unchanged
assert_equal([sqrt(s0 / n), sqrt(s2 / n), stdev(data)])
```

### Demonstration 4: stdev(data * m) = stdev(data) * abs(m)

If every data point is scaled by a constant, the standard deviation is scaled as well. This will be a bit more complicated.

```python
c = random_value()
data = random_values()
n = len(data)
µ = mean(data)

# Since stdev is unaffected by adding a constant to all data points (see Demo 3)
# we can make our lives much easier by subtracting the mean (µ) in advance. This will
# "shift" the data to a new mean of 0, while keeping the stdev unchanged. 

original_data = [d for d in data]
data = [d-µ for d in data]
µ = mean(data)  

# After this processing step, the new mean will be 0 (see Demonstration 1)
# while the stdev will be unchanged (see Demonstration 3)
assert_equal([µ, 0])
assert_equal([stdev(data), stdev(original_data)])


# Now that we've simplified our data set, let's multiply each term by a constant c. 
# This will also multiply the Mean by c (see Demonstration 2)
s0 = sum([(d*c - µ*c)**2 for d in data])

# Since we've set µ to 0, we don't have to bother expanding this square.
# This is simply (d*c - 0)**2 = d**2 * c**2
s1 = sum([(d*c - 0)**2 for d in data])
s2 = sum([d**2 * c**2 for d in data])

# Since c**2 is a constant, we can factor it out of the sum.
# This shows that the sum of squared distances scales by a factor of c**2
s3 = c**2 * sum([d**2 for d in data])

assert_equal([s1,s2,s3])


# Since the sum of squared distances scales by c**2, so does Variance
# I'll reset the step counter to make it clear this is a new subproblem,
# but feel free to do things differently
s0 = c**2 * sum([(d-µ)**2 for d in data]) * 1/n  # This is the definition of Variance
s1 = c**2 * variance(data)

assert_equal([s0, s1])

# And since stdev is the square root of variance, it's scaled by the absolute value of c
# ( since sqrt(c**2) == abs(c) )
s0 = sqrt(c**2 * variance(data))
s1 = abs(c) * stdev(data)

# And just to show that I wasn't doing anything tricky with the shifting,
# this also holds true for the original unshifted dataset
s2 = abs(c) * stdev(original_data)
s3 = stdev([d * c for d in original_data])

assert_equal([s0, s1, s2, s3])
```

You might need to read through that a few times or play around with it to follow all the steps, but again, the hope is that this is clearer than algebraic manipulation (or at least no worse).

### Redefining Z Scores

Let's put these rules together. The definition of a Z Score is how far a data point is from the mean, expressed in units of standard deviation. So for any specific data point d, `Z(d) = (d - µ) / σ`.

If you add a constant c to all data points, then `Z(d) = (d+c - µ+c) / (σ+0)`. The standard deviation is unchanged (Demonstration 3), and both d and µ shift by the same amount (Demonstration 1), so every individual data point's relative difference from the mean in standard deviations is unaffected. In other words, you can shift a dataset however you want without affecting its Z scores.

If you multiply all data points by c, then `Z(d) = c * (d-µ) / c * σ`. The difference from the mean is scaled by c, and the standard deviation is also scaled by c. These two scaling factors cancel each other out, so you're left with the original ratio: `(d-µ) / σ`.  In other words, you can scale a dataset however you want without affecting its Z Scores.

(There's a bit of nuance to these findings. Note that if `c` is 0, the new standard deviation will be 0, and the Z scores will be undefined. Also note that if `c` is negative, then the Z Scores will change sign.)

Let's define a "Linear Transformation" as any combination of 1) Adding or subtracting a constant and 2) Multiplying or dividing by a positive constant. What we've just shown that if two data sources are linear transformations of each other, they'll have the same Z Scores (and if the scaling factor is negative, the Z Scores will simply have opposite signs [Note: I feel like I should demonstrate this explicitly])

This is why you can convert data from different temperature scales, even if the scale is something random I made up! As long as one is a linear transformation of the other, we can standardize the units and simplify our lives. This is especially useful because Standardizing the data via Z Scores is itself a linear transformation, subtracting µ and multiplying by 1/σ. This means that `Z(Z(Z(data))) == data`, and this process is stable (or idempotent if you prefer that term).

Note, however, that Z Scores are **not** preserved across *non*linear transformations. For example, Z([1,2,3]) is very different from Z([1,4,9]).

### Mean is 0, Standard Deviation is 1

Beyond serving as a universal unit translator, our Standardized Z Scores have another nice property. Let's take another look at the formula:

```python
def z_scores(data):
    µ = average(data)
    σ = stdev(data)
    return [(d - µ) / σ for d in data]
```

Since µ is subtracted from each data point, the mean is shifted by the same amount, resulting in a mean of 0 (Demonstration 1). The mean is then scaled by 1/σ (Demonstration 2), but this is just `(µ-µ) * 1/σ`, which is `0 * 1/σ`, which is `0`.

As for the standard deviation, subtracting µ has no effect (Demonstration 3), then multiplying by 1/σ scales the standard deviation down by 1/σ. That means the standard deviation of the Z Scores is `σ * 1/σ`, which is 1.

By converting data points into Z Scores, we not only translate the units, we also set the mean and standard deviation to 0 and 1 respectively. This can simplify a lot of equations. For example, you might have seen this nightmarish equation for the Normal Distribution.

`1 / (σ * sqrt(2 * pi)) * e ** (-0.5 * (x - µ) / σ) ** 2)`

Don't worry about what this means for now. We'll come back to this equation in the future, and show that it's much less intimidating than it appears. 

What I want to demonstrate is that we already have a tool to make this tractable. Since we can translate any dataset we want into Z Scores, we can always set µ to 0 and σ to 1, simplifying this formula to:

`1 / (1 * sqrt(2 * pi)) * e ** (-0.5 * (x - 0) / 1) ** 2)`

or

`1 / (sqrt(2 * pi)) * e ** (-0.5 * (x ** 2)`

Yes, I know, this is still far from the most intuitive formula you've ever seen. But I hope this demonstrates the usefulness of Z scores in making life easier. Any time you see µ or σ in a formula, you should feel that formula calling out to you, begging you to Standardize your data, offering the promise of a better way.

Thanks for joining me on this journey. Next time, we'll look into Correlation, and how our universal unit translator can help us define what it means for two variables to move in sync.
