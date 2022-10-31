---
layout: post
title:  "3) Covariance and Correlation"
date:   2022-10-30 00:14:05 -0700
categories: jekyll update
---

In the first few chapters, we focused on how to summarize a single list of numbers, but we often want to summarize data on multiple dimensions and show how they relate to each other. For example, we might want to measure both heights and weights, or temperatures and latitudes, and see how one metric is (cor)related to the other.

I won't try to motivate the whole concept of Correlation from scratch. We already have a rough, informal sense of what it should mean: Do increases in one measure correspond to increases in another, or are the two unrelated? `[1, 2, 3]` and `[4, 5, 6]` are correlated in a way that `[1, 2, 3]` and `[8, 1, 5]` are not. Ice cream sales are positively correlated with temperature increases, but toothpaste sales are not. If you tell me a city's latitude, I can make a decent guess about its average temperature, but not if you give me its longitude. You've probably seen a thousand graphs illustrating why Correlation isn't Causation, so I won't belabor that point.

But, mathematically, what exactly is correlation? Our goal is to find a useful mathematical way to express this concept and make our intuition more robust.

Stats classes sometimes do this by defining a concept called Covariance, making you memorize a dozen covariance rules, then presenting Correlation as a special case of it. I don't love this approach, since I don't think it provides a strong foundation; covariance is wildly unintuitive and rarely used in practice. That said, it is useful to get some exposure to these terms and formulas, so I will go through this approach first. I'll quickly define Covariance, then show how Correlation is the Covariance of Standardized data.

However, I'd like to spend most of this chapter on a second way to think of Correlation: as the cosine of an angle between two vectors. This will be a bit more complicated, requiring detours through linear algebra and trigonometry, but I think it provides a firmer foundation. It might even help concepts from other math classes click into place.  

Before going through either approach, let's quickly define two linear algebra concepts:


### Vectors

A vector is a list of numbers, like `[1,2,3]`. This vector can also be interpreted as a position in 3d space, reached by going 1 unit along the x axis, 2 along the y axis, and 3 along the z axis. If the vector was a list of 28 numbers instead, it would represent a position in 28-dimensional space. 

This is not the definition your linear algebra professor would use, but the formal definition of a vector is surprisingly abstract. Since this isn't a linear algebra course, I think it's better to keep things simple and concrete (well, as concrete as 28-dimensional space can be). In previous chapters, I've described a list of values as a "dataset", but there's no meaningful distinction between the two terms; you can think of any collection of data as a vector.


### Dot Products

The Dot Product is one way to multiply two vectors together, provided those vectors are of equal length. To compute it, multiply each element in the first vector with the corresponding element in the second, then take the sum of those products.

For example, if X = [1,2,3] and Y = [4,5,6], the dot product X•Y is `1*4 + 2*5 + 3*6 = 4 + 10 + 18 = 32`. In Python:

```python
def dot(x, y):
    return sum([x[i] * y[i] for i in range(len(x))])
```

Since the dot product is a series of multiplications, most of the properties of traditional multiplication map very nicely onto it. For example: A•B = B•A. 

If you multiply each value in one of the datasets by a constant, the dot product will also be multiplied by that constant:

`[x1, x2, x3] • [c*y1, c*y2, c*y3]`
`(x1 * c * y1) + (x2 * c * y2) + (x3 * c * y3)`
`c * (x1 * y1 + x2 * y2 + x3 * y3)` (Factor out c)
`c * ([x1, x2, x3] • [y1, y2, y3])`

Dot products also expand the same way as tradition multiplication. That is,
`(a+b)•(c+d) = a•c + a•d + b•c + d•c`. This is because, for each individual product: `(a1+b1) * (c1+d1) = a1*c1 + a1*d1 + b1*c1 + b1*d1`.

## Approach 1: Covariance and Correlation

Covariance (often abbreviated cov) is the average of the dot product of two sets of distances from their respective means.

```python
def covariance(x, y):
    µ_x = mean(x)
    µ_y = mean(y)
    return mean([(x[i] - µ_x) * (y[i] - µ_y) for i in range(len(x))])
```

If that's a bit hard to follow, think of covariance as the result of "shifting" both datasets until they have a mean of zero, taking the dot product of the results, and dividing that dot product by the number of data points in each dataset. 

```python
def shift(data):
    µ = mean(data)
    return [d-µ for d in data]

def covariance(x, y):
    return dot(shift(x), shift(y)) / len(x)
```

Either way, this is difficult to motivate, and even harder to make intuitive. Here's my best effort: 

Covariance is a way to measure whether high and low deviations from two means are associated with each other. If unusually high ice cream sales correspond with unusually high temperatures, then the covariance between ice cream sales and temperatures will be large, since you'll multiply two large values together. On the other hand, if there's no consistent association, then low values in one dataset will "dampen" the effect of high values in the other. This is easiest to see with examples:

#### Perfect association (positive and negative)

[-2, -1, 1, 2] and [-2, -1, 1, 2] have a covariance of 4+1+1+4 / 4 = 10 / 4 = 2.5

[-2, -1, 1, 2] and [2, 1, -1, -2] have a covariance of -4-1-1-4 / 4 = -10 / 4 = -2.5

#### Weaker associations

[-2, -1, 1, 2] and [-2, -1, 2, 1] have a covariance of 4+1+2+2 / 4 = 9 / 4 = 2.25

[-2, -1, 1, 2] and [-1, -2, 2, 1] have a covariance of 2+2+2+2 / 4 = 8 / 4 = 2

#### No association

[-2, -1, 1, 2] and [1, -2, 2, -1] have a covariance of -2+2+2-2 / 4 = 0 / 4 = 0

[-2, -1, 1, 2] and [0, 0, 0, 0] also have a covariance of 0+0+0+0 / 4 = 0

### Unit Confusion

Unfortunately, any intuition falls apart when we introduce units. If I record the prices and areas of apartments, the covariance between these metrics isn't price *per* square foot, but price *times* square foot. The covariance between 6 feet and 2 yards is 12 foot-yards.

Even when the units appear to make sense, this is often a mirage. Say we have a set of square tables. If their widths and lengths in meters are both `[1, 2, 3, 4, 5]`, then the covariance between width and length is 2 m^2. That might seem like a sensible value, but it's not the average area of the tables (which is 11 m^2) or the average deviation from that area (which is 0 m^2). It's... the average of the products of each table's differences from the *average* width and length. 

Imagine you don't have the original data, and you didn't just read the definition of covariance two minutes ago, and I tell you the covariance between two measurements is 2 m^2. How would you try to interpret that? Is that high? Low? Can you visualize it in a way that's not actively misleading? Would you ever guess the underlying measurements were the same?

### Z Scores / Standardization

If this seems hopeless, remember that Z Scores act as a universal unit translator, and that we should Standardize our data whenever units get in our way. This is the key to making Covariance usable at all. To Standardize a dataset, we express each data point as its distance from the mean, in units of standard deviations. 

The covariance calculation already subtracts the mean from each data point. If we do that ourselves beforehand, the covariance formula will just subtract 0 (the tautological average distance from the mean) so the overall result will be the same. 

As for dividing by standard deviation, remember that X•(c * Y) = c * (X•Y). Since covariance is just a dot product divided by n, this also means that `cov(X, c*Y) = n * X•(c*Y) = c * n * X•Y = c * cov(X, Y)`. In other words, dividing all data points in x by `stdev(x)` will also divide the resulting covariance by `stdev(x)`, and doing the same for y will also divide the covariance by `stdev(y)`, so the final result will be divided by the product `stdev(x) * stdev(y)`.

This gives us two ways to express the same definition:

```python
def correlation(x, y):
    return covariance(x, y) / (stdev(x) * stdev(y))

def correlation(x, y):
    return covariance(z_scores(x), z_scores(y))
```

The first formula is how you'll usually see this in a textbook (probably because it's easier to write in formal math notation), but I find the second makes it much clearer what's going on.

### The problems with this approach

We could stop here, but I'm not satisfied with these formulas or how we derived them. We just created a bizarre measurement with nonsensical units, then coerced the units to solve the problem we just made, and now our mental model of Correlation rests on an extremely weak foundation. 

The upper bound of Correlation is 1. But... why? Well, the upper bound of `covariance(x,y)` is `stdev(x) * stdev(y)`, so if you divide covariance by that same product, the result can't be greater than 1. But... why is that the upper bound of covariance? This is where your textbook alludes to something called the Cauchy Schwarz Inequality, then tells you to take linear algebra to learn more. And this is where you throw up your hands in frustration, memorize this for the test, then promptly forget everything you've learned. 

## Approach 2: Dot Product -> Correlation

If we're going to bring in linear algebra, let's not go halfway. Let's start over and take a second approach, in which we think of Correlation as the cosine of an angle between vectors. We'll have to lay a lot of groundwork, and this won't be trivial, but it (hopefully) will be rewarding. Brace yourself.

### Vector Addition

<img src="/images/Vectors.png" width="800"/>

To add two vectors together, simply add the components together. To demonstrate why this makes sense, recall that vectors can be interpreted as positions in space, with each value representing a distance along an axis.

In the example above, A is the vector `[3, 0]` and B is the vector `[0, 4]`. If you walk 3 units along the X axis, then 4 units along the Y axis, you reach the position `(3, 4)`. This is the same result as if you had walked directly along C, the vector `[3, 4]`. When A, B, and C are vectors, we can represent this relationship as A + B = C.

<img src="/images/Vectors2.png" width="800"/>

For a more complex example, A is the vector `[6, 3]` and B is the vector `[-2, 1]`. The result of walking A then B is again the same as walking C, representing the vector `[6-2, 3+1]` or `[4, 4]`. Once again, we can express this as A + B = C.

### Vector Magnitudes

When describing vectors, the word "length" is somewhat ambiguous. We could say the vector `[3, 4]` has a length of 2, because it has two elements, but we might also want "length" to represent its total distance from the origin. To avoid ambiguity, we use the term "dimension" to refer to how many elements make up a vector, and "magnitude" to refer to its distance from the origin.

The magnitude of C is represented as `|C|`, and in the first example above, `|C| = |[3, 4]| = 5`. Notice that A and B form two legs of a right triangle, with C as the hypotenuse. The Pythagorean Theorem tells us C must have magnitude 5, since `3^2 + 4^2 = 5^2`.

In the second picture, `|C|` is sqrt(32), which is about 5.66. This is because A + B = C = [4, 4], and C can be thought of as the hypotenuse of a right triangle, where both sides have a length of 4. In this case, the Pythagorean Theorem tells us that `4^2 + 4^2 = 32 = |C|^2`, so `|C| = sqrt(32)`. 

This syntax might take some getting used to. Just remember that `A + B = C` does not mean that `|A| + |B| = |C|`. The first formula describes a destination, while the second describes the most direct route to get there. Just because you got somewhere doesn't mean you took the most efficient route!

### Magnitude in other dimensions

In two dimensions, the Pythagorean Theorem tells us that the magnitude of `[x, y]` is `sqrt(x^2 + y^2)`. But what about in other dimensions?

Well, in one dimension, the magnitude of the vector `[x]` is simply x, since going `x` units in a direction means you're now `x` units away from where you started. We can express this incredibly uninteresting fact as `|x| = x = sqrt(x^2)`, for reasons which will become clear in a moment.

What if we have a three-dimensional vector, like `[3, 4, 12]`? Remember that this vector represents going 3 units along x, then 4 along y, then 12 along z, and that the x, y, and z directions are perpendicular to each other (for "short", x, y, and z are orthogonal). If we break this down into three components, `[3, 0, 0]`, `[0, 4, 0]` and `[0, 0, 12]`, the first two components represent walking 3 units along x, then 4 along y. Since the z direction isn't involved, this is the same as going `[3, 4]` on a two-dimensional plane. We already saw this vector above and showed that it had a magnitude of 5. The third component, `[0, 0, 12]`, is perpendicular to the whole x-y plane, and the first two components work out to traveling 5 units on the x-y plane. This means we can represent the combined vector as another right triangle, with legs of length 5 and 12. The total distance traveled will be `sqrt((sqrt(3^2 + 4^2))^2 + 12^2) = sqrt(3^2 + 4^2 + 12^2) = sqrt(5^2 + 12^2) = 13`. This might make more sense with a picture:

<img src="/images/3D.gif"/>

[TODO: I need to make my own image here.]

If we add another orthogonal dimension, we can continue this line of thinking. The vector `[3, 4, 12, 17]` can be broken down into `[3, 4, 12, 0]` and `[0, 0, 0, 17]`, where the first vector represents traveling `sqrt(3^2 + 4^2 + 12^2) = 13` units in x-y-z space. If you travel 13 units, then turn and walk 17 units orthogonally, the Pythagorean Theorem describes your total distance in 4d space as `sqrt(13^2 + 17^2) = sqrt(3^2 + 4^2 + 12^2 + 17^2) ≈ 21.4`. This quickly becomes impossible to visualize, but the induction holds. For any dimension, the magnitude of a vector is the square root of the sum of the squares of its elements:

```python
def magnitude(vector):
    return sqrt(sum([v**2 for v in vector]))
```

### Dot Product and Magnitude

By this definition, if a vector V has the elements `[a, b, c, d, ...]`, then its magnitude is `sqrt(a^2 + b^2 + c^2 + d^2 + ...)`. Meanwhile, by the definition of a dot product, `V•V = a*a + b*b + c*c + d*d + ... = a^2 + b^2 + c^2 + d^2 + ...`. Put differently, the dot product of a vector with itself is equal to the vector's magnitude squared: `V•V = |V|^2`. 

```python
def magnitude(vector):
    return sqrt(sum([v**2 for v in vector])
```

### Magnitude and Standard Deviation

Two chapters ago, we defined the Variance of a dataset as the sum of each data point's squared distance from the mean, and Standard Deviation as the square root of variance. We can express this in Python as:

```python
def stdev(data):
    µ = mean(data)
    n = len(data)
    return sqrt(sum([(d - µ)**2 for d in data]) / n)
```
or
```python
def stdev(data):
    µ = mean(data)
    n = len(data)
    return sqrt(sum([(d - µ)**2 for d in data])) / sqrt(n)
```

If the mean is 0, this formula simplifies to:

```python
def stdev_with_zero_mean(data):
    return sqrt(sum([d**2 for d in data])) / sqrt(len(data))
```

Which we can express using the `magnitude` function we just defined:

```python
def stdev_with_zero_mean(data):
    return magnitude(data) / sqrt(len(data))
```

In the previous chapter, we demonstrated that adding or subtracting a constant from every data point ("shifting" the data) has no effect on standard deviation. Since we can "shift" the mean of any dataset to 0, we can always use this alternate formula to calculate standard deviation. Alternately, we can define the magnitude of a vector in terms of its standard deviation:

```python
def magnitude(data):
    n = len(data)
    return stdev(data) * sqrt(n)
```

## Cosines

Stepping back into two dimensions, let's (re?)acquaint ourselves with some trigonometry.

<img src="/images/tri0.png" width="800"/>

Consider a right triangle. One angle is 90 degrees, while another is represented as θ (theta). The three sides of the triangle are labeled A (**A**djacent to the angle θ), O (**O**pposite the angle θ), and H (the **H**ypotenuse). 

If you've taken a trig class, you probably learned the mnemonic "Soh-Cah-Toa". This defines three important functions as ratios: **S**ine (or Sin) is **O**pposite / **H**ypotenuse, **C**osine (or Cos) is **A**djacent / **H**ypotenuse, and **T**angent (or Tan) is **O**pposite / **A**djacent.

If θ is 36 degrees, then the adjacent side A is about 81% as long as the hypotenuse, which is the same as saying `cos(θ) ≈ 0.81`. Note that this ratio doesn't depend on any actual lengths. If the hypotenuse is 100 feet, then the adjacent side is 81 feet. If the hypotenuse is 750 cm, then the adjacent side is 607.5 cm. This is because the value of θ completely determines the shape of the triangle. Since the angles in any triangle must add up to 180 degrees, these angles will always be 90, θ, and 90-θ (in this case, 90, 36, and 54 degrees). Since these angles are fixed, the shape of the triangle is also fixed. We can scale it to be larger or smaller, but without changing the angles, there's no way to change the relative lengths of the sides. 

<img src="/images/tri1.png" width="800"/>

If we convert our units to "hypotenuse lengths", where the length of the hypotenuse itself is defined to be 1, the ratio `cos(θ) = Adjacent/Hypotenuse` becomes `cos(θ) = Adjacent/1`, or `Adjacent = cos(θ)`. By the same logic, `Opposite = sin(θ)`. By the Pythagorean Theorem, `sin(θ)^2 + cos(θ)^2 = 1^1 = 1`. 

This last relationship is not just a trick of the units. For any values o, a, and h:

`cos(θ)^2 = a^2 / h^2`
`sin(θ)^2 = o^2 / h^2`
`cos(θ)^2 + sin(θ)^2 + = (a^2 + o^2) / h^2`

Since the Pythagorean Theorem tells us `a^2 + o^2 = h^2` for any right triangle, this ratio will always equal 1.

### Trigonometry is about circles

<img src="/images/tri2.png" width="800"/>

We've been working with triangles, but it's often much more powerful to visualize these functions using a Unit Circle, so called because its radius is defined as 1 unit. 

By turning any angle θ and traveling 1 unit, we'll reach the edge of the circle (this is the definition of a circle with radius 1). This path can be represented as the hypotenuse of a right triangle, with one side on the x axis and another parallel to the y axis. By the definitions above, we know the distance traveled along the x axis is `cos(θ)` and the distance along the y axis is `sin(θ)`, so the point on the circle we reached is `(cos(θ), sin(θ))`. In other words, the radius we walked is the vector `[cos(θ), sin(θ)]`, which has a magnitude of `sqrt(cos(θ)^2 + sin(θ)^2) = sqrt(1) = 1`. 

### A detour on measuring angles

<img src="/images/tri3.png" width="800"/>

In everyday life, we usually measure angles in degrees; a rotation is 360 degrees, a quarter rotation is 90 degrees, etc. As you get deeper into math, you trade degrees for radians, and (try to) think of a rotation as 2π radians, a quarter rotation as π/2 radians, etc. 

We could do that, but if you're the type of person who reads about math on the internet, you've probably seen arguments that π is an awkward value to use here, and we should prefer using τ (tau, which rhymes with "cow"). For a detailed argument, you can read https://tauday.com/tau-manifesto. For the short version, π is the ratio of a circle's circumference to its *diameter*, but we usually care more about its *radius*. The circumference to radius ratio is 2π, or ~6.28, or τ, and if we use this instead of π, the math becomes much more intuitive.

τ/10 is ~0.628, and τ/10 radians formally represents "the angle which circumscribes an arc of a circle such that the arc is 0.628 times the length of the radius". An angle of τ circumscribes the whole circle, so the arc it forms is the circumference; the ratio of that circumference to its radius is, by definition, τ. 

This is a nice definition, especially compared to the arbitrary definition of a Degree, but you don't have to memorize it. Instead, just think of τ as being short for "turn" (τurn?). τ radians is a full turn, or 360 degrees. τ/4 is a quarter turn, or 90 degrees. A 12 degree angle is one thirtieth of a turn, which is much clearer if I call it τ/30. 

If an angle represents a number of turns, the cosine of that angle represents the x coordinate, or horizontal distance from the origin, reached after those turns, starting from the position (0, 1) and turning counter-clockwise (or clockwise if the angle is negative). The cosine of τ/10 is ~0.81, which means that if you start at (0, 1) and make one-tenth of a turn around the circle's edge, your horizontal distance from the origin is 81% what it would have been if you hadn't turned at all. 

### Cosines of Arbitrary Angles

<img src="/images/tri4.png" width="800"/>

We started defining the Cosine as the ratio between sides of a right triangle, but this model has its limitations. On a right triangle, it doesn't make sense for one side to have zero length, or for an angle to be negative or more than τ/4 radians (90 degrees).

If we think of the Cosine as the x coordinate on a unit circle, it's easier to see how it applies to angles beyond the 0 to τ/4 range. The input range is unbounded, and the results are periodic; making 3.5 turns leaves you in the same place as only making half a turn, so `cos(θ) = cos(θ % τ)`, where `%` is the modulo operator (that is, `θ % τ` is the remainder left over from dividing `θ / τ`).

Good so far? I promise this will eventually get us to Correlation.

## The Law of Cosines

We've been relying on the Pythagorean Theorem throughout this chapter, but that theorem only applies to right triangles. There's a more general formula that applies to all triangles, called the Law of Cosines: 

`a^2 + b^2 - 2 * a * b * cos(C) = c^2`

In this formula, capital `C` represents the angle between sides a and b, and opposite side `c`. On a right triangle, this angle is 90 degrees or τ/4 radians, and since `cos(τ/4) = 0`, this formula simplifies to the traditional Pythagorean Theorem: `a^2 + b^2 - 0 = c^2`.

### The Law of Cosines for Obtuse Triangles

<img src="/images/Obtuse.png" width="800"/>

To demonstrate the Law of Cosines on an obtuse triangle, imagine it's a subdivision of a hypothetical right triangle. 

By the Pythagorean Theorem:
`a^2 = y^2 + (b+x)^2`, so
`y^2 = a^2 - (b+x)^2`

By the right triangle definition of a cosine:
`cos(C) = (b+x) / a`, so
`x = a * cos(C) - b`

Also by the Pythagorean Theorem:
`c^2 = y^2 + x^2`, so
`c^2 = a^2 - (b+x)^2 + x^2` (substitute y^2)
`c^2 = a^2 - (b^2 + 2*b*x + x^2) + x^2` (expand (b+x)^2)
`c^2 = a^2 - b^2 - 2*b*x` (simplified)
`c^2 = a^2 - b^2 - 2*b*(a*cos(C) - b)` (substitute x)
`c^2 = a^2 - b^2 - 2*b*a*cos(C) + 2*b^2` (distribute -2*b)
`c^2 = a^2 + b^2 - 2 * a * b * cos(C)` (simplified, this is the Law of Cosines) 

### The Law of Cosines for Acute Triangles

<img src="/images/Acute.png" width="800"/>

Similarly, we can think of an acute triangle as being made up of two right triangles. 

By the right triangle definition of Cosine:
`cos(C) = w / a`, so
`w = a * cos(C)`

Since we've divided side `b` into `w + x`, by definition:
`x = b - w`, so
`x = b - a * cos(C)` (substitute w)

By the right triangle definition of Sine:
`sin(C) = y / a`, so
`y = a * sin(C)`

By the Pythagorean Theorem:
`c^2 = x^2 + y^2`, so
`c^2 = (b - a * cos(C))^2 + (a * sin(C))^2` (substitute x and y)
`c^2 = b^2 - 2*a*b*cos(C) + a^2 * (cosC)^2 + a^2 * (sin(C))^2 ` (expand the squares)
`c^2 = b^2 - 2*a*b*cos(C) + a^2 * (sin(C)^2 * cos(C)^2)` (factor out the a^2 term)
`c^2 = b^2 - 2*a*b*cos(C) + a^2 * (1)` (simplify sin^2 + cos^2 = 1)
`c^2 = a^2 + b^2 - 2 * a * b * cos(C)` (rearranged, this is the Law of Cosines)

We've now shown that the Law of Cosines applies for all triangles, whether right, obtuse, or acute.

## Relating the Cosine and Dot Product

<img src="/images/Vectors3.png" width="800"/>

Let's revisit an example from earlier. A and B are vectors, and C is the vector that results from traveling along A then B. This means that A + B = C, which can also be expressed as C = A - B. `|C|^2` (the squared magnitude, or length, of C) can be represented as:

`|C|^2 = C•C` (Demonstrated earlier)
`= (A-B)•(A-B)` (Substitute C = A - B)
`= A•A + B•B - 2 * A•B` (Dot product expansion works like normal multiplication)
`= |A|^2 + |B|^2 - 2*A•B` (Replace dot products with magnitudes squared)

By the Law of Cosines:
`|A|^2 + |B|^2 - 2*|A|*|B|*cos(θ) = |C|^2`, so
`|A|^2 + |B|^2 - 2*|A|*|B|*cos(θ) = |A|^2 + |B|^2 - 2*A•B` (substitute |C|^2)
`-2*|A|*|B|*cos(θ) = -2*A•B`  (Subtract |A|^2 + |B|^2 from both sides)
`|A|*|B|*cos(θ) = A•B` (Divide both sides by -2)
`cos(θ) = A•B / (|A|*|B|)` (Rearrange terms)

In other words, if you divide the dot product of two vectors by the product of their magnitudes, the result is the cosine of the angle between them. If both vectors are the same, the angle between them is 0, and the cosine of 0 (the x position after 0 turns) is `cos(0) = (A•A / |A|^2) = 1`.

<img src="/images/example.png" width="800"/>

It's easy to get lost in the math, so let's see an example of how this works in practice. If we have two vectors, `[3, 4]` and `[5, -12]`, we can use this formula to calculate the cosine of the angle between them. We can then take the inverse of that cosine (also called the arc cosine, or acos) to get the angle itself. In this case, that angle is about τ/3 radians, or one-third of a turn.

```python
π = 3.141592653589793
τ = 2*π  # You can also import pi and tau from the math library

a = [3, 4]   # A has a magnitude of 5
b = [5, -12] # B has a magnitude of 13

dot_product = dot(a, b)   # 3 * 5 + 4 * -12 = 15 - 48 = -33
cosθ = dot_product / (magnitude(a) * magnitude(b))  # -33 / (5*13) = -33/65 ≈ -0.50769
θ = math.acos(cosθ)  # Theta is about 2.1033 radians
rotations = θ / τ  # Which is ≈ 0.3348 * τ
print(rotations)  # Or about one third of a turn
```

In two dimensions, we can visually confirm that the angle between these vectors is roughly one third of a turn. Unfortunately, once we get to 4 dimensions, or 28 dimensions, or 1 million dimensions, we can't lean on visual guides. We'll have no choice except to trust the math. 

## Putting it Together: Correlation as a Cosine

Finally, we're ready to tie this all together. If X and Y are both n-dimensional vectors with a mean of 0, then the product of their magnitudes is:
`|X| * |Y|` 
`= stdev(X) * sqrt(n) * stdev(Y) * sqrt(n)` (since magnitude = stdev * sqrt(n))
`= stdev(X) * stdev(Y) * n` (substitute n = sqrt(n)^2)

Using the relationship defined above:
`cos(θ) = X•Y / (|X|*|Y|)`, so
`cos(θ) = X•Y / (stdev(X) * stdev(Y) * n)` (substitute |X|*|Y|)

`X•Y / (stdev(X) * stdev(Y)` is also the result of dividing every data point in X by stdev(X) and every data point in Y by stdev(Y).Shifting the mean to 0 and dividing by both standard deviations is the same as standardizing (taking the Z Scores of) both data sets. If X and Y are already standardized, these standard deviations will be 1, so this division will have no effect. In either case, we can represent this relationship as the dot product of Z Scores, divided by the dimension of the vectors. We can call the result the "cosine of the angle between two standardized vectors", or we can call it Correlation. This is the same Correlation function we defined in Approach 1. 

```python
def cosine_of_angle_between_vectors(x, y):
    assert len(x) == len(y)
    return dot(z_scores(x), z_scores(y)) / len(x)

def correlation(x, y):
    return cosine_of_angle_between_vectors(x, y)

def angle_between_vectors(x, y):
    return math.acos(correlation(x, y))

def turns_between_vectors(x, y):
    return angle_between_vectors(x, y) / τ
```

If we have two lists of N data points, such as heights and weights, we can treat those datasets as vectors in N dimension space, standardize their units by taking their Z Scores, then calculate the cosine of the angle between them to see how closely they line up.

If the standardized vectors point in the exact same direction, then the angle between them is 0, and the cosine of that angle is 1. If they point in opposite directions, the angle is τ/2 radians (half a turn, or 180 degrees) and the cosine of that angle is -1. Both of these situations are considered perfect correlation, since knowing one vector gives you perfect information about the other. If X is a linear transformation of Y, there will always be perfect correlation between X and Y.

If the standardized vectors are completely orthogonal (perpendicular on every axis), the angle between them is τ/4 radians (a quarter turn, or 90 degrees), and the cosine of that angle is 0. This represents no correlation; knowing the values of one vector gives you absolutely no information about the other. 

As these vectors get closer together, their correlation does not increase linearly. This can be unintuitive, especially if you think of correlation in terms of covariance, but the unit circle definition helps make it clear; a nonlinear relationship is less surprising if your mental model is a curve instead of a line! Halfway between 0 turns (with a cosine of 1) and a quarter turn (with a cosine of 0), the cosine of τ/8 radians is `sqrt(2)/2`, about 0.707. To get a correlation of 0.5, you have to find the angle which yields an x coordinate of 0.5, which is τ/6 radians (1/6 of a turn).

### Revisiting covariance

This took much longer than the first approach, but I think we've built a much stronger foundation. If nothing else, I hope it was interesting. Now that we have a clearer understanding of Correlation, we can flip our original approach on its head, and define Covariance in terms of Correlation:

```python
def covariance(x, y):
    return correlation(x, y) * stdev(x) * stdev(y)
```

This makes it easier to think through the properties of covariance, like why its upper bound is the product of both vectors' standard deviations. It also exposes the fundamental strangeness of this as a measurement, and why it's so rarely useful. Some textbooks present Variance as a special case of Covariance, in an attempt to make Covariance seem more fundamental than it actually is. It is true that `covariance(x, x) = variance(x)`, but once you've seen this formula, that's not a very interesting result:

`covariance(x, x)` 
`= correlation(x, x) * stdev(x) * stdev(x)` 
`= 1 * stdev(x)^2` 
`= variance(x)` 

You'll rarely see Covariance in practice, and the few times you do, you can use this formula to reorient around correlation. One of the (very) few places I have seen it used is in β (Beta), a financial metric that describes how a stock moves relative to the overall market. This is traditionally defined as `β = covariance(stock, market) / variance(market)`, but it can also be expressed as `β = correlation(stock, market) * stdev(stock) / stdev(market)`, which is arguably clearer.

Covariance does have some benefits over correlation. It's easier to calculate, so if you only care about the sign of a relationship and not its magnitude, you can use the covariance formula to save some work. If you want to use Calculus with a metric like Beta, the original formula has a simpler partial derivative. Perhaps most importantly, covariance supports homogenous datasets, while correlation does not; `covariance([1,2], [1,1])` is 0, but `correlation([1,2], [1,1])` is undefined, since Standardizing the homogenous dataset `[1,1]` would require dividing by 0. For our purposes, we can handle this with an explicit check or a try/except block.

I won't be dogmatic and say you should never use covariance, but I do recommend preferring correlation, say, 98% of the time. And if you're presenting data to someone without a strong stats background, that guideline approaches 100%.
