## links
- CN rxjs dokument: https://cn.rx.js.org/class/es6/Observable.js~Observable.html
- CN rxjs是如何实现的，设计模式，看这一篇就够了 https://juejin.cn/post/6985348097064828942
```
收起
基础概念
核心模块和基本原理
Observable
基本用法
实现原理
静态构造方法实现
Observer
实现原理
Subject
实现原理
BehaviorSubject
ReplaySubject
AsyncSubject
比较
Operator
基本用法
实现原理
参考
❤️
```
```
深入浅出 RxJS 核心原理（源码实现）

字节教育
team icon

2021-07-16
3,265
阅读10分钟
专栏： 
字节教育前端
avatar
前端工程师 @公众号：ELab团队

基础概念
js事件库，通过observable进行异步事件管理
基于观察者模式的响应式编程，生产者主动推送多个数据给订阅的消费者处理
使用纯函数保证应用状态的隔离，保证数据纯净性
通过observable、subject、observer的链式订阅，以及operators提供的数据转换，保证数据在observable中的流动性
核心模块和基本原理
下面通过实现一个简易的rxjs来解析下核心原理

Observable
概念：可观察对象，一个可调用的未来值或事件的集合

基本用法
// 创建observable

let observable = new Observable(function publish(observer) {

  observer.next("hello");

  var id = setTimeout(() => {

    observer.next("world");

    observer.complete();

  }, 1000);

});

// 订阅observable

observable.subscribe({

  next: (value) => console.log(value),

  error: (err) => console.log(err),

  complete: () => console.log("done"),

});

// 输出： hello->world->done
实现原理
根据基本用法，Observable可以执行同步或异步任务，并向observer推送数据，要实现核心功能，只需要如下两个步骤：

创建：作为发布者，observable需要设置一个可执行的publish方法，其入参是observer对象，该方法在构造实例的时候传入，在执行该方法的时候就可以调用observer对象的回调方法进行传值；
订阅：publish方法执行的时机是在observable被subscribe的时候，因此observable是惰性推送值，且对于每个观察者来说是独立执行的；
class Observable {

  constructor(publishFn) {

    this.publish = publishFn;

  }

  subscribe(observer) {

    this.publish(observer);

    return observer;

  }

}
静态构造方法实现
为了方便创建一些既定publish任务的Observable实例，Observable类提供了一些静态构造方法，常用方法包括of/from/fromEvent/interval等；
// 每隔200ms推送由0开始递增的number

const observable = Observable.interval(200);

observable.subscribe(value => console.log(value));

// 输出：0->1->2->....



// 监听document的click事件，推送事件回调的event对象

const observable = Observable.fromEvent(document, "click");

observable.subscribe(event => console.log(event));

// 输出：MouseEvent {isTrusted: true, screenX: 435, screenY: 386, clientX: 435, clientY: 275, …}
实现原理：通过调用构造函数返回一个既定publish方法的observable实例；
fromEvent返回的observable在被订阅时，就会调用target.addEventListener开始事件监听，然后将回调返回的event对象传递给observer
interval返回的observable在被订阅时，就会调用window.setInterval 开始定时任务，累加number，并传递给observer


Observable.fromEvent = function (target, eventName) {

  return new Observable(function (observer) {

    const handler = function (e) {

      observer.next(e);

    };

    target.addEventListener(eventName, handler);

    return () => {

      target.removeEventListener(eventName, handler);

    };

  });

};



Observable.interval = function (delay) {

  return new Observable(function (observer) {

    let index = 0;

    const id = window.setInterval(() => {

      observer.next(index++);

    }, delay);

    return () => {

      clearInterval(id);

    };

  });

};
Observer
概念：观察者， 一个回调函数的集合，它知道如何去监听由 Observable 提供的值

实现原理
作为观察者，需要包含next/error/complete回调方法，用于监听成功/失败/完成返回的值，最简单的observer就是包含回调方法的object
为了维护observer的订阅状态，我们可以封装一个observer类，isStopped属性代表当前是否停止订阅，传入回调方法，并对外提供封装过的回调；
Observer类对外提供unsubscribe方法，用于解除订阅；调用该方法后isStopped为true，数据推送停止，并执行unsubscribe的回调函数unsubscribeCb，该回调函数由对外方法onUnsubscribe传入；
class Observer {

  isStopped = false;

  unsubscribeCb;

  constructor(next, error, complete) {

    this._next = next || noop;

    this._error = error || noop;

    this._complete = complete || noop;

  }

  next(value) {

    if (!this.isStopped) {

      this._next(value);

    }

  }

  error(err) {

    if (!this.isStopped) {

      this._error(err);

      this.unsubscribe();

    }

  }

  complete() {

    if (!this.isStopped) {

      this._complete();

      this.unsubscribe();

    }

  }

  onUnsubscribe(unsubscribeCb) {

    this.unsubscribeCb = unsubscribeCb;

  }

  unsubscribe() {

    this.isStopped = true;

    this.unsubscribeCb && this.unsubscribeCb();

  }

}
根据封装的Observer类，可以进一步优化Observable类的实现

改造subscribe方法，支持传入observer对象或回调函数
将publish方法返回的清理函数传递给observer的onUnsubscribe方法
class Observable {

  constructor(publishFn) {

    this.publish = publishFn;

  }

 subscribe(observerOrNext, error, complete) {

    // 封装observer

    let observer;

    if (

      observerOrNext instanceof Observer ||

      observerOrNext instanceof Subject

    ) {

      observer = observerOrNext;

    } else if (typeof observerOrNext === "function") {

      observer = new Observer(observerOrNext, error, complete);

    } else {

      observer = new Observer(

        observerOrNext.next,

        observerOrNext.error,

        observerOrNext.complete

      );

    }

    // 传递unsubscribe回调清理函数

      const unsubscribeCb = this.publish(observer);

      observer.onUnsubscribe(unsubscribeCb);

      return observer;

    }

  }

}

// 示例

let observable = new Observable(function publish(observer) {

  var id = setTimeout(() => {

    observer.next("helloworld");

    observer.complete();

  }, 1000);

  return () => {

    console.log("clear");

    clearInterval(id);

  };

});



const observer = observable.subscribe(value => console.log(value));

setTimeout(() => observer.unsubscribe(), 2000);

// 输出：helloworld->done->clear
Subject
概念：相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式

上面说到Observable对于每个观察者都会执行一遍publish方法，订阅的数据是独立的，因此它是单播的；subject可以作为observable和observer的中介，通过订阅observable的数据然后分发给observer实现多播

无法复制加载中的内容

下面是observable单播和subject多播的简单示例
// 每隔200ms推送从0开始递增的num，取前6个推送

const observable = Observable.interval(200).pipe(take(6));

const observerA = new Observer((x) => console.log(`A next ${x}`)),

const observerB = new Observer((x) => console.log(`B next ${x}`)),



// observable单播模式，500ms后observerB订阅，重新执行一遍publish

 observable.subscribe(observerA);

 setTimeout(() => {

   observable.subscribe(observerB);

 }, 500);

// 输出：A next 0 -> 1 -> 2 -> 3 -> 4 -> 5

//      B next 0 -> 1 -> 2 -> 3 -> 4 -> 5





// subject 多播模式，500ms后observerB开始接收subject分发的数据，错过了前2个数据

const subject = new Subject();

observable.subscribe(subject);

subject.subscribe(observerA);

setTimeout(() => {

  subject.subscribe(observerB);

}, 500);

// 输出：A next 0 -> 1 -> 2 -> 3 -> 4 -> 5

//      B next 2 -> 3 -> 4 -> 5
实现原理
Subject继承自Observable，同时又实现了Observer的回调方法（next/complete/error）
Subject类维护一个subscribers数组，当Subject被observer订阅时，会执行publish方法将observer push到subscribers数组中；
Subject订阅Observable后，Observable向Subject推送数据，Subject再分发给数组中每个observer
class Subject extends Observable {

  subscribers = [];

  isStopped = false;

  publish(observer) {

    if (this.isStopped) {

      observer.complete();

    }

    // 添加订阅item

    this.subscribers.push(observer);

  }

  next(value) {

    if (this.isStopped) return;

    // 分发数据

    this.subscribers.forEach((observer) => {

      observer.next(value);

    });

  }

  error(error) {

    this.subscribers.forEach((observer) => {

      observer.error(error);

    });

    this.isStopped = true;

    this.subscribers = [];

  }

  complete() {

    this.subscribers.forEach((observer) => {

      observer.complete();

    });

    this.isStopped = true;

    this.subscribers = [];

  }

}
BehaviorSubject
继承Subject，维护当前最新值lastValue，observer订阅时立即传递最新值，防止订阅过晚引起的状态丢失；

示例
 // 示例

const observable = Observable.interval(200).pipe(take(6));

const observerA = new Observer((x) => console.log(`A next ${x}`)),

const observerB = new Observer((x) => console.log(`B next ${x}`)),

// 500ms后observerB开始接收subject分发的数据，能获取到最新数据1

const subject = new BehaviorSubject();

observable.subscribe(subject);

subject.subscribe(observerA);

setTimeout(() => {

  subject.subscribe(observerB); 

}, 500);

// 输出：A next 0 -> 1 -> 2 -> 3 -> 4 -> 5

//      B next 1 -> 2 -> 3 -> 4 -> 5
原理
class BehaviorSubject extends Subject {

  lastValue;

  constructor(value) {

    super();

    this.lastValue = value;

  }

  publish(observer) {

    if (!observer.isStopped) {

      // 被订阅时立即推送最新值

      observer.next(this.lastValue);

    }

    super.publish(observer);

  }

  next(value) {

    this.lastValue = value;

    super.next(value);

  }

}
ReplaySubject
和BehaviorSubject类似，根据bufferSize和windowSize，缓存某个时间段内多个最新值；若windowSize缺省，则最多缓存bufferSize个最近值；若windowSize存在，则缓存最近的windowSize时间窗口内的不超过bufferSize个值；

示例
const observable = Observable.interval(200).pipe(take(6));

const observerA = new Observer((x) => console.log(`A next ${x}`)),

const observerB = new Observer((x) => console.log(`B next ${x}`)),

// 500ms后observerB开始接收subject分发的数据，能获取到最新的3个缓存值

const subject = new ReplaySubject(3);

observable.subscribe(subject);

subject.subscribe(observerA);

setTimeout(() => {

  subject.subscribe(observerB); 

}, 500);

// 输出：A next 0 -> 1 -> 2 -> 3 -> 4 -> 5

//      B next 0 -> 1 -> 2 -> 3 -> 4 -> 5



// 500ms后observerB开始接收subject分发的数据，能获取到最新的200ms内的缓存值

const subject = new ReplaySubject(100, 200);

observable.subscribe(subject);

subject.subscribe(observerA);

setTimeout(() => {

  subject.subscribe(observerB); 

}, 500);

// 输出：A next 0 -> 1 -> 2 -> 3 -> 4 -> 5

//      B next 1 -> 2 -> 3 -> 4 -> 5
原理
class ReplaySubject extends Subject {

  bufferSize = 1;

  windowSize;

  events = []; // 缓存数组，格式为[[time, value], ....]

  constructor(bufferSize, windowSize) {

    super();

    this.bufferSize = Math.max(1, bufferSize);

    this.windowSize = windowSize || 0;

  }

  // 计算缓存数组

  getEvents() {

    let spliceIndex = 0;

    let len = this.events.length;

    if (this.windowSize > 0) {

      let beginTime = Date.now() - this.windowSize;

      while (spliceIndex < len && this.events[spliceIndex][0] <= beginTime) {

        spliceIndex++;

      }

    }

    spliceIndex = Math.max(spliceIndex, len - this.bufferSize);

    spliceIndex > 0 && this.events.splice(0, spliceIndex);

  }

  publish(observer) {

    this.getEvents();

    // 被订阅后立即推送当前所有缓存值

    this.events.forEach((event) => {

      !observer.isStopped && observer.next(event[1]);

    });

    super.publish(observer);

  }

  next(value) {

    // 缓存推送值和时间戳

    this.events.push([Date.now(), value]);

    // 更新缓存数组

    this.getEvents();

    super.next(value);

  }

}
AsyncSubject
只有在事件完成时，才会广播最终的值

示例
const observable = Observable.interval(200).pipe(take(6));

const observerA = new Observer((x) => console.log(`A next ${x}`)),

const observerB = new Observer((x) => console.log(`B next ${x}`)),

// observerA和observerB 接收最终数据5

const subject = new AsyncSubject();

observable.subscribe(subject);

subject.subscribe(observerA);

setTimeout(() => {

  subject.subscribe(observerB); 

}, 500);

// 输出：A next 5

//      B next 5
原理
class AsyncSubject extends Subject {

  hasNext = false;

  hasComplete = false;

  value;

  publish(observer) {

    if (this.hasComplete && this.hasNext) {

      observer.next(this.value);

    }

    super.publish(observer);

  }

  next(value) {

    // 还未结束就不推送，仅保存值

    if (!this.hasComplete) {

      this.value = value;

      this.hasNext = true;

    }

  }

  error(err) {

    if (!this.hasComplete) {

      super.error(err);

    }

  }

  complete() {

    this.hasComplete = true;

    if (this.hasNext) {

       // 任务完成则推送最终值

      super.next(this.value);

    }

    super.complete();

  }

}
比较
根据以上Observable单播和Subject多播，以及Subject子类BehaviorSubject 、AsyncSubject、ReplaySubject的多播示例，可以对比ObserverA和ObserverB接收的数据流；

对于ObserverA，因为它是同步订阅，因此除了AsyncSubject仅接收最后一个值之外，其他方式订阅的数据流表现一致，即在200ms接收到第一个数据0，每隔200ms以此类推；
对于ObserverB，因为它是异步订阅（500ms后开始订阅），对于不同的订阅方式数据的流动也表现不同，具体的表现可结合上面的示例和以下数据流动图进行对比；
数据流动图如下

无法复制加载中的内容

Operator
采用函数式编程风格的纯函数 (pure function)，使用像 map、filter、concat、flatMap 等这样的操作符来处理集合

基本用法
使用pipe方法，传入operator函数，可以对原始推送值进行一定的转换、拦截等处理；如下示例中，take operator实现获取前几个原始值的功能，map operator实现对原始值进行转换映射的功能；

const observable = Observable.interval(200).pipe(

  take(6),

  map((item) => item * 2)

);



observable.subscribe(value => console.log(value));

// 输出：0 -> 2 -> 4 -> 6 -> 8 -> 10
实现原理
以map为例，map((item) => item * 2) 返回的是一个带source入参的operation function，operation function将调用source.lift 返回一个新的source指向原observable，带operator的observable实例；
通过Observable 的pipe方法传入operation function，pipe方法使用reduce完成多个operation function的链式调用，初始source值是当前Observable，最终pipe返回的一个新的Observable实例；
当创建的带operator的Observable实例被subscribe时，会调用operator.call 方法；
operator.call方法中会对observer的next回调进行封装，返回新的转换值给原observer，最后将新封装的observer传递给source.subscribe;
class Observable {

  source;

  operator;

  ....

  subscribe(observerOrNext, error, complete) {

  ....

    if (this.operator) {

      return this.operator.call(observer, this.source);

    }

    ....

  }

  lift(operator) {

    const observable = new Observable();

    observable.source = this;

    observable.operator = operator;

    return observable;

  }

  pipe(...args) {

    const operations = args.slice(0);

    if (operations.length === 0) {

      return this;

    } else if (operations.length === 1) {

      return operations[0](this);

    } else {

      return operations.reduce((source, func) => func(source), this);

    }

  }

}



// map operator

function map(mapFn) {

  return function mapOperation(source) {

   // 返回带operator的新的observable实例

    return source.lift(new mapOperator(mapFn, thisArg));

  };

}

// operator类

class mapOperator {

  constructor(mapFn) {

    this.mapFn = mapFn；

  }

  // call 方法最终调用的是source observable的 subscribe方法

  // 对传入的observer进行一层封装

  call(observer, source) {

    return source.subscribe(

      new mapObserver(observer, this.mapFn);

    );

  }

}

// 对原始observer进行数据拦截处理

class mapObserver extends Observer {

  constructor(destination, mapFn, thisArg) {

    super();

    this.destination = destination;

    this.mapFn = mapFn;

  }



  next(value) {

    const result = this.mapFn(value);

    this.destination.next(result);

  }

  complete() {

    this.destination.complete();

  }

}
同理可实现filter、take、scan等常用operator;

takeUtil的实现稍有不同，需要传入一个notifyObservable，当notifyObservable首次发出值或complete的时，提示当前订阅结束

示例
// takeUntil示例，当点击了document后，停止每秒数据推送

const notifier = Observable.fromEvent(document, "click");

const observable = Observable.interval(1000).pipe(takeUntil(notifier));
原理
新增notifierObserver类，订阅notifyObservable，当notifyObservable数据到达时，notifierObserver就会通知outerObserver（原observer），这样原来的observer就可以知道notifyObservable的状态；

function takeUntil(notifier) {

  return function takeUntilOperation(source) {

    return source.lift(new takeUntilOperator(notifier));

  };

}



class takeUntilOperator {

  constructor(notifier) {

    this.notifier = notifier;

  }



// notifierObserver订阅notifyObservable

//当notifyObservable推送第一个值时，notifierObserver将调用outerObserver.notifyNext

  call(observer, source) {

    const outerObserver = new takeUntilObserver(observer, this.notifier);

    const notifierObserver = new NotifierObserver(outerObserver);

    this.notifier.subscribe(notifierObserver);

    if (!outerObserver.seenValue) {

      return source.subscribe(outerObserver);

    }

  }

}

class NotifierObserver extends Observer {

  constructor(outerObserver) {

    super();

    this.outerObserver = outerObserver;

  }

  // 接受到值就通知outerObserver

  next(value) {

    this.outerObserver.notifyNext(value);

  }

  error(err) {

    this.outerObserver.notifyError(err);

    this.unsubscribe();

  }

  complete() {

    this.outerObserver.notifyComplete();

    this.unsubscribe();

  }

}

class takeUntilObserver extends Observer {

  constructor(destination) {

    super();

    this.destination = destination;

    this.seenValue = false;

  }

  // 接收到notifyNext的值或notifyComplete时就完成订阅

  notifyNext(value) {

    this.seenValue = true;

    this.destination.complete();

  }

  notifyComplete() {

    this.seenValue = true;

    this.destination.complete();

  }

  next(value) {

    if (!this.seenValue) {

      this.destination.next(value);

    }

  }

```
