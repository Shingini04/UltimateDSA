# Consecutive Sequence Patterns

A structured guide covering the logic, invariants, optimal approaches, and pseudocode for three classic consecutive number problems:

1. **Longest Consecutive Sequence in an Array**
2. **Check if an Array is Consecutive**
3. **Hand of Straights** (Alice's Cards Partitioning)

---

## 1. Longest Consecutive Sequence in an Array

Given an unsorted integer array `nums`, return the length of the longest sequence of consecutive elements (e.g., $[1, 2, 3, 4]$ has length 4).

### Core Intuitions

#### Approach A: Sorting (Your Implementation)
- **Idea:** Sort the array first. Consecutive elements will sit next to each other.
- **Handling Duplicates:** If $nums[i] == nums[i-1]$, simply skip without resetting the sequence length.
- **Complexity:** $O(n \log n)$ time, $O(1)$ auxiliary space.
- **Edge Case:** Empty array ($n = 0$) must return $0$.

#### Approach B: Hash Set (Optimal Interview Expected — $O(n)$ Time)
- **Key Insight:** A number $x$ can only be the **start** of a consecutive streak if $x - 1$ is **NOT** in the set.
- If $x - 1$ exists, we do nothing because the sequence containing $x$ will be (or has been) explored from its true starting point.
- Every element is visited at most twice (once in the initial set scan, and once as part of a forward streak traversal), yielding amortized $O(n)$ time.

### Logic & Pseudocode ($O(n)$ Hash Set)

```text
function longestConsecutive(nums):
    if nums is empty:
        return 0

    num_set = Set(nums)
    longest_streak = 0

    for x in num_set:
        // Only start counting if x is the beginning of a sequence
        if (x - 1) not in num_set:
            current_num = x
            current_streak = 1

            while (current_num + 1) in num_set:
                current_num += 1
                current_streak += 1

            longest_streak = max(longest_streak, current_streak)

    return longest_streak
```

### Clean C++ Solution ($O(n)$ Time)

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.empty()) return 0;
        
        unordered_set<int> st(nums.begin(), nums.end());
        int maxStreak = 0;

        for (int num : st) {
            // Check if num is the start of a streak
            if (st.find(num - 1) == st.end()) {
                int currNum = num;
                int streak = 1;

                while (st.find(currNum + 1) != st.end()) {
                    currNum++;
                    streak++;
                }

                maxStreak = max(maxStreak, streak);
            }
        }

        return maxStreak;
    }
};
```

---

## 2. Check if an Array Is Consecutive

Given an array `nums` of length $n$, determine if it contains every number in the range $[x, x + n - 1]$ inclusive, where $x = \min(nums)$.

### Core Intuitions

An array of size $n$ is consecutive if and only if:
1. **No duplicates exist.**
2. $\max(nums) - \min(nums) + 1 == n$.

#### Approach A: Sorting
- Sort and verify $nums[i] == nums[i-1] + 1$ for all $1 \le i < n$.
- **Complexity:** $O(n \log n)$ time, $O(1)$ space.

#### Approach B: Min-Max + Hash Set ($O(n)$ Time, $O(n)$ Space)
- Find $\min$ and $\max$ in one pass.
- Verify that $\max - \min + 1 == n$.
- Insert all numbers into a hash set to confirm there are exactly $n$ unique elements.

#### Approach C: In-Place Marking / Cycle-Sort Check ($O(n)$ Time, $O(1)$ Space)
- If modifying the input array is permitted:
  1. Find $\min$ and $\max$. If $\max - \min + 1 \ne n$, return `false`.
  2. For each number $nums[i]$, normalize its target index as $idx = |nums[i]| - \min$.
  3. If $nums[idx] < 0$, we have seen this number before (duplicate found), so return `false`. Otherwise, negate $nums[idx]$ as a visited marker.

### Logic & Pseudocode ($O(n)$ Space / Hash Set)

```text
function isConsecutive(nums):
    n = length(nums)
    if n <= 1:
        return true

    min_val = min(nums)
    max_val = max(nums)

    // Range size condition
    if max_val - min_val + 1 != n:
        return false

    seen = Set()
    for x in nums:
        if x in seen:
            return false // Duplicate detected
        seen.add(x)

    return true
```

### Clean C++ Solution ($O(n)$ Time, $O(1)$ Extra Space via Sign-Marking)

```cpp
class Solution {
public:
    bool isConsecutive(vector<int>& nums) {
        int n = nums.size();
        if (n <= 1) return true;

        int minVal = *min_element(nums.begin(), nums.end());
        int maxVal = *max_element(nums.begin(), nums.end());

        if (maxVal - minVal + 1 != n) return false;

        // In-place visited flag using negative signs
        for (int i = 0; i < n; i++) {
            int targetIdx = abs(nums[i]) - minVal;

            // Duplicate element found
            if (nums[targetIdx] < 0) return false;

            nums[targetIdx] = -nums[targetIdx];
        }

        return true;
    }
};
```

---

## 3. Hand of Straights

Alice wants to rearrange cards into groups of size `groupSize`, where each group contains consecutive integers.

### Core Intuitions

1. **Divisibility Guard:** If $n \pmod{\text{groupSize}} \ne 0$, partitioning into equal complete groups is impossible; return `false` immediately.
2. **Greedy Anchor:** The smallest available card $c$ in the entire collection **must** be the start of a consecutive group $[c, c + 1, \dots, c + \text{groupSize} - 1]$. It cannot be placed anywhere else in that group because there is no smaller card available to precede it.
3. **Batch Decrement:** If card $c$ appears $f$ times, we must consume $f$ copies of every card from $c$ through $c + \text{groupSize} - 1$.

### Logic & Pseudocode (Ordered Map Frequency Table)

```text
function isNStraightHand(hand, groupSize):
    n = length(hand)
    if n % groupSize != 0:
        return false

    freq = SortedMap()
    for card in hand:
        freq[card] += 1

    for each (card, count) in freq:
        if count > 0:
            // Must complete 'count' groups starting from this card
            for offset from 1 to groupSize - 1:
                next_card = card + offset
                if freq[next_card] < count:
                    return false
                freq[next_card] -= count

    return true
```

### Clean C++ Solution

```cpp
class Solution {
public:
    bool isNStraightHand(vector<int>& hand, int groupSize) {
        int n = hand.size();
        if (n % groupSize != 0) return false;
        if (groupSize == 1) return true;

        map<int, int> counts;
        for (int card : hand) {
            counts[card]++;
        }

        for (auto& [card, count] : counts) {
            if (count > 0) {
                int needed = count;
                for (int i = 1; i < groupSize; i++) {
                    int nextCard = card + i;
                    if (counts[nextCard] < needed) {
                        return false;
                    }
                    counts[nextCard] -= needed;
                }
            }
        }

        return true;
    }
};
```

---

## Comparison & Takeaways

| Problem | Primary Pattern | Time Complexity | Space Complexity | Critical Gotcha |
| :--- | :--- | :--- | :--- | :--- |
| **Longest Consecutive Sequence** | Hash Set / Anchor at $x - 1 \notin S$ | $O(n)$ | $O(n)$ | Don't scan both directions from each element; only start counting when $x - 1$ is absent. |
| **Check Array Consecutive** | Range Check + Duplicate Detection | $O(n)$ | $O(1)$ (with sign negation) or $O(n)$ | Condition $\max - \min + 1 == n$ alone is insufficient; duplicates (e.g., $[1, 2, 2, 4]$) can break it. |
| **Hand of Straights** | Greedy Smallest Element + Frequency Map | $O(n \log m)$ | $O(m)$ unique cards | You must consume all consecutive offsets $c + i$, not just $c + 1$. |
