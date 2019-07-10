# **Fork/Join框架介绍**

Fork/Join框架是Java7提供的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。使用工作窃取（work-stealing）算法，主要用于实现“分而治之”。

# **工作窃取算法介绍**

工作窃取（work-stealing）算法优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。

![图片描述](https://img.mukewang.com/5abf3c33000156c304140358.png)

# **Fork/Join框架基础类**

- ForkJoinPool： 用来执行Task，或生成新的ForkJoinWorkerThread，执行 ForkJoinWorkerThread 间的 work-stealing 逻辑。ForkJoinPool 不是为了替代 ExecutorService，而是它的补充，在某些应用场景下性能比 ExecutorService 更好。
- ForkJoinTask： 执行具体的分支逻辑，声明以同步/异步方式进行执行
- ForkJoinWorkerThread： 是 ForkJoinPool 内的 worker thread，执行
- ForkJoinTask, 内部有 ForkJoinPool.WorkQueue来保存要执行的ForkJoinTask。
- ForkJoinPool.WorkQueue：保存要执行的ForkJoinTask。

# **基本思想**

- ForkJoinPool 的每个工作线程都维护着一个工作队列（WorkQueue），这是一个双端队列（Deque），里面存放的对象是任务（ForkJoinTask）。
- 每个工作线程在运行中产生新的任务（通常是因为调用了 fork()）时，会放入工作队列的队尾，并且工作线程在处理自己的工作队列时，使用的是 LIFO 方式，也就是说每次从队尾取出任务来执行。
- 每个工作线程在处理自己的工作队列同时，会尝试窃取一个任务（或是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的工作队列），窃取的任务位于其他线程的工作队列的队首，也就是说工作线程在窃取其他工作线程的任务时，使用的是 FIFO 方式。
- 在遇到 join() 时，如果需要 join 的任务尚未完成，则会先处理其他任务，并等待其完成。
- 在既没有自己的任务，也没有可以窃取的任务时，进入休眠。

# **代码演示**

大家学习时，通常借助的例子都类似于下面这段：

```java
@Slf4j
public class ForkJoinTaskExample extends RecursiveTask<Integer> {

    public static final int threshold = 2;
    private int start;
    private int end;

    public ForkJoinTaskExample(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;

        //如果任务足够小就计算任务
        boolean canCompute = (end - start) <= threshold;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            ForkJoinTaskExample leftTask = new ForkJoinTaskExample(start, middle);
            ForkJoinTaskExample rightTask = new ForkJoinTaskExample(middle + 1, end);

            // 执行子任务
            leftTask.fork();
            rightTask.fork();

            // 等待任务执行结束合并其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();

            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkjoinPool = new ForkJoinPool();

        //生成一个计算任务，计算1+2+3+4
        ForkJoinTaskExample task = new ForkJoinTaskExample(1, 100);

        //执行一个任务
        Future<Integer> result = forkjoinPool.submit(task);

        try {
            log.info("result:{}", result.get());
        } catch (Exception e) {
            log.error("exception", e);
        }
    }
}
```

**重点注意**

需要特别注意的是：

1. ForkJoinPool 使用submit 或 invoke 提交的区别：invoke是同步执行，调用之后需要等待任务完成，才能执行后面的代码；submit是异步执行，只有在Future调用get的时候会阻塞。

2. 这里继承的是RecursiveTask，还可以继承RecursiveAction。前者适用于有返回值的场景，而后者适合于没有返回值的场景

3. 这一点是最容易忽略的地方，其实这里执行子任务调用fork方法并不是最佳的选择，最佳的选择是invokeAll方法。

   ```
   leftTask.fork();  
   rightTask.fork();
   
   替换为
   
   invokeAll(leftTask, rightTask);
   ```

具体说一下原理：对于Fork/Join模式，假如Pool里面线程数量是固定的，那么调用子任务的fork方法相当于A先分工给B，然后A当监工不干活，B去完成A交代的任务。所以上面的模式相当于浪费了一个线程。那么如果使用invokeAll相当于A分工给B后，A和B都去完成工作。这样可以更好的利用线程池，缩短执行的时间。


作者：Jimin链接：https://www.imooc.com/article/24822来源：慕课网本文原创发布于慕课网 ，转载请注明出处，谢谢合作