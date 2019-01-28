RxJava
==========================
1.x
-------------------------
四个基本概念：        
1. Observable 被观察者		
2. Observer 观察者		
3. subscribe 订阅			
4. 事件			

事件的回调方法：		
1. onNext() 普通事件的回调			
2. onCompleted() 事件完结队列，RxJava不仅把每个事件单独处理，还把他们看作一个队列，当不会再有新的onNext()发出时，需要出发onCompleted()作为标志。				
3. onError() 事件队列异常，触发时，队列自动终止，不会再有新事件发出。与onCompleted互斥，在队列中调用了其中一个，就不会再调用另外一个。
				

基本实现：					

1. 创建Observer，	即观察者，决定事件触发时的行为。			
	
		Observer<String> observer = new Observer<String>() {
            @Override
            public void onNext(String s) {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        };
    	//除此之外，RxJava还内置了一个Observer的抽象类Subscriber
    	Subscriber<String> subscriber = new Subscriber<String>() {
            @Override
            public void onNext(String s) {

            }

            @Override
            public void onError(Throwable t) {

            }

            @Override
            public void onComplete() {

            }
        };
        
2. 创建Observable 被观察者      

		//当Observable被订阅的时候，call方法会自动被调用，这里传入了一个OnSubscribe参数
		Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
    	@Override
    	public void call(Subscriber<? super String> subscriber) {
        	subscriber.onNext("Hello");
        	subscriber.onNext("Hi");
        	subscriber.onNext("Aloha");
        	subscriber.onCompleted();
    		}
		});
		//除create方法外，还有其他方法快捷创建事件队列
		//just(T...): 将传入的参数依次发送出来。
		Observable observable = Observable.just("Hello", "Hi", "Aloha");
		// 将会依次调用：
		// onNext("Hello");
		// onNext("Hi");
		// onNext("Aloha");
		// onCompleted();
		//from(T[]) / from(Iterable<? extends T>) : 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来。
		String[] words = {"Hello", "Hi", "Aloha"};
		Observable observable = Observable.from(words);
		// 将会依次调用：
		// onNext("Hello");
		// onNext("Hi");
		// onNext("Aloha");
		// onCompleted();

3. Subscribe（订阅）
		
		observable.subscribe(observer);
		// 或者：
		observable.subscribe(subscriber);
		
		//Observable.subscribe(Subscriber) 的内部实现是这样的（仅核心代码）：
		// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
		// 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
		public Subscription subscribe(Subscriber subscriber) {
    		subscriber.onStart();
    		onSubscribe.call(subscriber);//订阅时才会发送事件
    		return subscriber;
		}
		
	

Scheduler
------------------------
在不指定线程的情况下，RxJava遵循线程不变的原则，即在哪个线程调用subscribe，就在哪个线程生产事件，在哪个线程生产事件，就在哪个线程消费事件，如果要切换线程，需要用到Scheduler。       

使用subscribeOn和observerOn来对线程进行控制，subscribeOn指定subscribe()发生的线程，即OnSubscribe被激活时也就是发射事件所处的线程，observeOn指定subscriber所运行的线程，即事件消费的线程。            
多次指定发射事件的线程只有第一次指定的有效，也就是说多次调用 subscribeOn() 只有第一次的有效，其余的会被忽略。但多次指定订阅者接收线程是可以的，也就是说每调用一次 observerOn()，下游的线程就会切换一次。

	Observable.just(1, 2, 3, 4)
    .subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
    .observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
    .subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer number) {
            Log.d(tag, "number:" + number);
        }
    });
    //1,2,3,4在io线程发出，Log.d方法在主线程运行，即常见的后台线程取数据，主线程显示
    
    
变换
------------------------------------
RxJava的核心功能之一，将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。

map：事件对象的直接变换

	Observable.just("images/logo.png") // 输入类型 String
    .map(new Func1<String, Bitmap>() {
        @Override
        public Bitmap call(String filePath) { // 参数类型 String
            return getBitmapFromPath(filePath); // 返回类型 Bitmap
        }
    })
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) { // 参数类型 Bitmap
            showBitmap(bitmap);
        }
    });
    // map操作符实际上将Observable<String>转换成了Observable<Bitmap>

flatmap

	//如果有一堆学生，每个学生有多个课程，需要打印出每个学生的课程
	Students[] students = ...;
	Subscriber<Course> subscriber = new Subscriber<Course>(){
		@Override
		public void onNext(Course course) {
			Log.d(tag, course.getName());
		}
	};
	Observable.from(student)
		.flatmap(new Func1<Student, Observable<Course>>() {
			@Override
			public Observable<Course> call(Student student) {
				return Observable.from(students.getCourses());
			}
		})
		.subscribe(subscriber);
	//与map不同的是，flatmap返回的是一个Observable对象，并不是直接被发送到Subscriber的回调方法。原理：1.使用传入的对象创建一个Observable对象2.并不发送这个Observable，而是将它激活，于是它开始发送事件3.每个创建出来的Observable发送的事件，都被汇入同一个Observable，这个observable负责将这些事件统一交给subscriber的回调方法。像是通过一组新创建的Observable将初始的对象铺平之后通过统一路径分发了下去。	
	
变换的原理：lift()				
变换的实质是针对事件序列的处理和再发送，在RxJava内部，他们是基于一个基础的变换方法lift(Operator)

	// 注意：这不是 lift() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
	// 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
	public <R> Observable<R> lift(Operator<? extends R, ? super T> operator) {
    	return Observable.create(new OnSubscribe<R>() {
        	@Override
        	public void call(Subscriber subscriber) {
            	Subscriber newSubscriber = operator.call(subscriber);
            	newSubscriber.onStart();
            	onSubscribe.call(newSubscriber);
        	}
    	});
	}
	Observable指行了lift之后，会返回一个新的Observable，这个新的Observable会像一个代理一样，负责接收原Observable发出的时间，并在处理后发送给Subscriber
		
		
2.x
----------------------------
一次订阅过程

	Observable.create(new ObservableOnSubscribe<Integer>() {// 1.初始化Observable
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
                e.onComplete();
                e.onNext(4);
            }
        }).subscribe(new Observer<Integer>() {// 3.订阅

            // 2.初始化Observer
            private int i;
            private Disposable mDiposable;

            @Override
            public void onSubscribe(Disposable d) {
                mDiposable = d;
            }

            @Override
            public void onNext(Integer integer) {
                i++;
                if (i == 2) {
                    // Diposable可以做切断操作，让Observer不再接收数据，onComplete也不会执行
                    mDiposable.dispose();
                }
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
        
        
        
        
其他操作符				
zip合并事件
	
	private Observable<String> getStringObservable() {
        return Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext("A");
                    e.onNext("B");
                    e.onNext("C");
                }
            }
        });
    }

    private Observable<Integer> getIntegerObservable() {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext(1);
                    e.onNext(2);
                    e.onNext(3);
                    e.onNext(4);
                    e.onNext(5);
                }
            }
        });
    }
	Observable.zip(getStringObservable(), getIntegerObservable(), new BiFunction<String, Integer, String>() {
            @Override
            public String apply(@NonNull String s, @NonNull Integer integer) throws Exception {
                return s + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(@NonNull String s) throws Exception {
                Log.e(TAG, "zip : accept : " + s + "\n");
            }
        });
    //最终形成的Observable发射事件的数目和少的那个相同
    //A1 B2 C3
   	
   	
concat

	Observable.concat(Observable.just(1,2,3), Observable.just(4,5,6))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "concat : "+ integer + "\n" );
                    }
                });
    //把两个Observable连接成一个Observabl
    //1 2 3 4 5 6
   	
flatMap

	Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
                List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "flatMap : accept : " + s + "\n");
                    }
      	 });
    //flatMap把一个Observable通过某种方法转换为多个Observables，然后把这些分散的Observables装进一个单一的发射器Observable，但不能保证事件的顺序

concatMap 与FlatMap的唯一区别就是保证了顺序

	Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).concatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
                List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "concatMap : accept : " + s + "\n");
                    }
                });
    //输出结果1 1 1 2 2 2 3 3 3 

distinct

	Observable.just(1, 1, 1, 2, 2, 3, 4, 5)
                .distinct()
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "distinct : " + integer + "\n");
                    }
                });
    //去重，结果1 2 3 4 5 
    
filter

	Observable.just(1, 20, 65, -5, 7, 19)
                .filter(new Predicate<Integer>() {
                    @Override
                    public boolean test(@NonNull Integer integer) throws Exception {
                        return integer >= 10;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                mRxOperatorsText.append("filter : " + integer + "\n");
                Log.e(TAG, "filter : " + integer + "\n");
            }
        });
    //20 65 19

buffer(count, skip)，将Observable中的数据按照skip分成最大不超过count的buffer，然后生成一个Observable

	Observable.just(1, 2, 3, 4, 5)
                .buffer(3, 2)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(@NonNull List<Integer> integers) throws Exception {
                        Log.e(TAG, "buffer size : " + integers.size() + "\n");
                        Log.e(TAG, "buffer value : " );
                        for (Integer i : integers) {
                            mRxOperatorsText.append(i + "");
                            Log.e(TAG, i + "");
                        }
                    }
                });	
    // skip为2，count为3，每次跳过2个值，取三个，分别为123 345 5
    
timer

	 Observable.timer(2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        Log.e(TAG, "timer :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                    }
                });
    // Consumer2s后接收到事件
    
interval

	mDisposable = Observable.interval(3, 2, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread()) // 由于interval默认在新线程，所以我们应该切回主线程
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        Log.e(TAG, "interval :" + aLong + " at " + TimeUtil.getNowStrTime() + "\n");
                    }
                });
    //第一次延迟3s后发送，后面每2s发送一次，subscribe返回一个Disposable对象，需要在Activity销毁时调用mDisposable.dispose()方法切断联系
    
doOnNext 实际上不算一个操作符，但经常用到，让订阅者在收到数据之前做一些事情

	Observable.just(1, 2, 3, 4)
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                    	//可以在这里保存数据
                      	Log.e(TAG, "doOnNext 保存 " + integer + "成功" + "\n");
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                Log.e(TAG, "doOnNext :" + integer + "\n");
            }
        });
    //保存1成功 1 保存2成功 2......
   
skip

	Observable.just(1,2,3,4,5)
                .skip(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "skip : "+integer + "\n");
                    }
                });
    //跳过2个数目开始接收
    //3 4 5
    
take

	Flowable.fromArray(1,2,3,4,5)
                .take(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "accept: take : "+integer + "\n" );
                    }
                });
    //指定订阅者最多接受多少个数据
    //1 2
    
just

	Observable.just("1", "2", "3")
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG,"accept : onNext : " + s + "\n" );
                    }
                });
    //依次调用subscriber的onNext方法
    //1 2 3
    
single

	Single.just(new Random().nextInt())
                .subscribe(new SingleObserver<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {

                    }

                    @Override
                    public void onSuccess(@NonNull Integer integer) {
                        Log.e(TAG, "single : onSuccess : "+integer+"\n" );
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "single : onError : "+e.getMessage()+"\n");
                    }
                });
    //single只接收一个参数， SingleObserver只会调用onError()或者onSuccess()
    
debounce

	Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                emitter.onNext(1); // skip
                Thread.sleep(400);
                emitter.onNext(2); // deliver
                Thread.sleep(505);
                emitter.onNext(3); // skip
                Thread.sleep(100);
                emitter.onNext(4); // deliver
                Thread.sleep(605);
                emitter.onNext(5); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        }).debounce(500, TimeUnit.MILLISECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG,"debounce :" + integer + "\n");
                    }
                });
    // 去除发送频率过快的项，去除发送间隔小于500毫秒的发射事件，发射的事件距离上一个发射事件的时间间隔小于debounce方法中设置的参数时，上一个发送的事件会被抛弃
    //2 4 5
    
defer

	Observable<Integer> observable = Observable.defer(new Callable<ObservableSource<Integer>>() {
            @Override
            public ObservableSource<Integer> call() throws Exception {
                return Observable.just(1, 2, 3);
            }
        });


        observable.subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {

            }

            @Override
            public void onNext(@NonNull Integer integer) {
                Log.e(TAG, "defer : " + integer + "\n");
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, "defer : onError : " + e.getMessage() + "\n");
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "defer : onComplete\n");
            }
        });
        
    // 每次订阅都会创建一个新的Observable，如果没有被订阅，就不会产生新的Observable
    // 1 2 3 omComplete
    
last

	Observable.just(1, 2, 3)
                .last(4)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "last : " + integer + "\n");
                    }
                });
    // 取出最后一项，参数是defaultValue，当发射队列为空时，取该值
    // 3
    
merge

	Observable.merge(Observable.just(1, 2), Observable.just(3, 4, 5))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Log.e(TAG, "accept: merge :" + integer + "\n" );
                    }
                });
                
    // 把多个Observable结合起来，与concat区别在于不用等A发射完所有事件再进行B的发射
    // 1 2 3 4 5

reduce

	Observable.just(1, 2, 3)
                .reduce(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                Log.e(TAG, "accept: reduce : " + integer + "\n");
            }
        });
    // 每次用一个方法处理一个值，可以有一个seed作为初始值，最后的值时1+2=3+3=6
    // 6
    
scan

	Observable.just(1, 2, 3)
                .scan(new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                Log.e(TAG, "accept: scan " + integer + "\n");
            }
        });
        
    // 与reduce作用一致，但会把每一步的结果都输出
    // 1 3 6
    
window

	Observable.interval(1, TimeUnit.SECONDS) // 间隔一秒发一次
                .take(15) // 最多接收15个
                .window(3, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Observable<Long>>() {
                    @Override
                    public void accept(@NonNull Observable<Long> longObservable) throws Exception {
                        Log.e(TAG, "Sub Divide begin...\n");
                        longObservable.subscribeOn(Schedulers.io())
                                .observeOn(AndroidSchedulers.mainThread())
                                .subscribe(new Consumer<Long>() {
                                    @Override
                                    public void accept(@NonNull Long aLong) throws Exception {
                                        Log.e(TAG, "Next:" + aLong + "\n");
                                    }
                                });
                    }
                });
    // 按照实际划分窗口，将数据发送给不同的Observable         
    // Sub Divide begin 
    // Next:0
    // Next:1
    // Sub Divide begin 
    // Next:2
    // Next:3
    // Next:4
    // Sub Divide begin 
    // Next:5
    // Next:6
    // Next:7
    // Sub Divide begin 
    // Next:8
    // Next:9
    // Next:10
    // Sub Divide begin 
    // Next:11
    // Next:12
    // Next:13