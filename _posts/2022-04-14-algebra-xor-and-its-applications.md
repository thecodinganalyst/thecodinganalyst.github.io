---
title: "Algrebra, XOR, and its applications"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Algebra
  - XOR
  - binary operations
  - interview questions
---

Recently, I chanced upon this neat solution to a rather simple interview question, that exploits the special property of the binary XOR operation. It leads me back to the wonderful world of algebra, how the proofing of an algebraic property can have such rather miraculous application. 

The question goes - `An array of integers contains all pairs of the same number, except for 1 number. Spot the exception`. For example, if the array is `A[7] = [1, 3, 3, 2, 4, 2, 1]`, the answer should be 4.

My immediate solution was to iterate through the array and put each number encountered into a Set if the number is not already in the Set, and remove the number from the Set if it can be found in it. It works, but it didn't scale well enough for the time limitation set for this question to run on a huge array. So in the spirit of learning from others, I tried to google for a better solution and found this neat explanation - [https://florian.github.io/xor-trick/](https://florian.github.io/xor-trick/), so I'm jotting it here for my memory safekeeping.

### 1. XOR operation

```
|-----|-----|-------|
|  x  |  y  | x ^ y |
|-----|-----|-------|
|  0  |  0  |   0   |
|  0  |  1  |   1   |
|  1  |  0  |   1   |
|  1  |  1  |   0   |
|-----|-----|-------|
```

### 2. XOR is commutative, `x ^ y = y ^ x`

```
|-----|-----|-------|-------|
|  x  |  y  | x ^ y | y ^ x |
|-----|-----|-------|-------|
|  0  |  0  |   0   |   0   |
|  0  |  1  |   1   |   1   |
|  1  |  0  |   1   |   1   |
|  1  |  1  |   0   |   0   |
|-----|-----|-------|-------|
```

and it also implies, `(x ^ y) ^ z = (z ^ y) ^ x = y ^ (x ^ z)`,or basically I can move the brackets around and it won't affect the result

```
|-------|-----|-----------|-----------|
| x ^ y |  z  | x ^ y ^ z | z ^ y ^ x |
|-------|-----|-----------|-----------|
|   0   |  0  |     0     |     0     |     
|   0   |  1  |     1     |     1     |
|   1   |  0  |     1     |     1     |
|   1   |  1  |     0     |     0     |
|-------|-----|-----------|-----------|
```

### 3. XOR on the same number gives 0, meaning if I XOR the same number twice (or any even times), it will remove it

```
|-------|-------|-------|
|   x   |   y   | x ^ y |
|-------|-------|-------|
| **0** | **0** | **0** |
|   0   |   1   |   1   |
|   1   |   0   |   1   |
| **1** | **1** | **0** |
|-------|-------|-------|
```

### 4. Then I can deduce, `x ^ y ^ z ^ x ^ y = z`.

Because from #3, when I XOR the same number twice, it will be removed. So having x and y occurring twice in the equation removes it. And it doesn't matter what order is it, whether it is `x ^ z ^ y ^ y ^ x` or `z ^ x ^ x ^ y ^ y`, or any other combinations. 

## Solution

With the above deduction, I can solve for the unique number just by XOR-ing all the numbers in the array, assuming the length of A will never be 0. 

```
class Solution {
    public int solution(int[] A) {
        int cache = A[0];
        for (int i = 1; i < A.length; i ++){
            cache = cache ^ A[i];
        }
        return cache;
    }
}
```

