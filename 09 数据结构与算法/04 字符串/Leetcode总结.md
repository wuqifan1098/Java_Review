# 面试题

## 1.

# [344. 反转字符串](https://leetcode-cn.com/problems/reverse-string/)

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `char[]` 的形式给出。

不要给另外的数组分配额外的空间，你必须**原地修改输入数组**、使用 O(1) 的额外空间解决这一问题。

你可以假设数组中的所有字符都是 [ASCII](https://baike.baidu.com/item/ASCII) 码表中的可打印字符。

**示例 1：**

```
输入：["h","e","l","l","o"]
输出：["o","l","l","e","h"]
```

**示例 2：**

```
输入：["H","a","n","n","a","h"]
输出：["h","a","n","n","a","H"]
```

**思路**：

首尾交换，双指针

直接从两头往中间走，同时交换两边的字符。

```java
class Solution{
    public void reverseString(char[] s){
        for(int i = 0; i < s.length/2 ; i++){
            char temp = s[i];
            s[i] = s[s.length - i];
            s[s.length - i] = temp;
        }
    }
}
```

# [796. 旋转字符串](https://leetcode-cn.com/problems/rotate-string/)

给定两个字符串, `A` 和 `B`。

`A` 的旋转操作就是将 `A` 最左边的字符移动到最右边。 例如, 若 `A = 'abcde'`，在移动一次之后结果就是`'bcdea'` 。如果在若干次旋转操作之后，`A` 能变成`B`，那么返回`True`。

```
示例 1:
输入: A = 'abcde', B = 'cdeab'
输出: true

示例 2:
输入: A = 'abcde', B = 'abced'
输出: false
```

**注意：**

- `A` 和 `B` 长度不超过 `100`。

```java
class Solution {
    public boolean rotateString(String A, String B) {
         if (A.length() != B.length()) return false;
        A = A + A;
        return A.contains(B);
    }
}
```

# [125. 验证回文串](https://leetcode-cn.com/problems/valid-palindrome/)

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

**说明：**本题中，我们将空字符串定义为有效的回文串。

**示例 1:**

```
输入: "A man, a plan, a canal: Panama"
输出: true
```

**示例 2:**

```
输入: "race a car"
输出: false
```

**思路**：

双指针。检测是否是字母或数字，全部转化为小写，不相等就false

```java
class Solution {
    public boolean isPalindrome(String s) {
       if (s == null) return true;
       int i = 0;
        int j = s.length() - 1;
        while(i<j){
            while(i < j && !Character.isLetterOrDigit(s.charAt(i))) i++;
            while(i < j && !Character.isLetterOrDigit(s.charAt(j))) j--;
            if(Character.toLowerCase(s.charAt(i)) != Character.toLowerCase(s.charAt(j)))
                return false;
            i++;
            j--;
        }
        return true;
    }
}
```

**思路**：

全部转换为小写，计算字符串的长度，新建一个StringBuilder，转换为字符串数组，如果是字母或数组，就append，最后再转为字符串和反转的字符转比较，用reverse。

```java
class Solution {
    public boolean isPalindrome(String s) {
       if (s == null) return true;
       int i = 0;
        int l = s.length();
        StringBuilder sb = new StringBuilder();
        for(char c : s.toCharArray()){
            if((c >= '0' && c <= '9') || (c >= 'a' && c <= 'z')){
                sb.append(c);
            }              
        }
        return sb.toString().equals(sb.reverse().toString);
    }
}
```

# [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

给定一个字符串 `s`，找到 `s` 中最长的回文子串。你可以假设 `s` 的最大长度为 1000。

**示例 1：**

```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

**示例 2：**

```
输入: "cbbd"
输出: "bb"
```

**思路**：

我们创建一个二维数组，boolean[][] dp,其中dp[i][j]表示字符串第i到j是否为回文。那么边界值其实很清楚了，字符串长度为1的都为true。状态转换如何设定呢？当字符串i所在的字符等于字符串j所在的字符，并且它的内部(dp[i+1][j-1])为回文那么dp[i][j]为true。因为这样的规律，我们要保证判断dp[i][j]的时候dp[i+1][j-1]已经判断，所以我们遍历采用i降序j升序的嵌套遍历的方式

作者：sxqiong

链接：https://www.jianshu.com/p/a7741619dd58

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s.isEmpty()) {
            return s;
        }
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        int left = 0;
        int right = 0;
        for (int i = n - 2; i >= 0; i--) {
            dp[i][i] = true;
            for (int j = i + 1; j < n; j++) {
                dp[i][j] = s.charAt(i) == s.charAt(j) &&( j-i<3||dp[i+1][j-1]);//小于3是因为aba一定是回文
                if(dp[i][j]&&right-left<j-i){
                    left=i;
                    right=j;
                }
            }
        }
        return s.substring(left,right+1);
    }
}
```

# [131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

给定一个字符串 s，将 s 分割成一些子串，使每个子串都是回文串。

返回 s 所有可能的分割方案。

**示例:**

```
输入: "aab"
输出:
[
  ["aa","b"],
  ["a","a","b"]
]
```

**思路：**

首先，对于一个字符串的分割，肯定需要将所有分割情况都遍历完毕才能判断是不是回文数。不能因为 **abba** 是回文串，就认为它的所有子串都是回文的。

既然需要将所有的分割方法都找出来，那么肯定需要用到DFS（深度优先搜索）或者BFS（广度优先搜索）。

在分割的过程中对于每一个字符串而言都可以分为两部分：左边一个回文串加右边一个子串，比如 "abc" 可分为 "a" + "bc" 。 然后对"bc"分割仍然是同样的方法，分为"b"+"c"。

**在处理的时候去优先寻找更短的回文串，然后回溯找稍微长一些的回文串分割方法，不断回溯，分割，直到找到所有的分割方法。**

举个🌰：分割"aac"。

1. 分割为 a + ac
2. 分割为 a + a + c，分割后，得到一组结果，再回溯到  a + ac
3. a + ac 中 ac 不是回文串，继续回溯，回溯到 aac
4. 分割为稍长的回文串，分割为 aa + c 分割完成得到一组结果，再回溯到 aac
5. aac 不是回文串，搜索结束

**代码**：

```java
class Solution {
    List<List<String>> res = new ArrayList<>();
    
    public List<List<String>> partition(String s) {
        if(s == null || s.length() == 0) 
            return res;
        dfs(s, new ArrayList<String>(), 0);
        return res;
    }
    
    public void dfs(String s, List<String> remain, int left){
        if(left == s.length()){ //判断终止条件
            res.add(new ArrayList<String>(remain)); //添加到结果中
            return;
        }
        for(int right=left;right<s.length();right++){//从left开始，依次判断left->right是不是回文串
            if(isPalindroom(s,left,right)){//判断是否是回文串
                remain.add(s.substring(left, right + 1)); // 添加到当前回文串到list中
                dfs(s,remain,right+1); //从right+1开始继续递归，寻找回文串
                remain.remove(remain.size()-1); //回溯，从而寻找更长的回文串
            }
        }
    }
     /**
    * 判断是否是回文串
    */
    public boolean isPalindroom(String s,int left,int right){
        while(left<right&&s.charAt(left)==s.charAt(right)){
            left++;
            right--;
        }
        return left>=right;
    }
}
```

# [35.将字符串转换为数字](https://www.cnblogs.com/edisonchou/p/4824335.html) （剑指offer)

将一个字符串转换成一个整数(实现Integer.valueOf(string)的功能，但是string不符合数字要求时返回0)，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0。

**输入描述**:

```
输入一个字符串,包括数字字母符号,可以为空
```

**输出描述**:

```
如果是合法的数值表达则返回该数字，否则返回0
```

示例1

**输入**

```
+2147483647
    1a33
```

**输出**

```
2147483647
    0
```

**思路**

常规思路，先判断第一位是不是符号位，如果有符号，有flag 做标记。
遍历字符串中的每个字符，如果存在非数字的字符，直接返回 0，否则，用当前字符减去'0'得到当前的数字，再进行运算。

**代码**

```java
public class Solution {
    public int StrToInt(String str) {
     if(str.length() == 0)
            return 0;
        int flag = 0;
        if(str.charAt(0) == '+')
            flag = 1;
        else if(str.charAt(0) == '-')
            flag = 2;
        int start = flag > 0 ? 1 : 0;
        long res = 0;
        while(start < str.length()){
            if(str.charAt(start) > '9' || str.charAt(start) < '0')
                return 0;
            res = res * 10 + (str.charAt(start) - '0');
            start ++;
        }
        return flag == 2 ? -(int)res : (int)res;
    }
}
```

