# Subarray Kadane's Patterns

This guide breaks down the core logic, state transitions, invariants, and clean pseudocode for three foundational contiguous subarray problems:

1. **Standard Kadane's Algorithm** (Maximum Subarray Sum)
2. **Kadane with State Machine** (Maximum Subarray Sum After One Operation)
3. **Kadane with Extremes Tracking** (Maximum Product Subarray)

---

## Pattern 1: Classic Kadane's Algorithm (Maximum Subarray Sum)

### 1. Core Intuition & Logic
At every index $i$, any valid contiguous subarray ending at $i$ either:
1. **Extends** an existing subarray that ended at index $i - 1$ (`current_sum + nums[i]`).
2. **Starts fresh** with just the current element `nums[i]`.

If the accumulated sum leading up to index $i - 1$ is negative ($< 0$), it acts as a penalty. Extending it would make the sum ending at $i$ strictly worse than simply restarting the subarray at $nums[i]$.

### 2. State Transition
Let $\text{dp}[i]$ be the maximum subarray sum ending at index $i$:
$$\text{dp}[i] = \max(\text{nums}[i],\; \text{dp}[i - 1] + \text{nums}[i])$$

Because $\text{dp}[i]$ only depends on $\text{dp}[i - 1]$, we compress the state to a single variable `current_sum`.

### 3. Pseudocode
```text
function maxSubArray(nums):
    current_sum = nums[0]
    global_max = nums[0]

    for i from 1 to length(nums) - 1:
        // Either extend or restart fresh
        current_sum = max(nums[i], current_sum + nums[i])
        global_max = max(global_max, current_sum)

    return global_max
```

### 4. Complexity & Key Edge Cases
- **Time Complexity:** $O(n)$ — Single linear pass.
- **Space Complexity:** $O(1)$ — Only two scalar tracking variables.
- **Edge Case (All Negatives):** E.g., `[-5, -2, -8]`. Initializing with `nums[0]` rather than `0` ensures the algorithm correctly returns the largest single negative number (`-2`).

---

## Pattern 2: Maximum Subarray Sum After One Operation

### 1. Core Intuition & Logic
You are allowed to square at most **one** element ($nums[j] \to nums[j]^2$) in your chosen subarray.

Because any integer $x$ satisfies $x^2 \ge x$, the optimal answer will always square **exactly one** element. At each index $i$, a subarray ending at $i$ can be in one of two distinct states:
1. **State $f$ (0 elements squared so far):** Standard Kadane state.
2. **State $g$ (1 element squared so far):** An element has already been squared either before index $i$ or at index $i$ itself.

### 2. State Transitions
- **State $f$ (No squares):**
  $$f' = \max(\text{nums}[i],\; f + \text{nums}[i])$$
- **State $g$ (One square):**
  A subarray ending at $i$ with exactly one squared element can be formed by:
  1. Starting a fresh subarray and squaring $nums[i]$ immediately: $\text{nums}[i]^2$
  2. Extending an un-squared subarray $f$ and squaring $nums[i]$ right now: $f + \text{nums}[i]^2$
  3. Extending an already-squared subarray $g$ and leaving $nums[i]$ unmodified: $g + \text{nums}[i]$

  $$g' = \max(\text{nums}[i]^2,\; f + \text{nums}[i]^2,\; g + \text{nums}[i])$$

> **Crucial Invariant:** When updating in $O(1)$ memory, $g$ must be computed using the **previous** iteration's $f$ value before $f$ is updated.

### 3. Pseudocode
```text
function maxSumAfterOperation(nums):
    // f: max subarray sum ending at current index with 0 operations used
    f = nums[0]
    // g: max subarray sum ending at current index with 1 operation used
    g = nums[0] * nums[0]
    
    global_max = g

    for i from 1 to length(nums) - 1:
        sq = nums[i] * nums[i]

        // Calculate transitions
        next_g = max(sq, f + sq, g + nums[i])
        next_f = max(nums[i], f + nums[i])

        g = next_g
        f = next_f

        global_max = max(global_max, g)

    return global_max
```

### 4. Complexity & Key Edge Cases
- **Time Complexity:** $O(n)$ — Single pass over the array.
- **Space Complexity:** $O(1)$ — Two scalar DP registers (`f`, `g`).
- **Edge Case (All Negatives):** Even if all elements are negative (e.g., `[-3, -4]`), squaring the best element turns it into a positive number (`(-3)^2 = 9`), which naturally maximizes the result.

---

## Pattern 3: Maximum Product Subarray

### 1. Core Intuition & Logic
Unlike addition, multiplication does not possess monotonic order:
- A large positive product multiplied by a negative number becomes a **very small negative product**.
- A small (large magnitude) negative product multiplied by a negative number flips into a **very large positive product**.
- A zero resets the contiguous product entirely.

Because negative numbers reverse inequalities ($\le$ becomes $\ge$), the maximum product ending at index $i$ can originate from the **minimum (most negative)** product ending at index $i - 1$. Therefore, we must track both bounds simultaneously.

### 2. State Transitions
Let:
- $\text{max\_prod}[i]$: Maximum product ending at index $i$.
- $\text{min\_prod}[i]$: Minimum product ending at index $i$.

At step $i$, consider the candidate set:
$$C = \{\text{nums}[i],\; \text{max\_prod}[i - 1] \times \text{nums}[i],\; \text{min\_prod}[i - 1] \times \text{nums}[i]\}$$

Transitions:
$$\text{max\_prod}[i] = \max(C)$$
$$\text{min\_prod}[i] = \min(C)$$

### 3. Pseudocode
```text
function maxProduct(nums):
    max_prod = nums[0]
    min_prod = nums[0]
    global_max = nums[0]

    for i from 1 to length(nums) - 1:
        x = nums[i]
        
        // If x is negative, max and min will swap roles upon multiplication
        if x < 0:
            swap(max_prod, min_prod)

        // Either restart fresh at x or extend the prior bound
        max_prod = max(x, max_prod * x)
        min_prod = min(x, min_prod * x)

        global_max = max(global_max, max_prod)

    return global_max
```

### 4. Complexity & Key Edge Cases
- **Time Complexity:** $O(n)$ — Constant operations per index.
- **Space Complexity:** $O(1)$ — Only `max_prod`, `min_prod`, and `global_max`.
- **Edge Cases:**
  - **Presence of 0:** `max(0, ...)` resets the chain to 0, correctly simulating subarray termination.
  - **Odd number of negatives:** E.g., `[2, 3, -2, 4]`. The swap keeps track of the prefix up to the negative number, while restarting handles fresh starts after the sign change.

---

## Summary Comparison Matrix

| Problem | Number of States | Key Insight | Main Transition |
| :--- | :--- | :--- | :--- |
| **Max Subarray Sum** | 1 (`sum`) | Negative prefixes hurt future sums; discard when `< 0`. | `sum = max(x, sum + x)` |
| **Sum After 1 Square** | 2 (`f`, `g`) | State machine: before operation ($f$) vs. after operation ($g$). | `g = max(x^2, f + x^2, g + x)` |
| **Max Product Subarray** | 2 (`maxi`, `mini`) | Negative signs invert orders; minimum negative becomes maximum positive. | `maxi = max(x, maxi*x, mini*x)` |
