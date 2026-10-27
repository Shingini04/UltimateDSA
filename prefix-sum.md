# Prefix Sum & Modulo Subarray Patterns

Quick-reference cheat sheet for the three foundational prefix sum hash map patterns.

---

## 1. Count Subarrays with Sum Equal to K

### Logic

If subarray `nums[j..i]` has sum $k$, then:


$$\text{prefix}[i] - \text{prefix}[j - 1] = k \implies \text{prefix}[j - 1] = \text{prefix}[i] - k$$


We track frequencies of each seen prefix sum.

### Pseudocode

```text
function subarraySum(nums, k):
    freq = hash_map()
    freq[0] = 1          // empty prefix sum
    running_sum = 0
    ans = 0

    for x in nums:
        running_sum += x
        target = running_sum - k
        if target in freq:
            ans += freq[target]
        freq[running_sum] += 1

    return ans

```

* **Time:** $O(n)$
* **Space:** $O(n)$

---

## 2. Count Subarrays with Sum Divisible by K

### Logic

A subarray sum is divisible by $k$ if:


$$(\text{prefix}[i] - \text{prefix}[j - 1]) \equiv 0 \pmod k \implies \text{prefix}[i] \equiv \text{prefix}[j - 1] \pmod k$$


Two prefix sums with the **same canonical remainder** form a valid subarray. In C++, negative numbers require normalizing remainders to $[0, k - 1]$ via `((rem % k) + k) % k`.

### Pseudocode

```text
function subarraysDivByK(nums, k):
    rem_count = array of size k initialized to 0
    rem_count[0] = 1     // prefix sum 0 has remainder 0
    running_sum = 0
    ans = 0

    for x in nums:
        running_sum += x
        rem = ((running_sum % k) + k) % k   // canonical positive remainder
        ans += rem_count[rem]
        rem_count[rem] += 1

    return ans

```

* **Time:** $O(n)$
* **Space:** $O(k)$

---

## 3. Longest Subarray with Sum Equal to K

### Logic

To maximize length $(i - j)$, find the **earliest possible index** $j$ where $\text{prefix}[j] = \text{prefix}[i] - k$.

* Only store `first_seen[prefix]` the very **first time** it appears.
* Length from index $j + 1$ to $i$ is $i - j$.

### Pseudocode

```text
function longestSubarray(nums, k):
    first_seen = hash_map()
    first_seen[0] = -1   // index before array starts
    running_sum = 0
    max_len = 0

    for i from 0 to n - 1:
        running_sum += nums[i]
        target = running_sum - k
        if target in first_seen:
            max_len = max(max_len, i - first_seen[target])
        
        // Record only the first occurrence to maximize span
        if running_sum not in first_seen:
            first_seen[running_sum] = i

    return max_len

```

* **Time:** $O(n)$
* **Space:** $O(n)$

---

## Summary Comparison

| Problem | Value Tracked in Map | Store On Conflict? | Base Case |
| --- | --- | --- | --- |
| **Count Sum = K** | `frequency` | Always increment (`count++`) | `mp[0] = 1` |
| **Count Divisible by K** | `frequency` (index $0 \dots k - 1$) | Always increment (`count++`) | `rem[0] = 1` |
| **Longest Sum = K** | `first_seen_index` | **Never overwrite** (keep smallest index) | `mp[0] = -1` |
