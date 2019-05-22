# Cousins in Binary Tree
## 问题描述
In a binary tree, the root node is at depth 0, and children of each depth k node are at depth k+1.  
Two nodes of a binary tree are cousins if they have the same depth, but have different parents.  
We are given the root of a binary tree with unique values, and the values x and y of two different nodes in the tree.  
Return true if and only if the nodes corresponding to the values x and y are cousins.  

在二叉树中，根节点深度为0，每个深度k节点的子节点深度为k+1。  
如果二叉树的两个节点具有相同的深度，但具有不同的父节点，则它们是表兄妹。  
给出了具有唯一值的二叉树的根，以及树中两个不同节点的x和y值。  
当且仅当与值x和y对应的节点是表兄妹时，返回true。  

## 解法一
###
+ 这里主要借助按层遍历树的思想，如果x，y是表兄妹节点说明两个值会出现在同一层上，不过要排除同一个父节点的情况。

```java
import java.util.*;
class Solution {
    public boolean isCousins(TreeNode root, int x, int y) {
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        TreeNode node = null;
        while(!queue.isEmpty()){
            //用来保存每层的所有值
            List<Integer> list = new ArrayList<>();
            //遍历当层节点
            int len = queue.size();
            for(int i = 0;i < len ;i++){
                node = queue.poll();
                //如果x，y同时出现在一个节点的左右孩子节点，flag值就为2，返回false
                int flag = 0;
                if(node.left!=null){
                    queue.offer(node.left);
                    list.add(node.left.val);
                    if(node.left.val == x || node.left.val == y){
                        flag++;
                    }
                }
                if(node.right!=null){
                    queue.offer(node.right);
                    list.add(node.right.val);
                    if(node.right.val == x || node.right.val == y){
                        flag++;
                    }
                }
                if(flag == 2){
                    return false;
                }
                
            }
            //查看x，y是否出现在同一层
            if(list.contains(x) && list.contains(y)) return true;
        }
        return false;
    }
}
```
