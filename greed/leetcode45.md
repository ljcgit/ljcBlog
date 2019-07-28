## Jump Game II

### Description

Given an array of non-negative integers, you are initially positioned at the first index of the array.
Each element in the array represents your maximum jump length at that position.
Your goal is to reach the last index in the minimum number of jumps.

给定一个非负整数数组，初始位置是数组的第一个索引。
数组中的每个元素表示该位置的最大跳转长度。
您的目标是在最少的跳跃次数中达到最后一个索引。


### Example
Input: [2,3,1,1,4]   
Output: 2   
Explanation: The minimum number of jumps to reach the last index is 2.   
Jump 1 step from index 0 to 1, then 3 steps to the last index.  

### 代码
> 在每个位置需要的步数成单调递增（后面的位置可以通过前面的一步到达）。
  只需将整个数组划分为若干个区间，区间内的步数相等，同时下一个区间比上一个区间步数多1。

```java
class Solution {
    public int jump(int[] nums) {
        int l = 0, r = 0, step = 0;
        while(l<=r&&r<nums.length-1){
            int mr = r;
            for(int i = l; i <= r; i++){
                mr = Math.max(mr,i+nums[i]);
            }
            r = mr;
            l = l+1;
            step++;
        }
        return step;
    }
}
```
