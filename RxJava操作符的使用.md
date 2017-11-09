# 创建操作

以下操作符用于创建Observable

### create： 
使用OnSubscribe从头创建一个Observable，这种方法比较简单。
需要注意的是，使用该方法创建时，建议在OnSubscribe#call方法中检查订阅状态，以便及时停止发射数据或者运算。


    Observable.create(new Observable.OnSubscribe<String>() {

        @Override
        public void call(Subscriber<? super String> subscriber) {

            subscriber.onNext("item1");
            subscriber.onNext("item2");
            subscriber.onCompleted();
        }
    });

### from： 
将一个Iterable, 一个Future, 或者一个数组，内部通过代理的方式转换成一个Observable。Future转换为OnSubscribe是通过`OnSubscribeToObservableFuture`进行的，Iterable转换通过`OnSubscribeFromIterable`进行。数组通过`OnSubscribeFromArray`转换。 
![image](http://static.zybuluo.com/maplejaw/1a7gi8os6u4kgk7aahzmj1d0/image_1arcl6d0a1iej60e6ccp48qic9.png)
```

 //Iterable
    List<String> list=new ArrayList<>();
    ...
    Observable.from(list)
            .subscribe(new Action1<String>() {
        @Override
        public void call(String s) {

        }
    });

    //Future
     Future<String> futrue= Executors.newSingleThreadExecutor().submit(new Callable<String>() {

        @Override
        public String call() throws Exception {
            Thread.sleep(1000);
            return "maplejaw";
        }
    });

    Observable.from(futrue)
              .subscribe(new Action1<String>() {
        @Override
        public void call(String s) {

        }
    });
;

```
### just： 
将一个或多个对象转换成发射这个或这些对象的一个Observable。如果是单个对象，内部创建的是`ScalarSynchronousObservable`对象。如果是多个对象，则是调用了from方法创建。

### empty：
创建一个什么都不做直接通知完成的Observable
### error： 
创建一个什么都不做直接通知错误的Observable
### never：
创建一个什么都不做的Observable

```
Observable observable1=Observable.empty();//直接调用onCompleted。
Observable observable2=Observable.error(new RuntimeException());//直接调用onError。这里可以自定义异常
Observable observable3=Observable.never();//啥都不做
```
### timer： 
创建一个在给定的延时之后发射数据项为0的`Observable<Long>`,内部通过`OnSubscribeTimerOnce`工作

```
Observable.timer(1000,TimeUnit.MILLISECONDS)
            .subscribe(new Action1<Long>() {
                @Override
                public void call(Long aLong) {
                    Log.d("JG",aLong.toString()); // 0
                }
            });

```
### interval： 
创建一个按照给定的时间间隔发射从0开始的整数序列的`Observable<Long>`，内部通过`OnSubscribeTimerPeriodically`工作。

```
  Observable.interval(1, TimeUnit.SECONDS)
            .subscribe(new Action1<Long>() {
                @Override
                public void call(Long aLong) {
                     //每隔1秒发送数据项，从0开始计数
                     //0,1,2,3....
                }
            });

```
### range： 
创建一个发射指定范围的整数序列的`Observable<Integer>`
```
Observable.range(2,5).subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer integer) {
            Log.d("JG",integer.toString());// 2,3,4,5,6 从2开始发射5个数据
        }
    });
```
### defer： 
只有当订阅者订阅才创建Observable，为每个订阅创建一个新的Observable。内部通过OnSubscribeDefer在订阅时调用Func0创建Observable。

```
  Observable.defer(new Func0<Observable<String>>() {
        @Override
        public Observable<String> call() {
            return Observable.just("hello");
        }
    }).subscribe(new Action1<String>() {
        @Override
        public void call(String s) {
            Log.d("JG",s);
        }
    });
```
# 合并操作

以下操作符用于组合多个Observable。

### concat： 
按顺序连接多个Observables。需要注意的是Observable.concat(a,b)等价于a.concatWith(b)。
```
Observable<Integer> observable1=Observable.just(1,2,3,4);
Observable<Integer>  observable2=Observable.just(4,5,6);

Observable.concat(observable1,observable2)
        .subscribe(item->Log.d("JG",item.toString()));//1,2,3,4,4,5,6
```

### startWith：
在数据序列的开头增加一项数据。startWith的内部也是调用了concat
```
 Observable.just(1,2,3,4,5)
            .startWith(6,7,8)
    .subscribe(item->Log.d("JG",item.toString()));//6,7,8,1,2,3,4,5
```
### merge： 
将多个Observable合并为一个。不同于concat，merge不是按照添加顺序连接，而是按照时间线来连接。其中mergeDelayError将异常延迟到其它没有错误的Observable发送完毕后才发射。而merge则是一遇到异常将停止发射数据，发送onError通知。 

![image](http://static.zybuluo.com/maplejaw/1wkzxcsyuqbw66lnffau12s2/image_1ase1qvgu113o5j8qo3rhoof09.png)

### zip： 
使用一个函数组合多个Observable发射的数据集合，然后再发射这个结果。如果多个Observable发射的数据量不一样，则以最少的Observable为标准进行压合。内部通过OperatorZip进行压合。

```
Observable<Integer>  observable1=Observable.just(1,2,3,4);
Observable<Integer>  observable2=Observable.just(4,5,6);


    Observable.zip(observable1, observable2, new Func2<Integer, Integer, String>() {
        @Override
        public String call(Integer item1, Integer item2) {
            return item1+"and"+item2;
        }
    })
    .subscribe(item->Log.d("JG",item)); //1and4,2and5,3and6
```
![image](http://static.zybuluo.com/maplejaw/bu67z13p279yu074arzslsyd/image_1ard6160913ui3r6orodc41pntm.png)

### combineLatest： 
当两个Observables中的任何一个发射了一个数据时，通过一个指定的函数组合每个Observable发射的最新数据（一共两个数据），然后发射这个函数的结果。

如下图所示：
- Observable1发射完1之后，1是这个Observable1的最新数据，Observable2发射完A之后，A是这个Observable2的最新数据，结果就返回1A。
- 当Observable1发射完2之后，2是这个Observable1的最新数据，Observable2的最新数据还是A，结果就是2A。
- 当Observable2发射完B之后，B是这个Observable2的最新数据, Observable1的最新数据还是2，结果就是2B。
- ....

类似于zip，但是，不同的是zip只有在每个Observable都发射了数据才工作，而combineLatest任何一个发射了数据都可以工作，每次与另一个Observable最近的数据压合。具体请看下面流程图。 
![image](http://static.zybuluo.com/maplejaw/tuo7jn6ijtzsa1c3ak77umtu/image_1ard609fsi3p9n7160iq0r1rqe9.png)

# 过滤操作

### filter： 
过滤数据。内部通过OnSubscribeFilter过滤数据。

```
  Observable.just(3,4,5,6)
            .filter(new Func1<Integer, Boolean>() {
                @Override
                public Boolean call(Integer integer) {
                    return integer>4;
                }
            })
    .subscribe(item->Log.d("JG",item.toString())); //5,6 
```

### ofType： 
过滤指定类型的数据，与filter类似

```
Observable.just(1,2,"3")
            .ofType(Integer.class)
            .subscribe(item -> Log.d("JG",item.toString()));//3

```
### take：
只发射开始的N项数据或者一定时间内的数据。内部通过OperatorTake和OperatorTakeTimed过滤数据。

```
Observable.just(3,4,5,6)
        .take(3)//发射前三个数据项
        .take(100, TimeUnit.MILLISECONDS)//发射100ms内的数据
```

### takeLast： 
只发射最后的N项数据或者一定时间内的数据。内部通过OperatorTakeLast和OperatorTakeLastTimed过滤数据。takeLastBuffer和takeLast类似，不同点在于takeLastBuffer会收集成List后发射

```
 Observable.just(3,4,5,6)
            .takeLast(3)
            .subscribe(integer -> Log.d("JG",integer.toString()));//4,5,6

```
### takeFirst：
提取满足条件的第一项。内部实现源码如下：

```
public final Observable<T> takeFirst(Func1<? super T, Boolean> predicate) {
      return filter(predicate).take(1); //先过滤，后提取
}
```
### first/firstOrDefault：
只发射第一项（或者满足某个条件的第一项）数据，可以指定默认值。
```
Observable.just(3,4,5,6)
            .first()
            .subscribe(integer -> Log.d("JG",integer.toString()));//3

    Observable.just(3,4,5,6)
               .first(new Func1<Integer, Boolean>() {
                   @Override
                   public Boolean call(Integer integer) {
                       return integer>3;
                   }
               }) .subscribe(integer -> Log.d("JG",integer.toString()));//4
```

### last/lastOrDefault：
只发射最后一项（或者满足某个条件的最后一项）数据，可以指定默认值。

### skip：
跳过开始的N项数据或者一定时间内的数据。内部通过OperatorSkip和OperatorSkipTimed实现过滤。

```
Observable.just(3,4,5,6)
            .skip(1)
            .subscribe(integer -> Log.d("JG",integer.toString()));//4,5,6
```
### skipLast：
跳过最后的N项数据或者一定时间内的数据。内部通过OperatorSkipLast和OperatorSkipLastTimed实现过滤。

### elementAt/elementAtOrDefault：
发射某一项数据，如果超过了范围可以的指定默认值。内部通过OperatorElementAt过滤。

```
Observable.just(3,4,5,6)
         .elementAt(2)
         .subscribe(item->Log.d("JG",item.toString())); //5
```
### ignoreElements：
丢弃所有数据，只发射错误或正常终止的通知。内部通过OperatorIgnoreElements实现。

### distinct：
过滤重复数据，内部通过OperatorDistinct实现。

```
 Observable.just(3,4,5,6,3,3,4,9)
       .distinct()
      .subscribe(item->Log.d("JG",item.toString())); //3,4,5,6,9
```
### distinctUntilChanged：
过滤掉连续重复的数据。内部通过OperatorDistinctUntilChanged实现

```
Observable.just(3,4,5,6,3,3,4,9)
       .distinctUntilChanged()
      .subscribe(item->Log.d("JG",item.toString())); //3,4,5,6,3,4,9
```
### throttleFirst：
定期发射Observable发射的第一项数据。内部通过OperatorThrottleFirst实现。

```
Observable.create(subscriber -> {
        subscriber.onNext(1);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(2);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }

        subscriber.onNext(3);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(4);
        subscriber.onNext(5);
        subscriber.onCompleted();

    }).throttleFirst(999, TimeUnit.MILLISECONDS)
            .subscribe(item-> Log.d("JG",item.toString())); //结果为1,3,4
```
### throttleWithTimeout/debounce：
发射数据时，如果两次数据的发射间隔小于指定时间，就会丢弃前一次的数据,直到指定时间内都没有新数据发射时才进行发射

```
Observable.create(subscriber -> {
        subscriber.onNext(1);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(2);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }

        subscriber.onNext(3);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(4);
        subscriber.onNext(5);
        subscriber.onCompleted();

    }).debounce(999, TimeUnit.MILLISECONDS)//或者为throttleWithTimeout(1000, TimeUnit.MILLISECONDS)
            .subscribe(item-> Log.d("JG",item.toString())); //结果为3,5
```

#sample/throttleLast：
定期发射的Observable离时间点最近的数据。内部通过OperatorSampleWithTime实现。
```
Observable.create(subscriber -> {
        subscriber.onNext(1);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(2);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }

        subscriber.onNext(3);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(4);
        subscriber.onNext(5);
        subscriber.onCompleted();

    }).sample(999, TimeUnit.MILLISECONDS)//或者为throttleLast(1000, TimeUnit.MILLISECONDS)
            .subscribe(item-> Log.d("JG",item.toString())); //结果为2,3,5
```
### timeout： 
如果原始Observable过了指定的一段时长没有发射任何数据，就发射一个异常或者使用备用的Observable。
```
   Observable.create(( subscriber) -> {
        subscriber.onNext(1);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw Exceptions.propagate(e);
        }
        subscriber.onNext(2);

        subscriber.onCompleted();

    }).timeout(999, TimeUnit.MILLISECONDS,Observable.just(99,100))//如果不指定备用Observable将会抛出异常
            .subscribe(item-> Log.d("JG",item.toString()),error->Log.d("JG","onError")); 
            //结果为1,99,100  如果不指定备用Observable结果为1,onError
}
```
# 条件/布尔操作
### all：
判断所有的数据项是否满足某个条件，内部通过OperatorAll实现。
```
  Observable.just(2,3,4,5)
            .all(new Func1<Integer, Boolean>() {
                @Override
                public Boolean call(Integer integer) {
                    return integer>3;
                }
            })
    .subscribe(new Action1<Boolean>() {
        @Override
        public void call(Boolean aBoolean) {
            Log.d("JG",aBoolean.toString()); //false
        }
    });
```
### exists： 
判断是否存在数据项满足某个条件。内部通过OperatorAny实现。
```
   Observable.just(2,3,4,5)
            .exists(integer -> integer>3)
            .subscribe(aBoolean -> Log.d("JG",aBoolean.toString())); //true
```
### contains： 
判断在发射的所有数据项中是否包含指定的数据，内部调用的其实是exists
```
  Observable.just(2,3,4,5)
            .contains(3)
            .subscribe(aBoolean -> Log.d("JG",aBoolean.toString())); //true

```
### sequenceEqual： 
用于判断两个Observable发射的数据是否相同（数据，发射顺序，终止状态）。
```
Observable.sequenceEqual(Observable.just(2,3,4,5),Observable.just(2,3,4,5))
            .subscribe(aBoolean -> Log.d("JG",aBoolean.toString()));//true

```
### isEmpty：
用于判断Observable发射完毕时，有没有发射数据。有数据false，如果只收到了onComplete通知则为true。
```
  Observable.just(3,4,5,6)
               .isEmpty()
              .subscribe(item -> Log.d("JG",item.toString()));//false
```
### amb： 
给定多个Observable，只让第一个发射数据的Observable发射全部数据，其他Observable将会被忽略。
```
    Observable<Integer> observable1=Observable.create(new Observable.OnSubscribe<Integer>() {
        @Override
        public void call(Subscriber<? super Integer> subscriber) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                subscriber.onError(e);
            }
            subscriber.onNext(1);
            subscriber.onNext(2);
            subscriber.onCompleted();
        }
    }).subscribeOn(Schedulers.computation());

    Observable<Integer> observable2=Observable.create(subscriber -> {
        subscriber.onNext(3);
        subscriber.onNext(4);
        subscriber.onCompleted();
    });

    Observable.amb(observable1,observable2)
    .subscribe(integer -> Log.d("JG",integer.toString())); //3,4
```
### switchIfEmpty： 
如果原始Observable正常终止后仍然没有发射任何数据，就使用备用的Observable。
```
Observable.empty()
            .switchIfEmpty(Observable.just(2,3,4))
    .subscribe(o -> Log.d("JG",o.toString())); //2,3,4
```
### defaultIfEmpty：
如果原始Observable正常终止后仍然没有发射任何数据，就发射一个默认值,内部调用的switchIfEmpty。
### takeUntil： 
当发射的数据满足某个条件后（包含该数据），或者第二个Observable发送完毕，终止第一个Observable发送数据。

```
Observable.just(2,3,4,5)
            .takeUntil(new Func1<Integer, Boolean>() {
                @Override
                public Boolean call(Integer integer) {
                    return integer==4;
                }
            }).subscribe(integer -> Log.d("JG",integer.toString())); //2,3,4
```
### takeWhile： 
当发射的数据满足某个条件时（不包含该数据），Observable终止发送数据。
```
Observable.just(2,3,4,5)
            .takeWhile(new Func1<Integer, Boolean>() {
                @Override
                public Boolean call(Integer integer) {
                    return integer==4;
                }
            })
            .subscribe(integer -> Log.d("JG",integer.toString())); //2,3
```
### skipUntil： 
丢弃Observable发射的数据，直到第二个Observable发送数据。（丢弃条件数据）
### skipWhile： 
丢弃Observable发射的数据，直到一个指定的条件不成立（不丢弃条件数据）

# 聚合操作
### reduce：
对序列使用reduce()函数并发射最终的结果,内部使用OnSubscribeReduce实现。

```
  Observable.just(2,3,4,5)
            .reduce(new Func2<Integer, Integer, Integer>() {
                @Override
                public Integer call(Integer sum, Integer item) {
                    return sum+item;
                }
            })
            .subscribe(integer -> Log.d("JG",integer.toString()));//14
```
### collect： 
使用collect收集数据到一个可变的数据结构。
```
Observable.just(3,4,5,6)
               .collect(new Func0<List<Integer>>() { //创建数据结构

                   @Override
                   public List<Integer> call() {
                       return new ArrayList<Integer>();
                   }
               }, new Action2<List<Integer>, Integer>() { //收集器
                   @Override
                   public void call(List<Integer> integers, Integer integer) {
                       integers.add(integer);
                   }
               })
              .subscribe(new Action1<List<Integer>>() {
                  @Override
                  public void call(List<Integer> integers) {

                  }
              });
```
### count/countLong：
计算发射的数量，内部调用的是reduce

# 转换操作
### toList： 
收集原始Observable发射的所有数据到一个列表，然后返回这个列表.
```
Observable.just(2,3,4,5)
            .toList()
            .subscribe(new Action1<List<Integer>>() {
                @Override
                public void call(List<Integer> integers) {

                }
            });
```
### toSortedList： 
收集原始Observable发射的所有数据到一个有序列表，然后返回这个列表。

```
   Observable.just(6,2,3,4,5)
            .toSortedList(new Func2<Integer, Integer, Integer>() {//自定义排序
                @Override
                public Integer call(Integer integer, Integer integer2) {
                    return integer-integer2; //>0 升序 ，<0 降序
                }
            })
            .subscribe(new Action1<List<Integer>>() {
                @Override
                public void call(List<Integer> integers) {
                    Log.d("JG",integers.toString()); // [2, 3, 4, 5, 6]
                }
            });
```
### toMap： 
将序列数据转换为一个Map。我们可以根据数据项生成key和生成value

```
    Observable.just(6,2,3,4,5)
            .toMap(new Func1<Integer, String>() {
                @Override
                public String call(Integer integer) {
                    return "key：" + integer; //根据数据项生成map的key
                }
            }, new Func1<Integer, String>() {
                @Override
                public String call(Integer integer) {
                    return "value："+integer; //根据数据项生成map的kvalue
                }
            }).subscribe(new Action1<Map<String, String>>() {
        @Override
        public void call(Map<String, String> stringStringMap) {
            Log.d("JG",stringStringMap.toString()); // {key：6=value：6, key：5=value：5, key：4=value：4, key：2=value：2, key：3=value：3}
        }
    });
```
### toMultiMap：
类似于toMap，不同的地方在于map的value是一个集合

# 变换操作
map： 对Observable发射的每一项数据都应用一个函数来变换。

```
 Observable.just(6,2,3,4,5)
            .map(integer -> "item:"+integer)
            .subscribe(s -> Log.d("JG",s));//item:6,item:2....
```
### cast： 
在发射之前强制将Observable发射的所有数据转换为指定类型
### flatMap： 
将Observable发射的数据变换为Observables集合，然后将这些Observable发射的数据平坦化的放进一个单独的Observable，内部采用merge合并。

```
 Observable.just(2,3,5)
            .flatMap(new Func1<Integer, Observable<String>>() {
                @Override
                public Observable<String> call(Integer integer) {
                    return Observable.create(subscriber -> {
                        subscriber.onNext(integer*10+"");
                        subscriber.onNext(integer*100+"");
                        subscriber.onCompleted();
                    });
                }
            })
    .subscribe(o -> Log.d("JG",o)) //20,200,30,300,50,500
```
### flatMapIterable： 
和flatMap的作用一样，只不过生成的是Iterable而不是Observable。
```
  Observable.just(2,3,5)
            .flatMapIterable(new Func1<Integer, Iterable<String>>() {
                @Override
                public Iterable<String> call(Integer integer) {
                    return Arrays.asList(integer*10+"",integer*100+"");
                }
            }).subscribe(new Action1<String>() {
              @Override
              public void call(String s) {

              }
    });
```
### concatMap： 
类似于flatMap，由于内部使用concat合并，所以是按照顺序连接发射。
### switchMap： 
和flatMap很像，将Observable发射的数据变换为Observables集合，当原始Observable发射一个新的数据（Observable）时，它将取消订阅前一个Observable。


```
Observable.create(new Observable.OnSubscribe<Integer>() {

        @Override
        public void call(Subscriber<? super Integer> subscriber) {
            for(int i=1;i<4;i++){
                subscriber.onNext(i);
                Utils.sleep(500,subscriber);//线程休眠500ms
            }

            subscriber.onCompleted();
        }
    }).subscribeOn(Schedulers.newThread())
      .switchMap(new Func1<Integer, Observable<Integer>>() {
             @Override
           public Observable<Integer> call(Integer integer) {
                   //每当接收到新的数据，之前的Observable将会被取消订阅
                    return Observable.create(new Observable.OnSubscribe<Integer>() {
                        @Override
                        public void call(Subscriber<? super Integer> subscriber) {
                            subscriber.onNext(integer*10);
                            Utils.sleep(500,subscriber);
                            subscriber.onNext(integer*100);
                            subscriber.onCompleted();
                        }
                    }).subscribeOn(Schedulers.newThread());
                }
            })
            .subscribe(s -> Log.d("JG",s.toString()));//10,20,30,300
```

















