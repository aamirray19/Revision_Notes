# 53. Maximum Subarray

## 1. Problem Understanding

Given a nonempty integer array, return the largest sum of any **nonempty contiguous** subarray. Negative values are allowed. For example, `[-2,1,-3,4,-1,2,1,-5,4]` returns `6` from `[4,-1,2,1]`. The usual LeetCode bounds are up to `10^5` elements, with values small enough that the answer fits a signed 32-bit integer.

## 2. Real-World Interview Scenario

A service records its net gain or loss each day. Find the consecutive run of days with the greatest total gain. You must choose at least one day.

## 3. Pattern Recognition

Clues: the answer is a **contiguous interval**, the score is an **additive sum**, and a negative prefix can only hurt any interval extended to the right. Translate the story into “best sum of a subarray ending here” and “best sum seen anywhere.” This is Kadane's algorithm (dynamic programming with constant state).

## 4. My Thought Process

Your `sum` accumulates the current candidate interval, while `maxi` stores the best completed candidate. At each value, update `maxi` **before** discarding a negative `sum`; this order preserves the correct answer for an all-negative array. Resetting a negative sum to zero means the next value starts a new interval. `INT_MIN` is necessary because zero is not a valid answer for every input.

## 5. Core Intuition

If the sum collected so far is negative, carrying it into tomorrow makes every future interval worse. Drop it and restart. If it is nonnegative, keeping it cannot hurt a future sum.

## 6. Step-by-Step Approach

1. Initialize `sum = 0` and `maxi = INT_MIN`.
2. Add the current element to `sum`.
3. Compare `sum` with `maxi` so this nonempty interval is considered.
4. If `sum < 0`, reset it to zero before the next element.
5. Return `maxi`.

## 7. Original Code (unchanged)

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) 
    {
        int sum = 0;
        int maxi = INT_MIN;

        for (int i = 0; i < nums.size(); i++) 
        {
            sum += nums[i];
            maxi = max(maxi, sum);

            if (sum < 0) sum = 0;
        }
        return maxi;
    }
};
```

## 8. Dry Run

For `[-2,1,-3,4,-1,2,1]`:

| Value | `sum` after addition | `maxi` | `sum` carried forward |
|---:|---:|---:|---:|
| -2 | -2 | -2 | 0 |
| 1 | 1 | 1 | 1 |
| -3 | -2 | 1 | 0 |
| 4 | 4 | 4 | 4 |
| -1 | 3 | 4 | 3 |
| 2 | 5 | 5 | 5 |
| 1 | 6 | 6 | 6 |

## 9. Progressive Optimization

| Approach | Idea | Time | Extra space | Improvement |
|---|---|---|---|---|
| Brute force | Enumerate every start and end, recomputing each sum. | O(n³) | O(1) | Baseline. |
| Better | Fix each start and extend a running sum across all ends. | O(n²) | O(1) | Avoids summing the same interval repeatedly. |
| Optimal: your code | Keep only the best useful prefix ending at the current position. | O(n) | O(1) | Every element is processed once; no auxiliary array is needed. |

The better version is useful for explaining the transition from interval enumeration to Kadane's algorithm:

```cpp
int maxSubArrayQuadratic(const vector<int>& nums) {
    int best = INT_MIN;
    for (int start = 0; start < (int)nums.size(); ++start) {
        int sum = 0;
        for (int end = start; end < (int)nums.size(); ++end) {
            sum += nums[end];
            best = max(best, sum);
        }
    }
    return best;
}
```

There is no general algorithm faster than O(n): even deciding whether the final value changes the answer requires reading it. Your implementation is therefore time and space optimal for the sum-only result.

## 10. Time and Space Complexity

- **Original / optimal:** Time O(n), because the loop visits each element once. Extra space O(1), because `sum` and `maxi` are the only state.
- **Better:** Time O(n²), one extension for each start/end pair. Extra space O(1), only counters and a running sum.
- **Brute force:** Time O(n³), because each of O(n²) intervals is summed in O(n). Extra space O(1), with direct summation.

## 11. Most Probable Interview Follow-Up Questions

1. **Return the interval too?** Track the tentative start whenever the running sum resets, and record its start/end when `maxi` improves. O(n) time, O(1) extra space.
2. **What if every value is negative?** The update before reset returns the largest (least negative) element. O(n) time, O(1) space.
3. **What if the array is circular?** Compare ordinary Kadane with `totalSum - minimumSubarraySum`; exclude the latter when all values are negative. O(n) time, O(1) space.
4. **What if sums can exceed `int`?** Use `long long` for `sum`, `maxi`, and the return type. Complexity stays O(n) time, O(1) space.

## 12. Interview Variations

1. **Best consecutive revenue period:** Daily profit/loss values map directly to array elements. Find the maximum contiguous sum with Kadane. O(n) time, O(1) space.
2. **Best circular shift of a daily log:** The interval may wrap from the end to the beginning. Compare non-wrapping maximum with total minus minimum interval. O(n) time, O(1) space.
3. **Report both score and dates:** The additive score is unchanged, but return boundaries when a new best appears. O(n) time, O(1) space.

## 13. Edge Cases

- One element: that element is the answer.
- All negative: the largest element wins; resetting before updating `maxi` would be wrong.
- Zeros: a zero may tie the best sum; the code still returns the correct value.
- Empty input: the original problem excludes it; this code would return `INT_MIN`, so define a separate contract if emptiness is allowed.

## 14. Key Takeaways

Recognize a contiguous additive score. A negative running prefix is never worth keeping. Record the best **before** resetting. Do not default the answer to zero when an interval must be nonempty. Your original solution already reaches O(n) time and O(1) space.
