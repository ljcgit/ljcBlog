# 孩子们的游戏
## 问题描述
每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。
其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,
然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,
并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1)

## 解题思路
### A 利用数组存储状态
+ 利用一个n大小的boolean数组保存所有状态，初始时状态都为false；
+ 循环遍历数组，直到只剩下一个为false的元素；
+ 返回false元素的下标。

### B 利用链表存储元素
+ 利用链表存储所有的元素；
+ 每次删除下标为（index+m-1)%list.size()的元素，直到链表只剩下一个元素；
+ 返回最后剩下的元素。

## 示例代码

### A
```java
public class Solution {
    public int LastRemaining_Solution(int n, int m) {
        boolean[] flag = new boolean[n];
        int num = n;
        int t = 0;
        for(int i = 0;i<n;i=(i+1)%n){
            if(!flag[i]){
                t++;
                if(t==m){
                    flag[i]=true;
                    t=0;
                    num--;
                }
                if(num==1){
                    break;
                }
            }
             
        }
        for(int i = 0;i<n;i++){
            if(!flag[i]){
                return i;
            }
        }
        return -1;
    }
}
```

### B
```java
import java.util.*;
public class Solution {
    public int LastRemaining_Solution(int n, int m) {
        if(n == 0 || m == 0) return -1;
        List<Integer> lists = new LinkedList<>();
        for(int i =0 ;i<n;i++){
            lists.add(i);
        }
        int t = 0;
        while(lists.size()>1){
            t = (t+m-1)%lists.size();
            lists.remove(t);
        }
        return lists.get(0);
    }
}
```
