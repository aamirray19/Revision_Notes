# 152. Maximum Product Subarray

## 1. Problem Understanding

Given a nonempty integer array, return the largest product of any **nonempty contiguous** subarray. Values may be negative or zero. `[2,3,-2,4]` returns `6` from `[2,3]`. Ask about numeric overflow if the input limits are changed.

## 2. Real-World Interview Scenario

A sequence of processing stages multiplies a signed score by each stage's factor. A zero wipes out the score; a negative factor flips its sign. Find the consecutive stage range with the highest final score.

## 3. Pattern Recognition

Clues: **contiguous range**, **multiplication**, **zero boundaries**, and **sign flips**. Translate the scenario into products of subarrays. A large negative product can become a large positive one after another negative factor, so tracking only a maximum is insufficient.

## 4. My Thought Process

Your `maxi` begins at `INT_MIN` to support negative answers. `prefix` and `suffix` scan from opposite ends. Reset a running product to one *after* it hits zero and *before* multiplying the next element, because no nonzero interval crosses zero. Comparing both products each iteration handles an odd number of negatives by exposing the interval after the first negative or before the last negative. That comparison completes your unfinished third note.

## 5. Core Intuition

In a zero-free stretch, an even number of negatives makes the full product positive. With an odd number, discard either the prefix through the first negative or the suffix from the last negative. Scanning both directions finds those choices.

## 6. Step-by-Step Approach

1. Set `maxi = INT_MIN` and both running products to one.
2. For each index, reset any zero running product to one.
3. Multiply by the next left and right elements respectively.
4. Compare both products with `maxi`, including zero when reached.
5. Return `maxi`.

## 7. Original Code (unchanged)

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int n = nums.size();
        int maxi = INT_MIN;
        int prefix = 1, suffix = 1;
        for(int i = 0; i < n; i++) 
        {
            if(prefix == 0) prefix = 1;
            if(suffix == 0) suffix = 1;
            prefix *= nums[i];
            suffix *= nums[n-i-1];
            maxi = max(maxi, max(prefix, suffix));
        }
        return maxi;
    }
};


```

## 8. Dry Run

For `[-2,3,-4,0,-1]`, `suffix` reads from the right:

| `i` | Left | Right | `prefix` | `suffix` | `maxi` |
|---:|---:|---:|---:|---:|---:|
| 0 | -2 | -1 | -2 | -1 | -1 |
| 1 | 3 | 0 | -6 | 0 | 0 |
| 2 | -4 | -4 | 24 | -4 | 24 |
| 3 | 0 | 3 | 0 | -12 | 24 |
| 4 | -1 | -2 | -1 | 24 | 24 |

The best interval is `[-2,3,-4]`, product `24`.

## 9. Progressive Optimization

| Approach | Idea | Time | Extra space | Improvement |
|---|---|---|---|---|
| Brute force | Recompute the product of every interval. | O(n³) | O(1) | Baseline. |
| Better | Fix a start and extend a running product. | O(n²) | O(1) | Reuses earlier multiplication. |
| Optimal: your code | Scan products from both ends, resetting at zero. | O(n) | O(1) | Visits each element a constant number of times. |
| Optimal alternative | Keep maximum and minimum products ending here. | O(n) | O(1) | Supports left-to-right streaming. |

Separate code for the streaming alternative:

```cpp
int maxProductMinMax(const vector<int>& nums) {
    int high = nums[0], low = nums[0], best = nums[0];
    for (int i = 1; i < (int)nums.size(); ++i) {
        int x = nums[i];
        int nextHigh = max({x, x * high, x * low});
        int nextLow = min({x, x * high, x * low});
        high = nextHigh;
        low = nextLow;
        best = max(best, high);
    }
    return best;
}
```

Calculate both next states from the previous pair. Every value may affect the answer, so O(n) time is optimal.

## 10. Time and Space Complexity

- **Original:** Time O(n), one loop with constant work per index. Extra space O(1), three integer variables.
- **Min/max alternative:** Time O(n), one state update per element. Extra space O(1), constant state.
- **Better:** Time O(n²), one extension per start/end pair. Extra space O(1), one running product.
- **Brute force:** Time O(n³), up to n multiplications for each of O(n²) intervals. Extra space O(1), direct multiplication.

## 11. Most Probable Interview Follow-Up Questions

1. **Can factors arrive as a stream?** Use min/max-ending-here state. O(n) time over n arrivals, O(1) space.
2. **Why keep a minimum product?** Multiplying it by a negative may produce the next maximum. O(n) time, O(1) space.
3. **What happens at zero?** Zero is a candidate and breaks nonzero intervals. Both optimal methods restart. O(n) time, O(1) space.
4. **Return the interval?** Carry candidate start indices with min/max state and save best boundaries. O(n) time, O(1) space.
5. **What if products overflow `int`?** Choose a type covering the specified range; `long long` helps only when 64 bits suffice. Fixed-width complexity remains O(n) time, O(1) space.

## 12. Interview Variations

1. **Best consecutive conversion factors:** Signed multiplicative factors map to a product subarray. Use either optimal method. O(n) time, O(1) space.
2. **Live pipeline score:** Reverse scanning is unavailable. Maintain maximum and minimum products as factors arrive. O(n) total time, O(1) space.
3. **Best stage range:** Keep start/end indices along with min/max state. O(n) time, O(1) space.

## 13. Edge Cases

- A single negative element is the answer if it is the only element.
- An even number of negatives can make the full segment positive.
- Zero can be the maximum when all nonzero products are negative.
- Multiple zeros separate independent segments.
- Empty input is outside the contract; the original code returns `INT_MIN` for it.

## 14. Key Takeaways

For products, negative signs reverse which prior value is useful. Watch for zero boundaries and parity of negatives. Your two-direction scan is already O(n) time and O(1) space; min/max state is the streaming alternative.
