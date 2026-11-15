# 169. Majority Element

## 1. Problem Understanding

Given a nonempty integer array of length `n`, return the value occurring **strictly more than `n/2` times**. This problem guarantees such a value exists. For `[2,2,1,1,1,2,2]`, return `2`. “More than half” is strict: exactly half does not qualify.

## 2. Real-World Interview Scenario

An incident log contains a source ID for every report. The team guarantees that one source produced more than half of all reports. Find that source ID using as little memory as possible.

## 3. Pattern Recognition

Clues: repeated IDs, a frequency threshold greater than half, and a guaranteed winner. A frequency map is the direct translation. The strict majority guarantee also enables cancellation: pair reports from different sources, and the winner still survives.

## 4. My Thought Process

You count each value in `mp`, then inspect the frequencies for one exceeding `n/2`. That is correct. The final `return -1` is unreachable under the stated guarantee; if the guarantee is removed, `-1` is ambiguous when `-1` itself is a valid input value. Use an optional result or a separate “found” flag for that changed contract.

## 5. Core Intuition

A true majority outnumbers *all other values combined*. Cancel one occurrence of the candidate against one different value repeatedly; the majority cannot be completely canceled.

## 6. Step-by-Step Approach

1. Count occurrences of every value in the map.
2. Scan the map for a count greater than `n/2`.
3. Return that value. The problem guarantee makes it unique.

## 7. Original Code (unchanged)

```cpp

class Solution {
public:
    int majorityElement(vector<int>& nums)
    {
        int n = nums.size();
        unordered_map<int, int> mp;

        for (int i = 0; i < n; i++)
        {
            mp[nums[i]]++;
        }

        for (auto it : mp)
        {
            if (it.second > n / 2)
            {
                return it.first;
            }
        }

        return -1;
    }
};


```

## 8. Dry Run

For `[2,2,1,1,1,2,2]`, `n = 7`. The map evolves to `{2:4, 1:3}`. The threshold is `7/2 = 3`; `4 > 3`, so return `2`. Map iteration order does not matter because at most one value can exceed half.

## 9. Progressive Optimization

| Approach | Idea | Time | Extra space | Improvement |
|---|---|---|---|---|
| Brute force | Count each value by scanning the entire array. | O(n²) | O(1) | Baseline. |
| Your hash map | Count all values once, then inspect unique keys. | Expected O(n) | O(k), k distinct values | Avoids repeated full scans. |
| Sorting | Sort and take the middle element. | O(n log n) | Depends on sort | Uses no frequency map, but loses linear time and may mutate input. |
| Optimal: Boyer–Moore | Cancel different values using one candidate and count. | O(n) | O(1) | Removes the map under the majority guarantee. |

Separate optimal code:

```cpp
int majorityElementVoting(const vector<int>& nums) {
    int candidate = 0, votes = 0;
    for (int x : nums) {
        if (votes == 0) candidate = x;
        votes += (x == candidate ? 1 : -1);
    }
    return candidate; // Valid because a majority is guaranteed.
}
```

Sorting's extra space depends on the implementation: an in-place heapsort uses O(1); `std::sort` typically uses O(log n) stack space. The original map uses expected O(n) time for hash operations; adversarial hash collisions can make it worse.

## 10. Time and Space Complexity

- **Original:** Expected time O(n): n hash updates plus at most k map entries inspected. Extra space O(k), at most O(n), for distinct keys.
- **Voting:** Time O(n), one cancellation step per value. Extra space O(1), one candidate and counter.
- **Sorting:** Time O(n log n), comparison sorting. Extra space O(1) with heapsort or typically O(log n) stack with `std::sort`.
- **Brute force:** Time O(n²), n possible candidates each counted across n values. Extra space O(1), counters only.

## 11. Most Probable Interview Follow-Up Questions

1. **Can you use constant memory?** Boyer–Moore voting gives O(n) time and O(1) space.
2. **What if no majority is guaranteed?** Vote once, then count the final candidate in a second pass and return an optional value only if count `> n/2`. O(n) time, O(1) space.
3. **Can reports stream in?** Maintain the voting candidate and count per arrival; if the stream ends with no guarantee, a second pass or retained data is needed to verify. O(n) processing time, O(1) state.
4. **What if the threshold is `> n/3`?** Keep two candidates and verify both; O(n) time, O(1) space. See problem 229.

## 12. Interview Variations

1. **Dominant error code:** One code is promised to occupy over half the log. Apply cancellation. O(n) time, O(1) space.
2. **Untrusted poll winner:** No winner is guaranteed. Vote for a candidate, then verify it in a second pass. O(n) time, O(1) space.
3. **Dominant source in a data stream:** Update a candidate as events arrive; the guarantee makes the final candidate sufficient. O(n) total time, O(1) space.

## 13. Edge Cases

- One element is automatically the majority.
- The majority value may be negative, including `-1`; the original sentinel is safe only because a majority is guaranteed.
- Exactly `n/2` occurrences are insufficient if the guarantee is removed.
- Empty input is outside the contract; an empty map leads the original code to return `-1`.

## 14. Key Takeaways

“More than half” gives a unique winner and a cancellation invariant. Your map solution is correct and expected O(n) time, but uses O(n) space in the worst case. Voting retains linear time with constant space; verify it when existence is not guaranteed.
