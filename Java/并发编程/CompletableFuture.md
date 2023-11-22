参考地址：https://juejin.cn/post/6844904195162636295

### 1. 基本介绍

#### 1.1使用场景

用于执行异步任务，可以自定义线程池和使用默认的线程池



### 2.使用

#### 2.1创建CompletableFuture并执行任务

- completeFuture 可以用于创建默认返回值
- runAsync 异步执行，无返回值
- supplyAsync 异步执行，有返回值
- anyOf 任意一个执行完成，就可以进行下一步动作
- allOf 全部完成所有任务，才可以进行下一步任务

```java
CompletableFuture.supplyAsync(()->test(),executor);
```

#### 2.2状态取值

- join 合并结果，等待
- get 合并等待结果，可以增加超时时间;get和join区别，join只会抛出unchecked异常，get会返回具体的异常
- getNow 如果结果计算完成或者异常了，则返回结果或异常；否则，返回valueIfAbsent的值
- isCancelled
- isCompletedExceptionally
- isDone

#### 2.3回调行为

- thenApply(Function<? super T, ? extends U> fn)：当CompletableFuture的计算结果可用时，将结果传递给指定的函数，并返回一个新的CompletableFuture，该CompletableFuture的结果是函数的返回值。
- thenAccept(Consumer<? super T> action)：当CompletableFuture的计算结果可用时，将结果传递给指定的消费者函数，但不返回任何结果。
- thenRun(Runnable action)：当CompletableFuture的计算结果可用时，执行指定的动作，但不接受任何参数和返回任何结果。
- thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)：当CompletableFuture的计算结果可用时，将结果传递给指定的函数，并返回一个新的CompletionStage，该CompletionStage表示函数的结果。
- thenCombine(CompletionStage<? extends U> other, BiFunction<? super T, ? super U, ? extends V> fn)：当两个CompletableFuture都完成时，将它们的结果传递给指定的函数，并返回一个新的CompletableFuture，该CompletableFuture的结果是函数的返回值。
- thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action)：当两个CompletableFuture都完成时，将它们的结果传递给指定的消费者函数，但不返回任何结果。
- runAfterBoth(CompletionStage<?> other, Runnable action)：当两个CompletableFuture都完成时，执行指定的动作，但不接受任何参数和返回任何结果。
- applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn)：当任意一个CompletableFuture完成时，将其结果传递给指定的函数，并返回一个新的CompletableFuture，该CompletableFuture的结果是函数的返回值。
- acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)：当任意一个CompletableFuture完成时，将其结果传递给指定的消费者函数，但不返回任何结果。
- runAfterEither(CompletionStage<?> other, Runnable action)：当任意一个CompletableFuture完成时，执行指定的动作，但不接受任何参数和返回任何结果。
- exceptionally(Function<Throwable, ? extends T> fn)：当CompletableFuture的计算过程中发生异常时，将异常传递给指定的函数，并返回一个新的CompletableFuture，该CompletableFuture的结果是函数的返回值。
- handle(BiFunction<? super T, Throwable, ? extends U> fn)：当CompletableFuture的计算结果可用时，将结果传递给指定的函数，如果计算过程中发生异常，则将异常传递给函数。返回一个新的CompletableFuture，该CompletableFuture的结果是函数的返回值。

```java
 CompletableFuture.supplyAsync(()->test()).whenComplete((res,e)->callbackMethod())
```

#### 2.4任务的编排和依赖场景

零依赖

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
//1、使用runAsync或supplyAsync发起异步调用
CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
  return "result1";
}, executor);
//2、CompletableFuture.completedFuture()直接创建一个已完成状态的CompletableFuture
CompletableFuture<String> cf2 = CompletableFuture.completedFuture("result2");
//3、先初始化一个未完成的CompletableFuture，然后通过complete()、completeExceptionally()，完成该CompletableFuture
CompletableFuture<String> cf = new CompletableFuture<>();
cf.complete("success");
```

一元依赖

```java
CompletableFuture<String> cf3 = cf1.thenApply(result1 -> {
  //result1为CF1的结果
  //......
  return "result3";
});
CompletableFuture<String> cf5 = cf2.thenApply(result2 -> {
  //result2为CF2的结果
  //......
  return "result5";
});
```

二元依赖

```java
CompletableFuture<String> cf4 = cf1.thenCombine(cf2, (result1, result2) -> {
  //result1和result2分别为cf1和cf2的结果
  return "result4";
});
```

多元依赖:使用allof 或者anyof

```java
CompletableFuture<Void> cf6 = CompletableFuture.allOf(cf3, cf4, cf5);
CompletableFuture<String> result = cf6.thenApply(v -> {
  //这里的join并不会阻塞，因为传给thenApply的函数是在CF3、CF4、CF5全部完成时，才会执行 。
  result3 = cf3.join();
  result4 = cf4.join();
  result5 = cf5.join();
  //根据result3、result4、result5组装最终result;
  return "result";
});
```

![图9 多元依赖](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/92248abd0a5b11dd36f9ccb1f1233d4e16045.png?token=AHPLFCKRBKULDSVC7ALRDI3FLWSBE)



### 3.实现原理

CompletableFuture中包含两个字段：**result**和**stack**。result用于存储当前CF的结果，stack（Completion）表示当前CF完成后需要触发的依赖动作（Dependency Actions），去触发依赖它的CF的计算，依赖动作可以有多个（表示有多个依赖它的CF），以栈（[Treiber stack](https://en.wikipedia.org/wiki/Treiber_stack)）的形式存储，stack表示栈顶元素。

![图12 thenApply简图](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/f45b271b656f3ae243875fcb2af36a1141224.png?token=AHPLFCNANZMLMRWE2C3TMU3FLW3MO)



### 4.实践总结

#### 4.1要明确知道代码执行在哪个线程

同步方法（不带Async后缀的方法）：

- 如果注册时被依赖的操作已经执行完成，则直接由当前线程执行。
- 如果注册时被依赖的操作还未执行完，则由回调线程执行。

异步方法：传递了线程池，则取线程池中的线程，否则使用默认线程池CommonPool（核心线程数为CPU核数-1）



#### 4.2异步回调要传线程池

1. 可以减少不同业务之间的相互干扰

2. 线程池的循环应用会导致死锁

   ```java
   public Object doGet() {
     ExecutorService threadPool1 = new ThreadPoolExecutor(10, 10, 0L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(100));
      // 当并发超过10以后，子线程就获取不到线程，但是父线程又依赖子线程执行完成，会出现循环应用死锁
     CompletableFuture cf1 = CompletableFuture.supplyAsync(() -> {
     //do sth
       return CompletableFuture.supplyAsync(() -> {
           System.out.println("child");
           return "child";
         }, threadPool1).join();//子任务
       }, threadPool1);
     return cf1.join();
   }
   ```

3.异步RPC调用注意不要阻塞IO线程池



#### 4.3.要对异常进行处理

由于异步执行的任务在其他线程上执行，而异常信息存储在线程栈中，因此当前线程除非阻塞等待返回结果，否则无法通过try\catch捕获异常。

```java
// 方式1 exceptionally
CompletableFuture<WmOrderOpRemarkResult> remarkResultFuture = wmOrderAdditionInfoThriftService.findOrderCancelledRemarkByOrderIdAsync(orderId);//业务方法，内部会发起异步rpc调用
    return remarkResultFuture
      .exceptionally(err -> {//通过exceptionally 捕获异常，打印日志并返回默认值
         log.error("WmOrderRemarkService.getCancelTypeAsync Exception orderId={}", orderId, err);
         return 0;
      });
// 方式2 whenComplete方法可以拿到exception
CompletableFuture.runAsync(() -> {
                System.out.println(Thread.currentThread().getName()+0);
        }).whenComplete((item,exception) -> {
            System.out.println(Thread.currentThread().getName() + 1);
        });
```

