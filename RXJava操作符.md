# RXJava

**代码参考：**[https://www.jianshu.com/p/d52ef3ad7460](https://www.jianshu.com/p/d52ef3ad7460)



### 创建操作符

- ```java
  creat
  ```

创建和发射事件完全自定义



- ```java
  just（T item1,  T item2, T item3 ···）
  ```

快速创建被观察者对象（Observable），最多能结接收10个参数

- ```java
  fromArray(T ··· item)
  ```

快速创建被观察者对象（Observable），与just相同，fromArry没有个数限制

- ```java 
  fromIterable(Iterable value)
  ```

将集合中的数据一个一个的发送出去，观察者接收的是集合中的单个item数据



- ```java
  empty()
  ```

发送onSubscribe和onComplete事件，没有onNext

- ```java
  Observable.error(new RuntimeException("报错了哦"))
  ```

发送onSubscribe和onError事件，没有onNext

- ```java
  Observable.never()
  ```

只发送onSubscribe事件



- ```java
  Observable.timer()
  ```

延迟指定时间后发送事件

- ```java
  Observable.interval()
  ```

每隔指定时间 就发送 事件

- ```java
  Observable.intervalRange
  ```

跟interval类似，可以指定事件的个数

- ```java
  Observable.range(start, count)
  ```

从start开始连续发送count个事件



### 变换操作符

------

- ```java
  Obserable.map(Function<? super T, ? extends R> mapper)
  ```

将被观察者发送的事件转换为任意类型的事件

- ```
  Observable.flatMap()
  ```

与map类似，区别在于map是把参数变换（事件变换），而flatMap是把整个被观察者(Observable)变换

- ```java
  Observable.concatMap()
  ```

与flatMap类似，区别在于concatMap能保证新合成的事件序列与旧的序列的顺序一致，而flatMap是不能保证顺序的



### buffer操作符

------

- ```java
  buffer(count)
  ```

最简单的重载将固定数量的值分组在一起，并在组准备就绪时发出该组

```
例：
Observable.just("A", "B", "C", "D").buffer(3).subscribe(new MyConsumer<>());
输出：
[A, B, C]
[D, E]
注释：每次从缓冲区中取3个
```



- ```java
  buffer(count, skip)
  ```

```java
例：
Observable.just("A", "B", "C", "D", "E").buffer(3, 2).subscribe(new MyConsumer<>());
输出：
[A, B, C]
[C, D, E]
[E]
注释：每次从缓冲区中取3个，下一次跳过前面的2个再取三个
```



- ```java
  buffer(long timespan, TimeUnit unit)
  ```

每隔指定时间从缓冲区中获取事件

```java
例：
Observable
        .interval(1, TimeUnit.SECONDS) // 每隔1秒发送一个事件
        .take(10) // 取10个事件
        .buffer(3, TimeUnit.SECONDS) // 每隔3秒从缓冲区中获取缓冲区的事件
        .subscribe(new MyObserver<>());
输出： 
[0, 1]
[2, 3, 4, 5]
[6, 7, 8]
[9]
注释：每次输出可能会不一样，以实际输出为准（事件个数取决于那个时间段发生了多少个）
```

```java
.buffer(long timespan, TimeUnit unit, int count)
```

每隔指定时间和缓冲区到达指定个数时，从缓冲区中获取事件

```java
示例：
Observable
        .interval(1, TimeUnit.SECONDS) // 每隔1秒发送一个事件
        .take(10) // 取10个事件
        .buffer(3, TimeUnit.SECONDS, 2) // 每隔3秒和缓冲区到达2个时，从缓冲区中获取事件
        .subscribe(new MyObserver<>());
输出：
[0, 1]
[]
[2, 3]
[4]
[5, 6]
[7]
[8, 9]
注释：这里看到很多空的，这是因为缓冲区在大小达到2时和时间窗口关闭时都会发出
```





### 组合/合并操作符

------

- ```java
  concat() / concatArray()
  ```

组合多个被观察者一起发送数据，合并后 按发送顺序串行执行

二者区别：组合被观察者的数量，即`concat()`组合被观察者数量≤4个，而`concatArray()`则可＞4个

```java
示例：
Observable.concat(
        Observable.just(1, 2, 3),
        Observable.just(7, 8)
).subscribe(new MyConsumer<>());
输出：
    1、2、3、7、8
```

- ```java
  merge() / mergeArray()
  ```

和`concat`/ `concatArray`相似，区别在于`concat`是严格按照发送顺序执行，而merge不一定有顺序

```java
示例：
Observable.merge(
        Observable.intervalRange(1, 3, 1, 1, TimeUnit.SECONDS),
        Observable.intervalRange(100, 2, 1, 1, TimeUnit.SECONDS)
).subscribe(new MyConsumer<>());
输出：1、100、2、101、3、4
注释：merge是无序，所以每次不一定是上面的输出顺序
```

- ```java
  mergeDelayError() / concatArrayDelayError()
  ```

和`merge`/`concatArray`相似，区别在于DelayError在其中一个发送Error事件时，另外的还会继续发送事件。

- ```java
  zip()
  ```

将多个事件合并（组装）后发送，被观察者最多9个

```java
示例：
Observable
        .zip(Observable.just(1, 2), Observable.just("A", "B", "C"), new BiFunction<Integer, String, String>() {
            @Override
            public String apply(Integer num, String str) throws Throwable {
                return num + str;
            }
        })
        .subscribe(new MyObserver<>());
输出：
onSubscribe
onNext::1A
onNext::2B
onComplete
```

- ```java
  combineLatest()
  ```

当两个`Observables`中的任何一个发送了数据后，将先发送了数据的`Observables` 的最新（最后）一个数据 与 另外一个`Observable`发送的每个数据结合，最终基于该函数的结果发送数据

```java
示例：
Observable
        .combineLatest(
                // 第1个发送数据事件（没有延迟，发送事件很快，事件序列中最新的事件value=5）
                Observable.just(1L, 2L, 5L),
                // 第2个发送数据事件的Observable：从21开始发送、共发送3个数据、第1次事件延迟发送时间 = 1s、间隔时间 = 1s
                Observable.intervalRange(21, 3, 1, 1, TimeUnit.SECONDS),
                new BiFunction<Long, Long, String>() {
                    @Override
                    public String apply(Long num1, Long num2) throws Throwable {
                        return "合并的数据是：【" + num1 + "、" + num2 + "】";
                    }
                })
        .subscribe(new MyObserver<>());

输出结果：
onSubscribe
onNext::合并的数据是：【5、21】
onNext::合并的数据是：【5、22】
onNext::合并的数据是：【5、23】
onComplete
```

- ```java
  reduce()
  ```

把被观察者需要发送的事件聚合成1个事件 & 发送

```java
示例：
Observable
        .range(1, 5)
        .reduce(new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer integer, Integer integer2) throws Throwable {
                System.out.println(String.format("apply:%d+%d=%d", integer, integer2, integer + integer2));
                return integer + integer2;
            }
        })
        .subscribe(new MyConsumer<>());

输出：
apply:1+2=3
apply:3+3=6
apply:6+4=10
apply:10+5=15
最后结果：15	
```

- ```java
  collect()
  ```

将被观察者`Observable`发送的数据事件收集到一个数据结构里，可理解为`fromIterable()`反向操作

``` java
示例：
Observable
        .just(1, 2, 3)
        .collect(
                new Supplier<List<Integer>>() {
                    @Override
                    public List<Integer> get() throws Throwable {
                        return new ArrayList();
                    }
                },
                new BiConsumer<List<Integer>, Integer>() {
                    @Override
                    public void accept(List<Integer> list, Integer integer) throws Throwable {
                        list.add(integer);
                    }
                })
        .subscribe(new MyConsumer<>());

输出：[1, 2, 3]
```



### 发送事件前追加发送事件

------

```java
startWith()/startWithArray()/startWithItem()
```

在一个被观察者发送事件前，追加发送一些数据 / 一个新的被观察者

``` java
示例：
Observable
        .just(6, 7)
        .startWith(Observable.just(4, 5))
        .startWithArray(2, 3)
        .startWithItem(1)
        .subscribe(new MyObserver<>());
        
输出：
onSubscribe
onNext::1
onNext::2
onNext::3
onNext::4
onNext::5
onNext::6
onNext::7
onComplete
```

### 统计发送事件数量

- ```java
  count()
  ```

  ```java
  示例：
  Observable
          .just(1, 2, 3, 4)
          .count()
          .subscribe(new Consumer<Long>() {
              @Override
              public void accept(Long aLong) throws Exception {
                  System.out.println("发送的事件数量 =  " + aLong);
  
              }
          });
  
  输出：发送的事件数量 =  4
  ```

  

### 功能性操作符

------

- ```java
  delay()
  ```

  延迟一段时间再发送事件

  ```java
  // 1. 指定延迟时间
  // 参数1 = 时间；参数2 = 时间单位
  delay(long delay,TimeUnit unit)
  
  // 2. 指定延迟时间 & 调度器
  // 参数1 = 时间；参数2 = 时间单位；参数3 = 线程调度器
  delay(long delay,TimeUnit unit,mScheduler scheduler)
  
  // 3. 指定延迟时间  & 错误延迟
  // 错误延迟，即：若存在Error事件，则如常执行，执行后再抛出错误异常
  // 参数1 = 时间；参数2 = 时间单位；参数3 = 错误延迟参数
  delay(long delay,TimeUnit unit,boolean delayError)
  
  // 4. 指定延迟时间 & 调度器 & 错误延迟
  // 参数1 = 时间；参数2 = 时间单位；参数3 = 线程调度器；参数4 = 错误延迟参数
  delay(long delay,TimeUnit unit,mScheduler scheduler,boolean delayError)
  ```

  

