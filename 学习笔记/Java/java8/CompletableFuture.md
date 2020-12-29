

### 背景

多线程优化性能--->将串行操作变成并行操作--->异步化

![](.\img\1.PNG)

```java
//当我们这样写代码的时候就是串行化编程,做完 业务A 再做 业务B
doBusinessA();
doBusinessB();
```

我们进行异步化操作

![](.\img\2.PNG)

```java
//创建两个线程
new Thread(()->doBusinessA()).start();
new Thread(()->doBusinessB()).start();
```

这两个线程看起来没有任何关系再让我们异步化的优化性能变得很容易，

但是实际业务中，任务之间的关系就是错综复杂(任务之间的并行,串行,聚合...)

![](.\img\3.PNG)

这种对任务进行编排在一定程度上就可以使用CompletableFuture

`所以说提前的设计多有必要,需求-->业务-->代码`

`优化性能还是得先理解业务`

### CompletableFuture什么东西？

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> 
```

对于一个异步操作，你需要关注两个问题：一个是异步操作什么时候结束，另一个是如何获取异步操作的执行结果。

这两个问题你都可以通过 Future 接口来解决

首先得看懂 `Future<T>` 表示一个带返回值的异步计算过程，内部方法包括获取返回值，cancel任务，查看任务是否完成等。

![](.\img\future.PNG)

单纯使用Future<T>的话 我们使用get()，那在Future 计算完成之前会一直处在 blocking 状态下对于真正的异步处理，我们希望的是可以通过传入回调函数，在Future 结束时自动调用该回调函数，这样，我们就不用等待结果.

再看看 `CompletionStage<T>` 是任务链的中间类，更多的体现在**任务的编排**，表示一个阶段状态。



------

### 创建CompletableFuture

```java
//返回一个已经创建好的CompletableFuture
public static <U> CompletableFuture<U> completedFuture(U value) {
        return new CompletableFuture<U>((value == null) ? NIL : value);
    }
//没有返回值,使用ForkJoinPool.commonPool()作为线程池执行任务
public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(asyncPool, runnable);
    }
//没有返回值,使用指定线程池执行任务        
public static CompletableFuture<Void> runAsync(Runnable runnable,
                                                   Executor executor) {
        return asyncRunStage(screenExecutor(executor), runnable);
    }
//有返回值,使用ForkJoinPool.commonPool()作为线程池执行任务
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return asyncSupplyStage(asyncPool, supplier);
    }
//有返回值,使用指定线程池执行任务
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                       Executor executor) {
        return asyncSupplyStage(screenExecutor(executor), supplier);
    }
```

总结一下:

​	`run开头的方法都没有返回值，需要提供返回值就找supply开头的方法`

​	`创建时不自己创建线程池的话就是会使用的JoinForkPool线程池` ForkJoinPool线程池的大小取决于CPU的核数,当然还是建议自己根据业务创建线程池

IO操作核心线程数被占用,线程池资源被占用

CPU操作并不会影响

线程池设置:

:sunflower:：方法不以Async结尾就意味着使用相同线程执行

​	    以Async结尾就意味着任务提交到线程池执行

```java

CompletableFuture<Integer> future = CompletableFuture.completedFuture(100);

CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> System.out.print("hello"));

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "hello");

System.out.prinrtln(future2.get());
```

------

### 串行关系



| 方法类型    | 入参          | 出参                  |      |
| ----------- | ------------- | --------------------- | ---- |
| thenApply   | Function<T,R> | CompletionStage<R>    |      |
| thenAccept  | Consumer<T>   | CompletionStage<Void> |      |
| thenRun     | Runnable      | CompletionStage<Void> |      |
| thenCompose | Function<T,R> | CompletionStage<R>    |      |

````java
CompletionStage<R> thenApply(fn);
CompletionStage<R> thenApplyAsync(fn);

CompletionStage<Void> thenAccept(consumer);
CompletionStage<Void> thenAcceptAsync(consumer);

CompletionStage<Void> thenRun(action);
CompletionStage<Void> thenRunAsync(action);

CompletionStage<R> thenCompose(fn);
CompletionStage<R> thenComposeAsync(fn);
````

````java
CompletableFuture f0 =  CompletableFuture.supplyAsync(() -> "Hello World")      //A
                                      .thenApply(s -> s + " QQ")  			   //B
                                      .thenApply(String::toUpperCase);          //C

System.out.println(f0.join());
//输出结果
HELLO WORLD QQ
````

### 

------

### 聚合关系

#### AND

````java
CompletionStage<R> thenCombine(other, fn);
CompletionStage<R> thenCombineAsync(other, fn);

CompletionStage<Void> thenAcceptBoth(other, consumer);
CompletionStage<Void> thenAcceptBothAsync(other, consumer);

CompletionStage<Void> runAfterBoth(other, action);
CompletionStage<Void> runAfterBothAsync(other, action);
````

#### OR

````java
CompletionStage applyToEither(other, fn);
CompletionStage applyToEitherAsync(other, fn);

CompletionStage acceptEither(other, consumer);
CompletionStage acceptEitherAsync(other, consumer);

CompletionStage runAfterEither(other, action);
CompletionStage runAfterEitherAsync(other, action);
````





------

#### 异常处理

````java
//类似于 try{}catch{}中的 catch{}
CompletionStage exceptionally(fn);
//类似于try{}finally{}中的 finally{}
//无论是否发生异常都会执行 whenComplete() 中的回调函数 consumer 和 handle() 中的回调函数 fn
CompletionStage<R> whenComplete(consumer);
CompletionStage<R> whenCompleteAsync(consumer);
CompletionStage<R> handle(fn);
CompletionStage<R> handleAsync(fn);
````

````java
public class Test {
    public static void main(String[] args) {

        CompletableFuture f1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("线程A开始");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程A结束");
            return " A";
        });

        CompletableFuture f3 = f1.thenApplyAsync(res -> {
            System.out.println("线程B异步开始");
            try {
                TimeUnit.MILLISECONDS.sleep(1000 + new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程B结束");
            return res + " B";
        }).thenApply(res -> {
            System.out.println("线程E开始");
            try {
                TimeUnit.MILLISECONDS.sleep(2000 + new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程E结束");
            return res + " E";
        });

        CompletableFuture f4 = f1.thenApplyAsync(res -> {
            System.out.println("线程C异步开始");
            try {
                TimeUnit.MILLISECONDS.sleep(1000 + new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程C结束");
            return res + " C";
        }).thenApply(res -> {
            System.out.println("线程D开始");
            try {
                TimeUnit.MILLISECONDS.sleep(1000 + new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程D结束");
            return res + " D";
        });

        CompletableFuture future = f3.thenCombine(f4, (res3, res4) -> {
            System.out.println("线程F开始");
            System.out.println(res3);
            System.out.println(res4);
            System.out.println("线程F结束");
            return null;
        });

        future.join();
    }
}
````

