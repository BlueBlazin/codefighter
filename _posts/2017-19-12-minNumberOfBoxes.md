---
layout: post
title:  "minimumNumberOfBoxes"
date:   2017-12-19
categories: NumberTheory
image: img/minNumberOfBoxes.png
---

**Request:** Give your best effort to solve it yourself before viewing the solution below.

---
**Problem:**

The store has an infinite number of apple boxes and banana boxes. There are a apples in each apple box, and b bananas in each banana box. We are interested in whether we can get exactly f pieces of fruit by ordering boxes of apples or bananas.

Given `a`, `b`, and `f`, your task is to find the minimum `n` such that it is possible to order `n` boxes of fruit (either apple or bananas) to get `f` pieces of fruit in total. Because we can only order whole boxes (and we cannot order a negative number of boxes), it may not be possible to complete the order. If it is not possible to order f pieces of fruit, return `-1`.

**Example:**

For `a = 5`, `b = 4` and `f = 19`, the output should be
`minimumNumberOfBoxes(a, b, f) = 4`

---

This problem is essentially asking us to solve an arithmetic equation, called a linear Diophantine equation, of the form: $ax + by = f$. The twist, however, is that we are also given two added constraints:

- $x \geq 0, y \geq 0$
- $x + y$ must be minimum of all possible $(x, y)$ solutions.

We'll solve this in steps. First, note that a solution to this equation will only exist if the gcd of $a, b$ divides $f$. In other words $f$ must be divisible by $d = gcd(a, b)$. Why is this so? Let $d = gcd(a, b)$. Then $a = dr$, and $b = ds$, for some integers $r$, and $s$. This means,

$$f = ax + by = drx + dsy = d(rx + sy).$$

Thus, $f$ must be divisible by $d$. As such, we can elimate tests where this condition is not met.

```python
from fractions import gcd

def minimumNumberOfBoxes(a, b, f):
  d = gcd(a, b)
  if f % d != 0 or d == 0: return -1
```

The `or d == 0` contidion it tacked on for the case where both `a`, and `b` are 0. There's a result from number theory which states that the gcd $d = gcd(a, b)$ can be expressed as a linear sum of $a$, and $b$:

$$ax + by = d.$$

While the Euclidean algorithm can help us find the gcd of two integers, a slightly tweaked version of it called the Extended Euclidean Algorithm can be used to find the $x$, and $y$ from the above equation. So, while we don't yet know how to solve $ax + by = f$, we can solve $ax + by = d$. Thinking about this for a while, we can use the following trick. First, divide both sides of our original equation by $d$.

$$\frac{a}{d}x + \frac{b}{d}y = \frac{f}{d}.$$

Now, $a/d$, and $b/d$ are relatively prime and so their gcd is 1. Thus we can find $x'$, and $y'$ such that $\frac{a}{d}x' + \frac{b}{d}y' = 1$.

$$\rightarrow a\frac{fx'}{d} + b\frac{fy'}{d} = f,$$

$$\rightarrow ax + by = f.$$

What do you know, we just recovered our original equation! So if we can find $x'$, and $y'$, we can multiply them by $f/d$ to get $x_0$, and $y_0$. 

$$x_0 = \frac{fx'}{d}, y_0 = \frac{fy'}{d}.$$

Here $x_0$, and $y_0$ are a particular solution of $ax + by = f$.

```python
from fractions import gcd

def egcd(a, b):
    prevx, x = 1, 0; prevy, y = 0, 1
    while b:
        q = a//b
        x, prevx = prevx - q*x, x
        y, prevy = prevy - q*y, y
        a, b = b, a % b
    return prevx, prevy

def minimumNumberOfBoxes(a, b, f):
  d = gcd(a, b)
  if f % d != 0 or d == 0: return -1

  xp, yp = egcd(a//d, b//d)
  x0, y0 = f * xp // d, f * yp // d
```

Here `egcd` is the Extended Euclidean Algorithm and you can find more information on it here: https://anh.cs.luc.edu/331/notes/xgcd.pdf

Alright! We have a solution, but it's probably not going to pass for most tests on CodeFights. First, we need both $x$, and $y$ to be nonnegative. On top, we also need $x + y$ to be minimum. Number theory tells us that if we know a particular solution, we can use it to find any solution as follows:

$$x = x_0 + \frac{b}{d}t, y = y_0 - \frac{a}{d}t,$$

where $t$ is an integer. We can use these formulas to find positive solutions as follows. We need both $x$, and $y$ to be nonnegative. So,

$$x_0 + \frac{b}{d}t \geq 0,$$

$$\rightarrow t \geq \frac{-x_0d}{b},$$

and,

$$y_0 - \frac{a}{d}t \geq 0,$$

$$\rightarrow t \leq \frac{y_0d}{a}.$$

Combining these together we get,

$$\frac{-x_0d}{b} \leq t \leq \frac{y_0d}{a}.$$

If you pick a value for $t$ between these bounds, and compute and print $x$ & $y$ you will observe that they are indeed always positive. The only little issue here is that $t$ must be an integer, so we can make the inequality even tighter,

$$\lceil \frac{-x_0d}{b} \rceil \leq t \leq \lfloor \frac{y_0d}{a} \rfloor.$$

Okay, that's a lot of math, and depending on your inclination toward it you may feel exhausted already. So let's implement this in code and use say, the lower bound as our value for $t$.

```python
from fractions import gcd
from math import ceil

def egcd(a, b):
    prevx, x = 1, 0; prevy, y = 0, 1
    while b:
        q = a//b
        x, prevx = prevx - q*x, x
        y, prevy = prevy - q*y, y
        a, b = b, a % b
    return prevx, prevy

def minimumNumberOfBoxes(a, b, f):
  d = gcd(a, b)
  if f % d != 0 or d == 0: return -1

  xp, yp = egcd(a//d, b//d)
  x0, y0 = f * xp // d, f * yp // d
  t = ceil(-x0 * d / b)
  x, y = x0 + b*t//d, y0 - a*t//d
  return x + y
```

Okay, this is great and it does produce the correct result on many test cases, but not all. First, we missed one detail from the problem. There doesn't necessarily have to be a positive solution to a given equation. Second, some of the solutions are just incorrect. Was our guess about always choosing the lower bound as the value for $t$ correct?

We could perform another mathematical analysis and figure our what to do, but instead let me take this opportunity to present another common problem solving technique. Computer Science is a science and as such we can, and should use experimentation to help us figure out a problem.

If you check out some of the incorrect solutions, you'll notice two things. One of $(x, y)$ has a negative value and the expected solution was $-1$. This is expected for those cases where a positive solution doesn't exist. The other thing you may notice is on incorrect solutions, the value of `a` happens to be greater than the value of `b`. If our guess is right and `a` always needs to be less than or equal to `b` for $t = \lceil \frac{-x_0d}{b} \rceil$ to work, we can just swap `a` and `b` to always make $a \leq b$. The **final code** then is,

```python
from fractions import gcd
from math import ceil

def egcd(a, b):
    prevx, x = 1, 0; prevy, y = 0, 1
    while b:
        q = a//b
        x, prevx = prevx - q*x, x
        y, prevy = prevy - q*y, y
        a, b = b, a % b
    return prevx, prevy

def minimumNumberOfBoxes(a, b, f):
  d = gcd(a, b)
  a, b = (a, b) if a <= b else (b, a)
  if f % d != 0 or d == 0: return -1
  if a == 0 or b == 0: return f // d
  xp, yp = egcd(a//d, b//d)
  x0, y0 = f * xp // d, f * yp // d
  t = ceil(-x0 * d / b)
  x, y = x0 + b*t//d, y0 - a*t//d
  return -1 if x < 0 or y < 0 else x + y
```

So there you have it. We used some number theory, some mathematical analysis, and finally some experimentation to find an elegant solution to this challenging problem.