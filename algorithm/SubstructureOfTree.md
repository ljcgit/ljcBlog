# 树的子结构 （剑指Offer）

## 问题描述
输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

## 解题思想

  1.如果A和B为空，则一定不满足条件，返回false；
  
  2.递归判断当前A节点的值是否相同；
  
  3.相同，则调用eq()方法两个树结构是否相同；
  
  4.eq()方法先判断B是否为空，为空则返回true，表示该路径下所有值都和A相同（已经排除B本身就为空的情况）；
  
  5.如果A为空，表示两棵树值不同直接返回false；
  
  6.不然继续判断左子树和右子树。

## 代码
```java
import java.util.*;
public class Solution {
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root2 == null|| root1==null) return false;
        if(root1.val == root2.val && eq(root1,root2)){
            return true;
        }
        return HasSubtree(root1.left,root2)||HasSubtree(root1.right,root2);
    }
    
    public boolean eq(TreeNode root1,TreeNode root2){
        if(root2 == null) return true;
        if(root1 == null) return false;
        if(root1.val != root2.val){
            return false;
        }
        return eq(root1.left,root2.left)&&eq(root1.right,root2.right);
        
    }
    
    
}
```
