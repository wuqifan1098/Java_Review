# [232. 用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

使用栈实现队列的下列操作：

push(x) -- 将一个元素放入队列的尾部。
pop() -- 从队列首部移除元素。
peek() -- 返回队列首部的元素。
empty() -- 返回队列是否为空。

示例:

```java
MyQueue queue = new MyQueue();

queue.push(1);
queue.push(2);  
queue.peek();  // 返回 1
queue.pop();   // 返回 1
queue.empty(); // 返回 false
```

思路：

```java
class MyQueue {
    private Stack<Integer> in;
    private Stack<Integer> out;
    /** Initialize your data structure here. */
    public MyQueue() {
        in = new Stack<>();
        out = new Stack<>();
    }
    
    /** Push element x to the back of queue. */
    public void push(int x) {
        in.push(x);
    }
    
    /** Removes the element from in front of queue and returns that element. */
    public int pop() {
        if(out.isEmpty() && in.isEmpty()){
            throw new RuntimeException("Queue is empty!");
        }else if(out.isEmpty()){
           while(!in.isEmpty()){
                out.push(in.pop());
            }           
        }
        return out.pop();
    }
    
    /** Get the front element. */
    public int peek() {
       if(out.isEmpty() && in.isEmpty()){
            throw new RuntimeException("Queue is empty!");
        }else if(out.isEmpty()){
           while(!in.isEmpty()){
                out.push(in.pop());
            }           
        }
        return out.peek(); 
    }
    
    /** Returns whether the queue is empty. */
    public boolean empty() {
        return in.isEmpty() && out.isEmpty();
    }
}

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue obj = new MyQueue();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.peek();
 * boolean param_4 = obj.empty();
 */
```

# [225. 用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

使用队列实现栈的下列操作：

push(x) -- 元素 x 入栈
pop() -- 移除栈顶元素
top() -- 获取栈顶元素
empty() -- 返回栈是否为空

```java
public class twoQueueToStack{

    Queue<Integer> q1 = new LinkedList<Integer>();
    Queue<Integer> q2 = new LinkedList<Integer>();
    
    /** Push element x onto stack. */
    public void push(int x) {
        if(q1.isEmpty()){
            q1.add(x);
            for(int i = 0; i < q2.size(); i++){
                q1.add(q2.poll());
            }
        }else{
            q2.add(x);
            for(int i = 0; i < q1.size(); i++){
                q2.add(q1.poll());
            }
        }
    }
    
    /** Removes the element on top of the stack and returns that element. */
    public int pop() {
        return q1.isEmpty() ? q2.poll() : q1.poll();
    }
    
    /** Get the top element. */
    public int top() {
        return q1.isEmpty() ? q2.peek() : q1.peek();
    }
    
    /** Returns whether the stack is empty. */
    public boolean empty() {
        return q1.isEmpty() && q2.isEmpty();
    }
}
```



