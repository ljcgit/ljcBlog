# 重建二叉树
## 问题描述
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。
假设输入的前序遍历和中序遍历的结果中都不含重复的数字。
例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

## 解法1
### 思路
  + 将前序集合转换为Map集合key为前序集合的值，value为前序集合的下标；将中序集合转换为链表；
  + 根据前序集合获取到二叉树的根节点，把中序集合根节点左边的取出作为左孩子集合，右边的为右孩子；
  + 递推。
### 代码
```java
    public TreeNode reConstructBinaryTree(int [] pre, int [] in) {

        Map<Integer,Integer> maps = new HashMap<>();
        for(int i = 0; i < pre.length ; i++){
            maps.put(pre[i],i);
        }
        List<Integer> lists = new ArrayList<>();
        for(int t : in){
            lists.add(t);
        }
        return dfs(maps,lists);
    }

    private TreeNode dfs(Map<Integer,Integer> pre, List<Integer> in){
        if(in.size() == 1){
            return new TreeNode(in.get(0));
        }
        if(in.size() < 1){
            return null;
        }
        //找到根节点
        int p = in.get(0);
        for(int i = 1 ;i<in.size();i++){
            if(pre.get(in.get(i)) < pre.get(p)){
                p = in.get(i);
            }
        }
        //创建根节点
        TreeNode node = new TreeNode(p);
        //左子树
        List<Integer> left = new ArrayList<>();
        int j = 0;
        for(;j<in.size();j++){
            if(in.get(j) == p)
                break;
            left.add(in.get(j));
        }
        node.left = dfs(pre,left);
        // 右子树
        List<Integer> right = new ArrayList<>();
        j++;   //跳过根节点
        for(;j<in.size();j++){
            right.add(in.get(j));
        }
        node.right = dfs(pre,right);
        return node;
    }
```


## 解法2
### 代码
```java
    private int[] pre ;
    private int[] in;
    public TreeNode reConstructBinaryTree(int [] pre, int [] in) {
        this.pre = pre;
        this.in = in;
        return dfs(0,pre.length-1,0,in.length-1);
    }

    private TreeNode dfs( int preStart,int preEnd,int inStart,int inEnd){
        if(inStart == inEnd){
            return new TreeNode(in[inStart]);
        }
        if(inStart>inEnd){
            return null;
        }
        //根节点
        TreeNode node = new TreeNode(pre[preStart]);
        for(int i = inStart;i<=inEnd;i++){
            if(in[i] == pre[preStart]){
            //i-inStart为个数
                node.left = dfs(preStart+1,preStart+i-inStart,inStart,i-1);
                node.right = dfs(preStart+i-inStart+1,preEnd,i+1,inEnd);
                break;
            }
        }
        return node;
    }
```
