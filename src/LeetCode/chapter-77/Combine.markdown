# Combine
> 给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。 [链接](https://leetcode-cn.com/problems/combinations/)

## 1.解题思路一 回溯
### 1.1 思路
![图示](./images/tree.png)
> 额，没啥可说的，把图画出来思路就清晰了，剩下的就差实现了,还是先画图。本题重要的是优化后的回溯法，请往下看。
### 1.2 具体实现
```
public class Combine {

    private static List<List<Integer>> ret = new ArrayList<>();

    private static void handle(int n, int k, int now, List<Integer> list){
        if(list.size() == k){
            ret.add(list);
        }
        for (int i = now; i <= n; i++) {
            List<Integer> temp = new ArrayList<>(list);
            temp.add(i);
            handle(n,k,i+1,temp);
        }
    }

    public static List<List<Integer>> combine(int n, int k) {
        ret.clear();
        if(n == 0 || k == 0) return ret;
        handle(n,k,1,new ArrayList<>());
        return ret;
    }

    public static void main(String[] args) {
        System.out.println(combine(4,2));
    }
}

```

## 2. 优化回溯法 剪枝
### 2.1 思路
> 
###
```
public class Combine {

    private static List<List<Integer>> ret = new ArrayList<>();

    private static void handle(int n, int k, int now, List<Integer> list){
        if(list.size() == k){
            ret.add(list);
            return;
        }
        for (int i = now; i <= n-(k-list.size())+1; i++) { // i <= n-(k-list.size())+1 剪枝
            List<Integer> temp = new ArrayList<>(list);
            temp.add(i);
            handle(n,k,i+1,temp);
        }
    }

    public static List<List<Integer>> combine(int n, int k) {
        ret.clear();
        if(n == 0 || k == 0) return ret;
        handle(n,k,1,new ArrayList<>());
        return ret;
    }

    public static void main(String[] args) {
        System.out.println(combine(4,2));
    }
}
```