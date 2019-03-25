Manacher's Algorithm俗称马拉车算法，对于求字符串中最长回文子串效率极高。

在求最长回文子串的时候，常见的写法是根据每个字符往两边找回文子串，比较最大值。

> Manacher's Algorithm算法思想：
> + 首先将原字符串用'\$'和'\#'填充成奇数长度，这样可以避免对于奇数和偶数长度字符串的讨论。例如："abc"填充成"$#a#b#c#"。
> + 然后根据当前下标是否在回文字符串的最右边界内，如果在边界内，则p[i] = maxR > i ? Math.min(maxR-i,p[2*maxI-i]) : 1;
> + 如果p[i]>=maxR-i，则说明当前字符不在边界内或者边界外还可能存在回文字符串的部分，则继续对边界外的部分进行遍历。
> + 最长回文串的长度就等于最大半径-1，起始位置就是（最大回文串的中间位置-半径）/2。

代码实现：
```java
    public static  String longestPalindrome(String s) {
        char[] ch = new char[2*s.length()+2];
        ch[0]='$';
        ch[1]='#';
        for(int i=0;i<s.length();i++){
            ch[(i+1)*2+1] = '#';
            ch[(i+1)*2] = s.charAt(i);
        }
        int[] p=new int[ch.length];
        p[0]=p[1]=1;
        int maxR=1;  //右边界
        int maxIndex=1;   //最长回文子串的下标
        int maxI=1;  //右边界对应的中点
        int ii=-1,jj=-1;
        for(int i=2;i<ch.length;i++){
            p[i] = maxR > i ? Math.min(maxR-i,p[2*maxI-i]) : 1;
            if(p[i]>=maxR-i){
                ii=i-p[i];
                jj=i+p[i];
                while(ii>=0&&jj<ch.length&&ch[ii]==ch[jj]){
                    ii--;
                    jj++;
                }
                p[i]=jj-i;
                if(p[i]>p[maxIndex]){
                    maxIndex=i;
                }
                maxR=jj;
                maxI=i;
            }
        }
        return s.substring((maxIndex-p[maxIndex])/2,(maxIndex-p[maxIndex])/2+p[maxIndex]-1);
    }
```
