---
layout: post
title:  "calculateBasins"
date:   2017-12-24
categories: Geometry
image: img/calculateBasins.png
---

A problem which looks harder than it is. Try solving it on your own then check out this simple solution.

---
**Problem:**

A group of farmers has some elevation data that we are going to use to help them understand how rainfall flows over their farmland. We represent the farmland as a 2D array of altitudes, the `grid`, and use the following model, based on the fact that water flows downhill:

- If a cell's four neighboring cells all have altitudes not lower that its own, this cell is a **sink** in which water collects.
- Otherwise, water will flow into the neighboring cell with the lowest altitude. If a cell is not a sink, you can assume it has a unique lowest neighbor and that this neighbor will be lower than the cell.
- Cells that drain into the same sink, directly or indirectly, are part of the same **basin**.

Given an `n × n` `grid` of elevations, your goal is to partition the map into basins and output the sizes of the basins, in descending order.

**Example:**

For
```
grid = [[1, 5, 2], 
        [2, 4, 7], 
        [3, 6, 9]]
```

the output should be `calculateBasins(grid) = [7, 2]`.

The two basins in this map, labeled as `A`s and `B`s, are:
```
  A A B 
  A A B 
  A A A 
```

---

This is a very interesting problem for two reasons. The first reason is that it looks deceptively difficult. The second reason is because we'll use the seldom used deform-and-conquer technique to solve it.

The problem is asking us to find the sizes of these "basins", and it seems obvious that the first step would be to calculate them. But how do we even begin? Should we find the minimum element and add it to its own basin. Then move on to its neighbors and try to figure out their basins and so on?

Well, we don't really need to follow any order at all. Like I said, the problem looks like it might require some clever algorithm on graphs but if you stare at this for long enough:

```
  A A B 
  A A B 
  A A A 
```

you'll realize this looks an awful lot like connected components. Before we try to think about the structure of the code let's take a moment to understand why a simple connected components based solution might work.

Say we were considering the cell $(2, 3)$. Its flow will depend **only** on its immediate neighbors, namely, $(3, 3), (1, 3), (2, 4), (2, 1)$. Thus as long as each cell points to its lowest altitude neighbor (or itself if it's a local minima), it doesn't matter how we order the processing of each cell. The problem is local by nature.

But there's a data structure built exactly for this purpose, and it's union-find. The data structure comes with two operations - union, and find. Union takes two arguments and joins their connected components into one. Find returns the value (id) of the connected component its argument belongs to.

Here's an array based implementation which requires us to know the number of elements beforehand.

```python
class CC:
  def __init__(self, n):
    self.parents = list(range(n))
  
  def find(self, u):
    if self.parents[u] == u: return u
    return self.find(self.parents[u])

  def union(self, u, v):
    self.parents[self.find(v)] = self.find(u)
```

It's incredibly simple but I should mention, this is a naive implementation. It can get very inefficient depending on the union operations made. However, for our purpose it's sufficient.

There's just one problem. Our problem is on a 2D grid, whereas the way we're written this data structure seems to require a single dimensional array. I guess we could use tuples with cell locations and a two dimensional array instead. But instead let's deform the problem into a 1d problem which will leave our union-find data structure unchanged and require little change. We can convert a 2d index into a 1d one by using this formula: `i * number_of_columns + j`.

Let's start writing the code.

```python
def calculateBasins(grid):
  n = len(grid)

  for i in range(n):
    for j in range(n):
      # TODO: get the minimum neighbor and union
```

We'll visit each cell in order using a double for loop. For each cell, we'll find its minimum neighbor (could be itself) and then union that cell with the current one. Once we have our connected components we'll figure out how to their counts. For now that's the game plan.

```python
def calculateBasins(grid):
  n, basins = len(grid), [u for u in range(len(grid)**2)]

  def find(u): return u if basins[u] == u else find(basins[u])
  def union(u, v): basins[find(v)] = find(u)

  for i in range(n):
    for j in range(n):
      mini, minj = get_min(grid, i, j, n)
      union(i*n + j, mini*n + minj)

def get_min(grid, i, j, n):
  mini, minj = i, j
  if i > 0   and grid[i-1][j] < grid[mini][minj]: mini, minj = i - 1, j
  if i < n-1 and grid[i+1][j] < grid[mini][minj]: mini, minj = i + 1, j
  if j > 0   and grid[i][j-1] < grid[mini][minj]: mini, minj = i, j - 1
  if j < n-1 and grid[i][j+1] < grid[mini][minj]: mini, minj = i, j + 1
  return mini, minj
```

Basic stuff so far. We're just checking each of the four neighbors to see if they have a value less that the current cell. If so we return that as the minimum valued neighbor, and that's where the water will flow. So we call union on the current cell and the location of this minimum valued cell. I've also added the union find code in the `calculateBasins` function itself instead of defining a separate class. Also, what was `parents` in the class definiton for union-find above is now named `basins`.

All that's left now is to return the answer. Instead of writing our own code, for this we'll use python's Counter from the collections module.

```python
from collections import Counter as cntr

def calculateBasins(grid):
  n, basins = len(grid), [u for u in range(len(grid)**2)]

  def find(u): return u if basins[u] == u else find(basins[u])
  def union(u, v): basins[find(v)] = find(u)

  for i in range(n):
    for j in range(n):
      mini, minj = get_min(grid, i, j, n)
      union(i*n + j, mini*n + minj)
  return sorted(cntr(find(u) for u in range(n*n)).values(), reverse=True)

def get_min(grid, i, j, n):
  mini, minj = i, j
  if i > 0   and grid[i-1][j] < grid[mini][minj]: mini, minj = i - 1, j
  if i < n-1 and grid[i+1][j] < grid[mini][minj]: mini, minj = i + 1, j
  if j > 0   and grid[i][j-1] < grid[mini][minj]: mini, minj = i, j - 1
  if j < n-1 and grid[i][j+1] < grid[mini][minj]: mini, minj = i, j + 1
  return mini, minj
```

And just like that, in one fell swoop, we've finished off our code for this problem. There's some function nesting going on here so I'll walk you through it. First, we're using a list comprehension as an argument to the `cntr` function call. This is giving us a list (a generator) of id's of the connected components each cell belongs to. The counter is simply counting the number of occurences of each element in the list and storing it in a dictionary. So we get those numbers using `dict.values()`, and finally we return a list of these values, sorted in descending order (using the `reverse` flag). That's it! For extra challenge, try optimizing the union find data structure.