---
layout: post
title:  "countInversions"
date:   2017-12-21
categories: CommonTechniques
image: img/countInversions.png
---

If you need a hint, think of sorting algorithms. Once you're done, come check out a simple solution.

---
**Problem:**

The inversion count for an array indicates how far the array is from being sorted. If the array is already sorted, then the inversion count is `0`. If the array is sorted in reverse order, then the inversion count is the maximum possible value.

Given an array `a`, find its inversion count. Since this number could be quite large, take it modulo $10^9 + 7$.

**Example:**

For `a = [3, 1, 5, 6, 4]`, the output should be countInversions(a) = `3`.

The three inversions in this case are: `(3, 1), (5, 4), (6, 4)`.

---

In this problem you're basically asked to enumerate the pairs of numbers that are out of (sorted) order. So, given an array `[2, 1, 3]`, the answer would be 1. This is because only `2`, and `1` are out of order. But if the array were to be `[3, 2, 1]`, the answer would be 3, since we have 3 out of order pairs: `(3, 2)`, `(3, 1)`, and `(2, 1)`.

A brute force solution would be to look at every pair `p` of numbers in the array and increment the count by one if `p[0] > p[1]`. This will work, but obviously we're gonna do better.

Before we begin developing our solution, observe the following. The number of inversions in a sorted array are 0. Also, in all comparison based sorting algorithms, we compare and/or swap items. So if we could modify an efficient sorting algorithm to count the number of inversions while sorting the array, we'll get our answer when it's done sorting and the time complexity will be $O(n\;log\;n)$. 

The sorting algorithm we'll use is merge sort. It's guaranteed $O(n\;log\;n)$, and easy to implement. Let's start writing mergesort and try out something arbitrary first.

```python
def countInversions(a):
  return mergesort(a)[1] % 1000000007
```

Here I'm returning `mergesort(a)[1]` because this modified mergesort will return both the sorted array, and the number of inversions. Before writing the `mergesort` function let's think about it. In mergesort we usually divide the array into two halves - left, and right - and then call mergesort on them recursively. Since normal `mergesort` just returns the sorted list, we receive the sorted left and right arrays which we then merge into the final sorted array and return.

Now, we also care about the number of inversions. So how about we assume our calls to `mergesort` on the left, and right arrays return not only sorted arrays but also the number of inversions counted in those arrays. Then we can count the number of inversions detected while merging the two arrays and sum them up with the inversions we had from the left and right arrays and return this as the final result. 

That's a lot of words, let's just understand what this means in code.

```python
def countInversions(a):
  return mergesort(a)[1] % 1000000007

def mergesort(a):
  if len(a) < 2: return a, 0
  left,  linvs = mergesort(a[:len(a)//2])
  right, rinvs = mergesort(a[len(a)//2:])
  res, invs = merge(left, right)
  return res, linvs + invs + rinvs
```

Okay, so that's all those words in code. For the base case, if the array is empty or just contains a single element then it's already sorted and no inversions are required. So we return 0.

```python
def countInversions(a):
  return mergesort(a)[1] % 1000000007

def mergesort(a):
  if len(a) < 2: return a, 0
  left,  linvs = mergesort(a[:len(a)//2])
  right, rinvs = mergesort(a[len(a)//2:])
  res, invs = merge(left, right)
  return res, linvs + invs + rinvs

def merge(left, right):
  res, invs = [], 0
  while left and right:
    if left[0] <= right[0]: res += [left.pop(0)]
    else:
      # TODO: count inversion
  return res + left + right, invs
```

Alright, so the `merge` function so far is pretty standard. We are gonna continue to loop until either `left`, or `right` becomes empty. Then we simply append the non-empty list to the result, and return `res`, and `invs`. The `res + left + right` is a very elegant pythonic way of writing the merge function which removes the need to explicity loop through the non-empty list and append each of its elements.

Anyways, the main logic will be in the `else` case, which indicated that we found an out of order pair of elements.

```python
def countInversions(a):
  return mergesort(a)[1] % 1000000007

def mergesort(a):
  if len(a) < 2: return a, 0
  left,  linvs = mergesort(a[:len(a)//2])
  right, rinvs = mergesort(a[len(a)//2:])
  res, invs = merge(left, right)
  return res, linvs + invs + rinvs

def merge(left, right):
  res, invs = [], 0
  while left and right:
    if left[0] <= right[0]: res += [left.pop(0)]
    else:
      res += [right.pop(0)]
      invs += 1
  return res + left + right, invs
```

After doing appending the first element of `right` to `res`, we could try incrementing `invs` by one as a guess. But this will not work. Here's an example of why. Suppose you were merging the arrays:

```
left = [3, 4, 5]
right = [1, 6, 7]
```
Our procedure would return 1, but clearly all three elements of `left` are out of order with `right[0] = 1`. This is a problem with mergesort because it can swap elements that are far away (not next to each other), unlike, say, bubble sort. So, was our choice of sorting algorithm wrong? 

No, we can easily get around it by adding not `1`, but rather the length of the left array to `invs`. Note how this fits nicely in our implementation where we have used `list.pop()` instead of using pointers. 

So, here's the final code:

```python
def countInversions(a):
  return mergesort(a)[1] % 1000000007

def mergesort(a):
  if len(a) < 2: return a, 0
  left,  linvs = mergesort(a[:len(a)//2])
  right, rinvs = mergesort(a[len(a)//2:])
  res, invs = merge(left, right)
  return res, linvs + invs + rinvs

def merge(left, right):
  res, invs = [], 0
  while left and right:
    if left[0] <= right[0]: res += [left.pop(0)]
    else:
      res += [right.pop(0)]
      invs += len(left)
  return res + left + right, invs
```

A simple solution using an elegant implementation of mergesort.