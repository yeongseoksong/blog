---
{"dg-publish":true,"permalink":"///lowerbound-upperbound/"}
---

```java
 class Solution {
    public int[] searchRange(int[] nums, int target) {
        int start = findFirst(nums, target);
        int end = findLast(nums, target);
        return new int[] { start, end };
    }

    private int findFirst(int[] nums, int target) {
        int index = -1;
        int left = 0, right = nums.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            } else {
                index = mid;
                right = mid - 1; // 왼쪽으로 계속 탐색
            }
        }

        return index;
    }

    private int findLast(int[] nums, int target) {
        int index = -1;
        int left = 0, right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            } else {
                index = mid;
                left = mid + 1; // 오른쪽으로 계속 탐색
            }
        }

        return index;
    }
}
```