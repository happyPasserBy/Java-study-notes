# CQueue
>[链接](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)
```
用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

示例 1：
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]
```
## 1. 解题思路一
### 1.2 具体实现
```
class CQueue {

        Stack<Integer> stack1 = new Stack<>();
        Stack<Integer> stack2 = new Stack<>();
        private int size = 0;

        public CQueue() {
        }

        public void appendTail(int value) {
            while (!stack2.isEmpty()){
                stack1.push(stack2.pop());
            }
            stack1.push(value);
            size++;
        }

        public int deleteHead() {
            if(size == 0) return -1;
            while (!stack1.isEmpty()){
                stack2.push(stack1.pop());
            }
            size--;
            return stack2.pop();
        }
}
```


