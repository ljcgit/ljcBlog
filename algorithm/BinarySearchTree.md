# 二叉搜索树与双向链表
## 问题
输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。
## 思路
+ 二叉搜索树的中序遍历集合就是一个有序的集合。
## 代码
```java
import java.util.*;
public class Solution {
    public TreeNode Convert(TreeNode pRootOfTree) {
        if(pRootOfTree == null || (pRootOfTree.left==null&&pRootOfTree.right==null)){
            return pRootOfTree;
        }
        List<TreeNode> lists = new ArrayList<>();
        dfs(lists,pRootOfTree);
        lists.get(0).left=null;
        lists.get(0).right=lists.get(1);
        for(int i = 1;i<lists.size()-1;i++){
            lists.get(i).left=lists.get(i-1);
            lists.get(i).right=lists.get(i+1);
        }
        lists.get(lists.size()-1).right=null;
        lists.get(lists.size()-1).left=lists.get(lists.size()-2);
        return lists.get(0);
    }
    
    private void dfs(List<TreeNode> node,TreeNode t){
        if(t==null){
            return;
        }
        dfs(node,t.left);
        node.add(t);
        dfs(node,t.right);
    }
}
```
