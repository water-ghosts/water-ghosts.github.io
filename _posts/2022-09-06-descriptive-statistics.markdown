---
layout: post
title:  "1) Mean and Standard Deviation"
date:   2022-09-06 18:00:05 -0700
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

There's an intuitive logic to Mode, since it answers the question "If we picked a data point at random, what's the most likely value we'll get?", but that's really its only advantage. It's surprisingly awkward to calculate (although this could be streamlined with Python's `collections.counter`, which feels tailor-made for this exact situation), and it's often extremely misleading. For example, the series `[1, 1, 996, 997, 998, 999...]` has a mode of 1, perhaps its least useful or interesting feature.

Mode can also be extremely sensitive to small variations, since it's an all-or-nothing measurement. The series `[1, 1, 999.999, 1000]` can see its Mode increase three orders of magnitude if you nudge the third data point. If you're working with imprecise floating point math, the Mode might depend on whether this rounds to 1000 or 1000.0000000000001. Alternately, the Mode might not change at all, since it's unclear how this definition breaks ties. Meanwhile, if your dataset is `[1, 1, x]`, then no value of x can ever change the mode, whether it's 1, 2, or 2 trillion.

Mode can be useful when categorizing numbers where the magnitude and order are irrelevant (e.g. phone numbers, serial numbers, and IDs). But outside of that narrow context, Mode mostly offers countless ways to lie with statistics. If you hear someone say "the modal outcome is X", exercise the same caution you would when a salesman extols a used car. Clearly, we need something better.

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

We could measure the total deviation of all data points from the mean, but this wouldn't mean much on its own; a total temperature deviation of 30 degrees C means something very different if your time horizon is three hours vs three years. We can't solve this by taking the average, since the average deviation from the mean will always be 0 (don't worry if this isn't immediately obvious; we'll demonstrate why it is in a few minutes). 

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