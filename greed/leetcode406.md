## Queue Reconstruction by Height

### Description
Suppose you have a random list of people standing in a queue. Each person is described by a pair of integers (h, k), 
where h is the height of the person and k is the number of people in front of this person who have a height greater than or equal to h. 
Write an algorithm to reconstruct the queue.

Note:
The number of people is less than 1,100.

假设你有一列随机的人在排队。每个人由一对整数(h, k)描述，
其中h是这个人的身高k是这个人前面身高大于等于h的人数。
编写一个算法来重建队列。

### Example
Input:  
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

Output:  
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]


### Demo
#### 1

> 首先我们可以对原来队列进行默认排序（按照高度降序，高度一样的按照K值升序）。然后遍历数组将数据插入到合适的位置中（即K指定位置处）。
  因为默认排序后身高高的都在最前面，当后面的人插入到他们中时，不会影响高个子的顺序，而相同高度的又根据K值进行排序，这样就能保证队列顺序合适。

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        if (people == null || people.length == 0 || people[0].length == 0)
            return new int[0][0];

        Arrays.sort(people, new Comparator<int[]>() {
            public int compare(int[] a, int[] b) {
                if (b[0] == a[0]) return a[1] - b[1];
                return b[0] - a[0];
            }
        });

        List<int[]> da = new ArrayList<>();

        int len = people.length;
        for(int i = 0; i < len; i++){
            int k = people[i][1];
            da.add(k,people[i]);
        }
        for(int i = 0;i < len;i++){
            people[i] = da.get(i);
        }
        return people;
    }
}
```
