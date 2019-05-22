# 数据流中的中位数
如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。
如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。
我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

## 方法一
```java
import java.util.*;
public class Solution {
 
    List<Integer> lists = new ArrayList<>();
    public void Insert(Integer num) {
        lists.add(num);
    }
 
    public Double GetMedian() {
        Collections.sort(lists);
        if((lists.size()&1)==1){
            return lists.get(lists.size()/2)*1.0;
        }else{
            return (lists.get(lists.size()/2)+lists.get(lists.size()/2-1))/2.0;
        }
    }
 
 
}
```

## 方法二
### 思路
 + 构建一个小根堆，一个大根堆，两个堆元素个数相差不能超过1；
 + 每次将小根堆中最小的放入大根堆中或者大根堆中最大的元素放入小根堆
 + 最后可以达到两个堆的堆顶就是中位数。
 
import java.util.Comparator;
import java.util.PriorityQueue;

public class Solution {
    int count;
    PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(11, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            //PriorityQueue默认是小顶堆，实现大顶堆，需要反转默认排序器
            return o2.compareTo(o1);
        }
    });

    public void Insert(Integer num) {
        count++;
        if ((count & 1) == 0) { // 判断偶数的高效写法
            if (!maxHeap.isEmpty() && num < maxHeap.peek()) {
                maxHeap.offer(num);
                num = maxHeap.poll();
            }
            minHeap.offer(num);
        } else {
            if (!minHeap.isEmpty() && num > minHeap.peek()) {
                minHeap.offer(num);
                num = minHeap.poll();
            }
            maxHeap.offer(num);
        }
    }

    public Double GetMedian() {
        if(count==0)
            throw new RuntimeException("no available number!");
        double result;
        //总数为奇数时，大顶堆堆顶就是中位数
        if((count&1)==1)
            result=maxHeap.peek();
        else
            result=(minHeap.peek()+maxHeap.peek())/2.0;
        return result;
    }

    @Test
    public void test1(){
        Insert(1);
        Insert(2);        Insert(3);
        Insert(4);
        System.out.println(GetMedian());
    }
}
