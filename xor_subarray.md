# Count Subarrays with Given XOR K

### Core Invariant

If a contiguous subarray from index $j$ to $i$ has an XOR product equal to $k$:


$$\text{prefix\_xor}[i] \oplus \text{prefix\_xor}[j - 1] = k$$

Using the self-inverse property of XOR ($A \oplus B = C \iff A \oplus C = B$):


$$\text{prefix\_xor}[j - 1] = \text{prefix\_xor}[i] \oplus k$$

Count previous occurrences of $(\text{prefix\_xor}[i] \oplus k)$ using a frequency table.

---

### Pseudocode

```text
function subarraysWithXorK(nums, k):
    freq = hash_map()
    freq[0] = 1          // base case: empty prefix before array has XOR 0
    running_xor = 0
    count = 0

    for x in nums:
        running_xor = running_xor ^ x
        target = running_xor ^ k

        if target in freq:
            count += freq[target]

        freq[running_xor] += 1

    return count

```

---

### Complexity

* **Time Complexity:** $O(n)$ average using a hash map ($O(n \log n)$ with an ordered tree map).
* **Space Complexity:** $O(n)$ auxiliary space to store prefix XOR frequencies.
