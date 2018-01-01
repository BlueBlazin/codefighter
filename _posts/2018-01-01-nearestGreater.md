---
layout: post
title:  "nearestGreater"
date:   2018-01-01
categories: HeapsStacksQueues
image: img/nearestGreater.png
---

If you can't do linear time, try solving this in quadratic time. Then come back for a linear time solution.

---
**Problem:**

Given an array of integers `a`, return a new array `b` using the following guidelines:

- For each index `i` in `b`, the value of `bᵢ` is the index of the `aⱼ` nearest to `aᵢ` and is also greater than `aᵢ`.
- If there are two options for `bᵢ`, put the leftmost one in `bᵢ`.
- If there are no options for `bᵢ`, put `-1` in `bᵢ`.

**Example:**

For a = `[1, 4, 2, 1, 7, 6]`, the output should be `nearestGreater(a) = [1, 4, 1, 2, -1, 4]`.

---

This problem is a generalization of similar problems which only ask for the nearest greater element along a single direction. Now you must find the index of the element closest in distance and greater in value to the current element and return a list `b` of such indices for each element of `a`.

You could solve this problem in $O(n^2)$ time complexity by scanning the list in both directions for each element and then finding the nearest greater element that way for each element in the list.

But we can, and will, do better. We'll solve this problem in $O(n)$ time but the solution is a little tricky to wrap your head around. So, let's first talk about it for a bit before diving into the code.

The first flash of insight we need is to recognize that we don't need to store the correct answer the first time we see an element. This sounds a little contradictory to our $O(n)$ claim so let me elaborate.

Let's say we have access to some blackbox which, when given an index from `a`, will either return the index of the nearest element greater than `a` to the **left** of `a`, or `-1` if this doesn't exist. Formal language always helps so,

$$
f(i) = \begin{cases}
  j\:\: |\: j = \max\limits_{a[k] > a[i],\: k \in 0 \dots i-1}(k) \:\text{ if exists}, \\
  -1 \text{ otherwise. }
\end{cases}
$$

The objective of the problem is to find the nearest larger element to the left, and the nearest larger to the right. Then see which one of these is nearer and place its index in `b`.

Assuming we found the nearest greater to the left of the element (even if it's `-1`) we can move on and find nearest greater to the right at a later point in time. Here's how you do that.

First you put this element on a stack, then as you're iterating over the elements, you compare their values to the top of the stack. If the top of the stack element has a smaller value than the current element, we pop it off the stack and process it. Then the current element must be the nearest greater of this element to its right. Think about it, if there was any other element greater than this element we just popped off the stack and appeared before in `a` than our current element, then we would've popped from our stack on that round of the loop. So we know the current element is the nearest greater to the right for the element we just popped off the stack. We'll pop off all elements on the stack which have greater value that the current.

Since the objective is to find the nearest greater from the left, and from the right, having found both -- nearest left using the black box and nearest right using the stack, we can solve the problem for that particular element. Similarly we will do so for every element in `a`, and the fill out all of `b`.

**If you skipped the wall of text, or didn't fully understand it**, that's okay. Maybe the code below will help clear things. Like I mentioned before, this one is tricky.

```python
def nearestGreater(a):
  b, stack = [-1] * len(a), []
```

So right off the bat, we're gonna declare the aforementioned stack on which we'll push indices of all elements we've visited so far in order to, perhaps at a later point in time, find their nearest greater element to the right.

An implementation detail I didn't mention before was that we'll be pushing indices onto the stack, not the elements itself. We also declare `b` and initialize it to all `-1`s.

Now, as promised the solution is $O(n)$ so we're of course going to loop over all elements in `a` once. Here's how that'll look like in code.

```python
def nearestGreater(a):
  b, stack = [-1] * len(a), []
  for i in range(len(a)):
    while stack and a[stack[-1]] < a[i]:
      # TODO: write logic to find nearest greater to right
    # TODO: implement blackbox to find nearest greater to left
```

So, we're obviously going to be popping off entries from the stack if the element they represent is less than the current element. We'll also assume that all elements in `b[0:i]` currently hold the value of the nearest greater to the left element's index. We'll delay implementing the 'blackbox' for finding the nearest greater element on the left till later when it'll make most sense.

```python
def nearestGreater(a):
  b, stack = [-1] * len(a), []
  for i in range(len(a)):
    while stack and a[stack[-1]] < a[i]:
      j = stack.pop()
      # compare nearest greater left with nearest greater right
      if b[j] == -1 or i - j < j - b[j]: b[j] = i
    # TODO: implement blackbox to find nearest greater to left
```

Here we're comparing the two nearest greater values we've found for `j`, and based on the result, placing the answer in `b[j]`. Remember `b[j]` might already have had the index of the nearest greater element of `a[j]` to its left. If this were true (in which case `b[j] != -1`), and this index was also nearer to the nearest greater element of `a[j]` to its right, then we keep thing as is. Else we assign `b[j]` the index `i`.

Let's finally implement the last part.

```python
def nearestGreater(a):
  b, stack = [-1] * len(a), []
  for i in range(len(a)):
    while stack and a[stack[-1]] < a[i]:
      j = stack.pop()
      if b[j] == -1 or i - j < j - b[j]: b[j] = i
    if not stack: b[i] = -1
    else: b[i] = stack[-1] if a[i] != a[stack[-1]] else b[stack[-1]]    
```

Here we've implemented the blackbox for finding the nearest greater element to the left. It's just whatever is remaining on the stack after popping off all elements less than the current element in the `while` loop. 

Think about that for a minute, it will make perfect sense! The top element on the stack which still remained is the nearest element which appeared in `a` before `a[i]` and was **greater or equal** to `a[i]`.

However equal doesn't cut it, it must be strictly greater. There's an easy fix to that, if the top element on the stack is equal in value to the current element, we just return its nearest greater on the left instead! That's just its current value on `b`, since it hasn't been popped off the stack yet. This value might very well be `-1`, but that's okay it just means our current element doesn't have a nearest greater on the left.

We also need to check if there's anything on the stack at all, if not we just set `b[i]` to `-1`.

```python
def nearestGreater(a):
  b, stack = [-1] * len(a), []
  for i in range(len(a)):
    while stack and a[stack[-1]] < a[i]:
      j = stack.pop()
      if b[j] == -1 or i - j < j - b[j]: b[j] = i
    if not stack: b[i] = -1
    else: b[i] = stack[-1] if a[i] != a[stack[-1]] else b[stack[-1]]    
    stack.append(i)
  return b
```

Finally, we push the current element on top of the stack, and return `b` after the `for` loop is done.

This is perhaps one of the trickier challenges on CodeFights and deserves to be a 'Hard' rated problem. I recommend going over the code multiple times until it clicks and you grasp exactly what's happening. Good luck!