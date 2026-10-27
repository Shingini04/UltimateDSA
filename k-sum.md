# Two-Pointer & K-Sum Problem Suite

A comprehensive reference for the core **3Sum** pattern and its fundamental relatives (**Two Sum**, **4Sum**, **3Sum Smaller**, **Two Sum Less Than K**, and **Count Pairs in Two Arrays**).

---

## Table of Contents
1. [Core Problem: 3Sum](#1-core-problem-3sum)
2. [Related 1: Two Sum (Hash Map vs. Two Pointers)](#2-related-1-two-sum)
3. [Related 2: 4Sum (Generalizing K-Sum)](#3-related-2-4sum)
4. [Related 3: 3Sum Smaller](#4-related-3-3sum-smaller)
5. [Related 4: Two Sum Less Than K](#5-related-4-two-sum-less-than-k)
6. [Related 5: Count Pairs in Two Arrays](#6-related-5-count-pairs-in-two-arrays)
7. [Complexity & Pattern Summary Matrix](#7-complexity--pattern-summary-matrix)

---

## 1. Core Problem: 3Sum

### Problem Statement
Given an integer array `nums`, return all unique triplets `[nums[i], nums[j], nums[k]]` such that $i \ne j \ne k$ and $nums[i] + nums[j] + nums[k] = 0$.

### Core Intuition
- **Sorting first:** Sorting transforms an unstructured $O(n^3)$ search into an $O(n^2)$ two-pointer traversal and clusters duplicate elements together.
- **Fixing the first element:** For each index $i$, we treat $-nums[i]$ as the `target` for the remaining subarray `nums[i + 1 ... n - 1]`.
- **Duplicate handling:**
  - For the outer loop: skip $nums[i]$ if $nums[i] == nums[i - 1]$.
  - For the two pointers: once a valid triplet is found, advance `l` and decrement `r` past any duplicate elements before continuing.

### C++ Implementation
```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> ans;
        int n = nums.size();
        sort(nums.begin(), nums.end());

        for (int i = 0; i < n - 2; i++) {
            // Early break: if the smallest element is > 0, three elements cannot sum to 0
            if (nums[i] > 0) break;

            // Skip duplicate outer pivots
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            int target = -nums[i];
            int l = i + 1, r = n - 1;

            while (l < r) {
                int sum = nums[l] + nums[r];
                if (sum == target) {
                    ans.push_back({nums[i], nums[l], nums[r]});
                    
                    // Skip duplicates for second and third elements
                    while (l < r && nums[l] == nums[l + 1]) l++;
                    while (l < r && nums[r] == nums[r - 1]) r--;

                    l++;
                    r--;
                } 
                else if (sum < target) {
                    l++;
                } 
                else {
                    r--;
                }
            }
        }
        return ans;
    }
};
```

- **Time Complexity:** $O(n^2)$ — $O(n \log n)$ sort + $O(n^2)$ nested scans.
- **Space Complexity:** $O(1)$ auxiliary (ignoring sort overhead & return buffer).

---

## 2. Related 1: Two Sum

### Problem Statement
Given an array `nums` and target `target`, return the indices of the two numbers that add up to `target`.

### Core Intuition
- **Unsorted Indices needed:** A hash map lookup achieves optimal $O(n)$ time by mapping `value -> index` in a single pass.
- If only values or existence are needed, sorting + standard two pointers achieves $O(n \log n)$ time with $O(1)$ auxiliary space.

### C++ Implementation (Hash Map - $O(n)$)
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> seen;
        for (int i = 0; i < nums.size(); i++) {
            int complement = target - nums[i];
            if (seen.count(complement)) {
                return {seen[complement], i};
            }
            seen[nums[i]] = i;
        }
        return {};
    }
};
```

---

## 3. Related 2: 4Sum

### Problem Statement
Given an array `nums` and a target `target`, return all unique quadruplets `[nums[a], nums[b], nums[c], nums[d]]` summing to `target`.

### Core Intuition
- Extends 3Sum by adding one additional outer loop.
- **Critical Interview Optimization (Pruning):** Avoid useless branches by calculating the minimum possible sum (`nums[i] + nums[i+1] + ...`) and maximum possible sum (`nums[i] + nums[n-1] + ...`) at each level.
- Handle 64-bit integer overflow during calculations by casting sums to `long long`.

### C++ Implementation
```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> ans;
        int n = nums.size();
        if (n < 4) return ans;

        sort(nums.begin(), nums.end());

        for (int i = 0; i < n - 3; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            // Pruning 1: Smallest 4-sum is already too large
            if ((long long)nums[i] + nums[i+1] + nums[i+2] + nums[i+3] > target) break;
            // Pruning 2: Largest 4-sum with this pivot is still too small
            if ((long long)nums[i] + nums[n-3] + nums[n-2] + nums[n-1] < target) continue;

            for (int j = i + 1; j < n - 2; j++) {
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;

                if ((long long)nums[i] + nums[j] + nums[j+1] + nums[j+2] > target) break;
                if ((long long)nums[i] + nums[j] + nums[n-2] + nums[n-1] < target) continue;

                int l = j + 1, r = n - 1;
                while (l < r) {
                    long long sum = (long long)nums[i] + nums[j] + nums[l] + nums[r];
                    if (sum == target) {
                        ans.push_back({nums[i], nums[j], nums[l], nums[r]});
                        while (l < r && nums[l] == nums[l + 1]) l++;
                        while (l < r && nums[r] == nums[r - 1]) r--;
                        l++;
                        r--;
                    } 
                    else if (sum < target) {
                        l++;
                    } 
                    else {
                        r--;
                    }
                }
            }
        }
        return ans;
    }
};
```
- **Time Complexity:** $O(n^3)$
- **Space Complexity:** $O(1)$ auxiliary space.

---

## 4. Related 3: 3Sum Smaller

### Problem Statement
Given an array `nums` of $n$ integers and a `target`, find the **number of index triplets** `(i, j, k)` with $0 \le i < j < k < n$ that satisfy $nums[i] + nums[j] + nums[k] < target$.

### Core Intuition
- Sort the array.
- For each fixed index $i$, initialize pointers $l = i + 1$ and $r = n - 1$.
- If `nums[i] + nums[l] + nums[r] < target`, then because the array is sorted, replacing `r` with **any** index from $l + 1$ up to $r$ will also produce a sum $< target$.
- Therefore, we can batch-count all $(r - l)$ pairs instantly and advance `l++`.

### C++ Implementation
```cpp
class Solution {
public:
    int threeSumSmaller(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        int count = 0;
        int n = nums.size();

        for (int i = 0; i < n - 2; i++) {
            int l = i + 1, r = n - 1;
            while (l < r) {
                if (nums[i] + nums[l] + nums[r] < target) {
                    // All pairs (l, l+1), (l, l+2), ..., (l, r) are valid
                    count += (r - l);
                    l++;
                } else {
                    r--;
                }
            }
        }
        return count;
    }
};
```
- **Time Complexity:** $O(n^2)$
- **Space Complexity:** $O(1)$ auxiliary space.

---

## 5. Related 4: Two Sum Less Than K

### Problem Statement
Given an array `nums` and an integer $k$, return the maximum sum $S$ such that there exists $i < j$ with $nums[i] + nums[j] = S$ and $S < k$. If no such $(i, j)$ exists, return $-1$.

### Core Intuition
- Sort the array and place two pointers at ends: $l = 0, r = n - 1$.
- If `nums[l] + nums[r] < k`:
  - It is a candidate answer. Record `ans = max(ans, nums[l] + nums[r])`.
  - Increment `l++` to search for an even larger sum that is still below $k$.
- If `nums[l] + nums[r] >= k`:
  - The sum is too large; decrement `r--`.

### C++ Implementation
```cpp
class Solution {
public:
    int twoSumLessThanK(vector<int>& nums, int k) {
        sort(nums.begin(), nums.end());
        int l = 0, r = nums.size() - 1;
        int max_sum = -1;

        while (l < r) {
            int sum = nums[l] + nums[r];
            if (sum < k) {
                max_sum = max(max_sum, sum);
                l++;
            } else {
                r--;
            }
        }
        return max_sum;
    }
};
```
- **Time Complexity:** $O(n \log n)$
- **Space Complexity:** $O(1)$ auxiliary space.

---

## 6. Related 5: Count Pairs in Two Arrays

### Problem Statement
Given two integer arrays `nums1` and `nums2` of length $n$, count the pairs of indices $(i, j)$ such that $i < j$ and:
$$nums1[i] + nums1[j] > nums2[i] + nums2[j]$$

### Core Intuition
Rearrange the inequality by grouping terms by index:
$$(nums1[i] - nums2[i]) + (nums1[j] - nums2[j]) > 0$$

Define a difference array $D[k] = nums1[k] - nums2[k]$. The problem reduces to:
$$\text{Find pairs } (i, j) \text{ with } i < j \text{ such that } D[i] + D[j] > 0$$

Since the condition depends strictly on the sum of two values, original index order $i < j$ simply prevents self-pairing and duplicate orientation. Sorting $D$ preserves all combinations:
- If $D[l] + D[r] > 0$, every element between $l$ and $r - 1$ paired with $D[r]$ will also sum to $> 0$.
- Add $(r - l)$ to the count and move $r--$.
- Otherwise, $D[l]$ is too negative; advance $l++$.

### C++ Implementation
```cpp
class Solution {
public:
    long long countPairs(vector<int>& nums1, vector<int>& nums2) {
        int n = nums1.size();
        vector<int> diff(n);
        for (int i = 0; i < n; i++) {
            diff[i] = nums1[i] - nums2[i];
        }

        sort(diff.begin(), diff.end());

        long long count = 0;
        int l = 0, r = n - 1;

        while (l < r) {
            if (diff[l] + diff[r] > 0) {
                // All indices from l to r-1 paired with r are valid
                count += (r - l);
                r--;
            } else {
                l++;
            }
        }
        return count;
    }
};
```
- **Time Complexity:** $O(n \log n)$ due to sorting.
- **Space Complexity:** $O(n)$ for the difference array.

---

## 7. Complexity & Pattern Summary Matrix

| Problem | Target Relation | Primary Pattern | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- | :--- |
| **Two Sum** | $A[i] + A[j] == \text{target}$ | Hash Map / Pointers | $O(n)$ | $O(n)$ |
| **Two Sum Less Than K** | $A[i] + A[j] < K$ (maximize) | Sort + Two Pointers | $O(n \log n)$ | $O(1)$ |
| **Count Pairs in Two Arrays** | $(A_1[i]-A_2[i]) + (A_1[j]-A_2[j]) > 0$ | Algebraic reduction + Two Pointers | $O(n \log n)$ | $O(n)$ |
| **3Sum** | $A[i] + A[j] + A[k] == 0$ | Sort + Fix 1 + Two Pointers + Duplicate skips | $O(n^2)$ | $O(1)$ |
| **3Sum Smaller** | $A[i] + A[j] + A[k] < \text{target}$ | Sort + Fix 1 + Range Counting $(r - l)$ | $O(n^2)$ | $O(1)$ |
| **4Sum** | $A[a] + A[b] + A[c] + A[d] == \text{target}$ | Sort + Fix 2 + Two Pointers + Pruning | $O(n^3)$ | $O(1)$ |
