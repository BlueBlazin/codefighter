---
layout: post
title:  "knighOnBoardProbability"
date:   2017-12-23
categories: Counting
image: img/knightOnBoard.png
---

An elegant dynamic programming based top-down solution.

---
**Problem:**

You have a chessboard with `n` rows and `m` columns. The knight is placed on the (`x`, `y`) cell (`0`-indexed). Find the probability that the knight stays on the board after performing the usual knight move `steps` times randomly. If the knight gets moved off of the board, it can't return to the board again. For example, after the move `(0, 0) - (-1, -2)`, it's impossible for the knight to return on the board by performing the move `(-1, -2) - (0, 0)`.

**Example:**

For `n = 8`, `m = 8`, `steps = 2`, `x = 4` and `y = 4`, the output should be

`knightOnBoardProbability(n, m, steps, x, y) = 0.875`.

---

In this problem we are given as starting location, and a number of steps. Our knight will randomly move to a valid square `steps` number of times, and we're asked to compute the probability that the knight is on the board after the `steps` number of random jumps. We're also told that if the knight happens to move off the board, it's stuck there and cannot return.

![knight movements](https://i.imgur.com/8W2b19T.png?1){: .custom-post-image .custom-image-scale}
*Possible knight moves.*{: .custom-image-info}

Because squares which are off the board are allowed as valid moves, the probability for jumping to any of the eight (possible outside the board) squares is $\frac{1}{8}$. As the probability is equal, our problem of finding the total probability reduces to enumerating the number of possible positions after `steps` moves which keep the knight on the board, and using it to calculate our answer.

We'll use recursion as our main strategy, and recursively add up all probabilities to get the final answer. If you think long enough, or you have enough experience with such problems that you readily identify these patterns, you'll notice that **many of the recursive calls will be identical**.

You can see this without even thinking about the structure of the recursive solutions. For example, say our initial position is the same as that of the black knight in the above figure, and that `steps` equals 5. 

Now suppose we moved two squares east and one square north, then moves back to our starting square. We'll have 3 steps remaining. On the other hand, suppose we moves two squares west and one square north instead, and then moves back to our original starting square. We'll still have 3 steps remaining and we're our state is identical to what we got from moving right and then back.

This overlap in subproblems means **we're going to use Dynamic Programming** to solve this problem. More specifically, we'll use the top-down variant of dynamic programming which uses memoization.

The wall of text above demands that we start writing code without any more delay.

```python
def knightOnBoardProbability(m, n, steps, x, y):
  p = 0
  # TODO: We'll recurse here.
  return p
```

Okay so we're declaring `p` which is the probability we want to return. Between that and the return statement, we need to loop over all next squares the knight can hop on to, and then add the probability returned from them to `p`.

```python
def knightOnBoardProbability(m, n, steps, x, y):
  p = 0
  for i, j in [(-2, -1), (-2, 1), (-1, -2), (-1, 2)]:
    p += knightOnBoardProbability(m, n, steps-1, x+i, y+j) * 1/8
    p += knightOnBoardProbability(m, n, steps-1, x-i, y-j) * 1/8
  return p
```

There's eight different moves the kniht can make but I've simplified the loop since half of those are mirror moves. What exactly is happening with the probability calculation? 

We're asking the recursive call to give us the probability that the knight will remain on board after making random moves `steps - 1` number of times. We're also doing this with the condition that our move on the current step was $(x + i, y + j)$. So we get,

$$P(\text{knightOnBoard after s steps}) = \sum_{m \in \text{valid moves}} P(\text{knightOnBoard after s-1 steps}) \cdot P(m).$$

Now, since the probability of moving to any particular square is always $\frac{1}{8}$, the above equation just reduces to,

$$P(\text{knightOnBoard after s steps}) = \sum_{m \in \text{valid moves}} P(\text{knightOnBoard after s-1 steps}) / 8.$$

Okay, that takes care of the math, now we need to handle the base cases.

```python
def knightOnBoardProbability(m, n, steps, x, y):
  if x < 0 or x >= m: return 0 # check if we moved off the board
  if y < 0 or y >= n: return 0 # we can never get back on if we move off the board
  if steps == 0: return 1.0
  p = 0
  for i, j in [(-2, -1), (-2, 1), (-1, -2), (-1, 2)]:
    p += knightOnBoardProbability(m, n, steps-1, x+i, y+j) * 1/8
    p += knightOnBoardProbability(m, n, steps-1, x-i, y-j) * 1/8
  return p
```

At the start of every call we check if we moved off the board. If that happens, regardless of the number of moves still left, we're done, and we return `0`. If we did pass that check, we check if `steps == 0`. This means we survived this specific sequence of moves and we can return `1` as the probability. Thee's just one important thing missing here. I said earlier that we'll solve this problem using dynamic programming but where's the DP?

The beauty of memoization is that you add the memoization checks right at the end, after you've finished implementing the recursive calls in an intuitive top-down manner.

```python
memo = {}

def knightOnBoardProbability(m, n, steps, x, y):
  if x < 0 or x >= m: return 0
  if y < 0 or y >= n: return 0
  if steps == 0: return 1.0
  if (x, y, steps) in memo: return memo[(x, y, steps)]
  p = 0
  for i, j in [(-2, -1), (-2, 1), (-1, -2), (-1, 2)]:
    p += knightOnBoardProbability(m, n, steps-1, x+i, y+j) * 1/8
    p += knightOnBoardProbability(m, n, steps-1, x-i, y-j) * 1/8
  memo[(x, y, steps)] = p
  return p
```

There you go, a succinct and intuitive implementation using a top-down approach. We solved this hard rated problem in just 13 lines of code, and it's still easy to read, and understand. As an extra challenge, try improving the memoization so it's more efficient.