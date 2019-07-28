## Jump Game

### Description
Given an array of non-negative integers, you are initially positioned at the first index of the array.
Each element in the array represents your maximum jump length at that position.
Determine if you are able to reach the last index.

给定一个非负整数数组，初始位置是数组的第一个索引。
数组中的每个元素表示该位置的最大跳转长度。
你需要确定是否能够达到最后一个索引。

### Example

**Example 1:**

Input: [2,3,1,1,4]
Output: true
Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.

**Example 2:**

Input: [3,2,1,0,4]
Output: false
Explanation: You will always arrive at index 3 no matter what. Its maximum
             jump length is 0, which makes it impossible to reach the last index.
             
### 思路
#### 1
> 从终点出发判断是否可以到达起点
```java
class Solution {
    public boolean canJump(int[] nums) {
     	int n=nums.length;
        int last=n-1,i,j;
        for(i=n-2;i>=0;i--){
            if(i+nums[i]>=last)last=i;
        }
        return last<=0;      
    }
}
```


#### 2
> 从前往后，依次判断能够到达的最远距离。
```java
class Solution {
    public boolean canJump(int[] nums) {
     	int dist = 0;
        for(int i = 0; i < nums.length-1 && i <= dist ;i++){
            dist = Math.max(dist,i+nums[i]);
        }
        return dist >= nums.length-1;
    }
}
```
