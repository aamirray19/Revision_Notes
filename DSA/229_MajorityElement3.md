# 229. Majority Element II (`> n/3`)

## 1. Problem Understanding

Given an integer array of length `n`, return **all** values appearing strictly more than `n/3` times. There may be zero, one, or two answers; `[3,2,3]` returns `[3]`. The input may be empty. Answer order is unspecified unless an interviewer adds an ordering requirement.

## 2. Real-World Interview Scenario

A support queue records the customer ID on each ticket. Find every customer responsible for more than one third of tickets. There may be no such customer, and you want to avoid storing every distinct ID when the queue is large.

## 3. Pattern Recognition

Clues: a **frequency threshold** and a request for **all qualifying values**. Since three distinct values cannot each occupy more than one third of the array, at most two answers exist. This suggests two candidate slots and cancellation of triples of different values. Unlike problem 169, existence is not guaranteed, so candidates must be verified.

## 4. My Thought Process

Your approach correctly extends the frequency map from problem 169: count every value, then keep counts greater than `n/3`. The main conceptual difference is that there can be **two** winners or **none**. The returned vector's order follows `unordered_map` iteration and is therefore not stable; this is acceptable when order does not matter. If sorted output is requested, sort the answer separately.

## 5. Core Intuition

Remove one occurrence each of three different values. A value that truly appears more than one third of the time cannot be eliminated entirely. After cancellation, only two possible winners remain, but cancellation can leave false candidates, so count them again.

## 6. Step-by-Step Approach

1. Count every value with `mp`.
2. Scan each distinct entry.
3. Add a value to `ans` only when its count is strictly greater than `n/3`.
4. Return `ans`.

## 7. Original Code (unchanged)

```cpp
class Solution {
public:
    vector<int> majorityElement(vector<int>& nums) 
    {
        unordered_map<int, int> mp;
        vector<int> ans;
        int n = nums.size();

        for (int i = 0; i < n; i++)
        {
            mp[nums[i]]++;
        }

        for (const auto& num : mp)
        {
            if (num.second > n/3)
            {
                ans.push_back(num.first);
            }
        }

        return ans;
    }
};

```

## 8. Dry Run

For `[1,2,1,3,2,1,2,4]`, `n = 8`, so the integer threshold is `8/3 = 2`. Frequencies become `{1:3, 2:3, 3:1, 4:1}`. Both `1` and `2` pass `count > 2`; `3` and `4` do not. The returned order may be `[1,2]` or `[2,1]`.

## 9. Progressive Optimization

| Approach | Idea | Time | Extra space | Improvement |
|---|---|---|---|---|
| Brute force | For each distinct candidate, scan the array to count it. | O(n²) | O(1) | Baseline if duplicate candidates are skipped without storage by scanning earlier positions. |
| Your hash map | Count every value once. | Expected O(n) | O(k), k distinct values | Removes repeated counting. |
| Optimal: extended Boyer–Moore | Cancel triples using two candidates; verify in a second pass. | O(n) | O(1) | Removes the map while retaining linear time. |

Separate optimal code:

```cpp
vector<int> majorityElementVoting(const vector<int>& nums) {
    int a = 0, b = 1, countA = 0, countB = 0;
    for (int x : nums) {
        if (x == a) ++countA;
        else if (x == b) ++countB;
        else if (countA == 0) { a = x; countA = 1; }
        else if (countB == 0) { b = x; countB = 1; }
        else { --countA; --countB; }
    }
    countA = countB = 0;
    for (int x : nums) {
        if (x == a) ++countA;
        else if (x == b) ++countB;
    }
    vector<int> result;
    if (countA > (int)nums.size() / 3) result.push_back(a);
    if (countB > (int)nums.size() / 3) result.push_back(b);
    return result;
}
```

The initial candidate values are placeholders until their counts become positive. The `else if` order keeps candidate slots distinct. Verification is essential because surviving cancellation does not prove the threshold.

## 10. Time and Space Complexity

- **Original:** Expected time O(n): n hash updates plus at most k map entries scanned. Extra space O(k), at most O(n), for distinct values. Pathological collisions may worsen hash time.
- **Voting:** Time O(n), one candidate pass and one verification pass. Extra space O(1), two candidates, counters, and at most two output values.
- **Brute force:** Time O(n²), up to n candidates scanned over up to n values, including checking earlier positions to avoid duplicate work. Extra space O(1), counters and at most two results.

## 11. Most Probable Interview Follow-Up Questions

1. **Why only two candidate slots?** Three values each occurring `> n/3` would require more than n positions. O(1) slots suffice.
2. **Why verify after voting?** With no guaranteed winner, cancellation can retain values occurring too rarely. A second pass keeps O(n) time, O(1) space.
3. **What if the threshold is `> n/k`?** Keep at most `k-1` candidates and verify them. Straightforward implementation costs O(nk) time, O(k) space; for fixed k, that is O(n) time.
4. **What if output must be sorted?** Sort the at-most-two answers; this is O(1) extra time and space beyond either main algorithm.
5. **Can you process a non-replayable stream?** You can maintain candidate slots in O(1) memory, but exact final verification needs a replay or additional stored counts/data; candidate counters alone are insufficient.

## 12. Interview Variations

1. **Large customers in a ticket queue:** More than one third of ticket IDs qualifies. Use two slots, then recount. O(n) time, O(1) space.
2. **Popular device types above one quarter of events:** There can be at most three answers. Use three slots and verification. O(n) time for fixed threshold, O(1) space.
3. **Sorted list of frequent IDs:** Compute the qualifying values as above, then order the at-most-two answers. The overall cost remains O(n) time, O(1) extra space with voting.

## 13. Edge Cases

- Empty input returns an empty vector in both approaches.
- `n = 1` or `n = 2`: every distinct element appears more than `n/3`.
- Exactly `n/3` occurrences do not qualify when n is divisible by three.
- Negative values and duplicates work; no special numeric range is needed for map keys.
- Map iteration gives arbitrary answer order.

## 14. Key Takeaways

For a threshold `> n/k`, at most `k-1` values can qualify. Your map is correct and simple but may use O(n) space. Two-candidate voting reaches O(n) time and O(1) space; always verify candidates when winners are not guaranteed.
