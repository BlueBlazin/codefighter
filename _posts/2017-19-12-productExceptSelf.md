---
layout: post
title:  "productExceptSelf"
date:   2017-12-19
categories: CommonTechniques
image: img/productExceptSelf.png
---

A solution using Horner's method, and dynamic programming.

---
**Problem:**

You have an array nums. We determine two functions to perform on nums. In both cases, n is the length of nums:

- `fi(nums) = nums[0] · nums[1] · ... · nums[i - 1] · nums[i + 1] · ... · nums[n - 1]`. (In other words, `fi(nums)` is the product of all array elements except the $i^{th}f$.)
- `g(nums) = f0(nums) + f1(nums) + ... + fn-1(nums)`.

Using these two functions, calculate all values of `f` modulo the given `m`. Take these new values and add them together to get `g`. You should return the value of `g` modulo the given m``.

**Example:**

For `nums = [1, 2, 3, 4]` and `m = 12`, the output should be
`productExceptSelf(nums, m) = 2`.

---

On its surface, this problem appears really easy. You just multiply all nums first. Then for each $f_i$, you divide this product by `nums[i]`. The problem is, multiplying large numbers, and division are both expensive operations. This causes the semi-naive solution to fail. All this is explained well in Code Fights's own blog article on the problem: http://blog.codefights.com/productexceptself-solution/

However, I want to discuss a much more elegant solution which makes use of Horner's method.

Horner's method allows us to efficiently evalutate polynomials. For example, if we wished to compute the value of $x^3 + 2x^2 - x + 5$ for some particular value of x, a naive solution would require 6 multiply ops. Horner's method on the other hand would work as follows:

We first rewrite the polynomial,

$$x^3 + 2x^2 - x + 5,$$

$$ = (x^2 + 2x - 1)x + 5,$$

$$ = ((x + 2)x - 1)x + 5.$$

Now we just need 2 multiply ops. In general, this method can reduce the number of multiplications required to compute the value of a polynomial. While Horner's method is pretty straightforward, it's not easy to see right away how we can use it for our problem. For that reason, let's write down a general version of our problem.

Assume the numbers in `nums` are represented by the variables $a, b, c, d$, and $e$. What we want then is,

$$abcd + abce + abde + acde + bcde.$$

Let's start using the distributive law to pull out variables.

$$abcd + abce + abde + acde + bcde,$$

$$ = abcd + (abc + abd + acd + bcd)e,$$

$$ = abcd + (abc + (ab + ac + bc)d)e,$$

$$ = abcd + (abc + (ab + (a + b)c)d)e.$$

At this point, some patterns start to emerge in the sum. Look at the products, for instance, starting from the innermost parentheses. They grow by one as you move out - $ab, abc, abcd$. What's more, look at the sum in the innermost parentheses. We multiply $(a + b)$ by $c$, the same variable we multiply $ab$ by. If any of this is confusing, let's focus on just $(ab + (a + b)c)$.

In the parentheses just outside this one, we need $abc$, and $(ab + ac + bc)$. Let's label $(ab + ac + bc)$ as $s_{i}$, and $abc$ as $p_{i}$ for sum and product respectively.

Observe how $s_{i} = p_{i-1} + s_{i-1} * c$, and $p_{i} = p_{i-1} * c$. You can easily see how this relation holds for all outer parentheses (with the $c$ changed with the appropriate variable).

So the general relation is:

```
s[i] = p[i-1] + s[i-1] * nums[i]
p[i] = p[i-1] * nums[i]
```
where nums is 1-indexed.

What about at the start? If we start off with `s[0] = 0`, and `p[0] = 1`, we get,

```
s[1] = p[0] + s[0] * nums[1] = 1 + 0 * a = 1
p[1] = p[0] * nums[1] = 1 * a = a
```

Perfect! Without further ado then, here's the code:

```python
def productExceptSelf(nums, m):
  s, p = 0, 1
  for num in nums:
    s = p + s*num
    p = p*num
  return s % m
```

It's a direct translation of our logical argument into code. We can further optimize this by making sure the sum never gets larger than `m` by taking the mod of both `s`, and `p` on every iteration of the loop. This works because of two fundamental results from the theory of congruences:

$$\text{If } a \equiv b\: (\text{mod } m), \text{and } c \equiv d\: (\text{mod } m),$$

$$\text{then, } a + b \equiv c + d\: (\text{mod } m),$$

and,

$$\text{If } a \equiv b\: (\text{mod } m), \text{and } c \equiv d\: (\text{mod } m),$$

$$\text{then, } a \cdot b \equiv c \cdot d\: (\text{mod } m).$$


So, $(a + b + c + \dots) \% m = (a \% m + b \% m + c \% m + \dots)$, and also $(a \cdot b \cdot \dots) \% m = (a \% m \cdot b \% m \cdot \dots)$. So we can use these two rules along with the obvious rule that $(a \% m) \% m = (a \% m)$, to never let `p` or `s` to get bigger than `m`.

```python
def productExceptSelf(nums, m):
  s, p = 0, 1
  for num in nums:
    s, p = (p + s*num) % m, p*num % m
  return s
```

And this is it! Our final version, and a solution to a hard problem in 5 lines of code.

The first takeaway from this post is that sometimes it helps to convert a particular problem into its more general algebraic form and use the tools we vhave at our disposal to solve it. The second takeaway is that the innocent looking operations of multiplication, and division, can in fact get very expensive and an understanding of computer architecture is important to know the limitations of the Random Access Machine model that we take for granted.