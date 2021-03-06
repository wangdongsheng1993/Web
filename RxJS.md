# RxJS （Reactive Extensions for JavaScript）

#### 用来做什么？
处理异步回调
#### 和Promise的区别？
RxJS可以取消订阅，Promise不可以<br/>
RxJS可以发射多次，Promise只有一次就freeze<br/>
RxJS集成Promise,回调，事件数据 例如DOM Input，Web Worker, Web Socket<br/>
#### 特点
`Observables + Operators + Schedulers` （可观察对象+运算符+调度器）
* 用Observable表示异步数据流
* 用Operator查询异步数据流
* 用Scheduler使异步数据流的并发性参数化
#### 兼容
支持IE6+、Chrome4+、Firefox1+
支持Node.jsV0.4+


## 基本概念
#### Observable (可观察对象)
Observables是多个值的延迟推送集合<br/>
Pull是由消费者决定何时获取数据<br/>
Push是由生产者决定何时发送数据<br/>
Promise和RxJS引入的Observable都是JavaScript的Push系统<br/>
| | SINGLE |	MULTIPLE |
-|-|-
| Pull |	Function |	Iterator |
| Push |	Promise |	Observable |
    
#### Observer (观察者)
Observer是Observable传递给消费者的值<br/>
Observers是一组回调，是具有next、error和complete三种回调的对象<br/>
Observable可能传递的三种类型中任一种通知<br/>
` observable.subscribe(observer); `
```
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```
订阅Observable时，也可以只提供一个回调作为参数，而不附加到Observer对象，如此，会将回调参数用于next的handler从而创建一个observer
`observable.subscribe(x => console.log('Observer got a next value: ' + x));`

#### Operators(操作符)  [Reference](https://rxjs-dev.firebaseapp.com/api/operators)
Operators是functions，它分为两种类型：
1. **Pipeable Operators**是一个纯函数(pure function)，它将一个Observable作为输入，使用语法`ObservaleInstance.pipe(operator())`并返回另一个Observable。这是一个纯粹的操作：先前的可观测数据保持不变。
2. **Creation Operators**作为一个独立的方法被调用来创建一个新的Observable。用于创建具有一些常见预定义行为的Observable，或通过连接其他Observables来创建一个Observable。 [Creation Operators List](https://rxjs-dev.firebaseapp.com/guide/operators#creation-operators-list)

Observables一般情况是一些常见类型的值，但也有可能是Observable类型，这就需要处理Observable的Observable，即high-order Observables(高阶Observables)

[join operators 连接操作符](rxjs-dev.firebaseapp.com/guide/operators#join-operators)可以用来处理高阶Observables
* [concatAll()操作符](https://rxjs-dev.firebaseapp.com/api/operators/concatAll)，订阅从外部Observable中产生的每个内部Observable，并复制所有发出的值，直到该Observable完成，然后继续下一个。所有值都以这种方式连接在一起。
* [mergeAll()操作符](https://rxjs-dev.firebaseapp.com/api/operators/mergeAll)，在每个内部Observable到达时订阅，然后在其到达时发出每个值
* [switchAll()操作符](https://rxjs-dev.firebaseapp.com/api/operators/switchAll)，当第一个内部Observable到达时订阅，并在其到达时发出每个值，但当下一个内部Observable到达时，取消订阅上一个，并订阅新的。
* [exhaustAll()操作符](https://rxjs-dev.firebaseapp.com/api/operators/exhaustAll)，当第一个内部Observable到达时订阅，并在其到达时发出每个值，丢弃所有新到达的内部Observable，直到第一个内部Observable完成，然后等待下一个内部Observable。

#### Subscription (订阅)
Subscription是一个表示一次性的Observable的执行，主要用于取消Observable的执行。<br/>
Subscription本质上只有一个unsubscribe()函数来释放资源或取消Observable的执行。`subscription.unsubscribe();`<br/>
Subscription也可以放在一起，可以通过将一个Subscription添加到另一个Subscription中来实现`subscription.add(childSubscription);`，因此对一个Subscription的unsubscribe()的调用可以取消多个订阅。<br/>
Subscription还有一个`subscription.remove(childSubscription);`方法，用于撤消子订阅的添加。

#### Subject (主体)
Subject是一种特殊类型的Observable，允许将值多播给许多Observers。普通的Observable是单播的（每个订阅的Observer拥有一个独立的Observable执行，但Subject是多播的，并且是将值或事件多路推送给多个Observer的唯一方式。<br/>
相当于EventEmitter，维护一个包含许多linsteners的registry。<br/>
**Every Subject is an Observable.** 给定一个Subject，可以订阅它，提供一个Observer，它将开始正常接收值。从Observer的角度来看，它无法判断可观察的执行是来自普通的单播Observable还是来自Subject。<br/>
subscribe不会在Subject内部调用新执行来传递值。它只是在Observers列表中注册给定的Observer，类似于addListener在其他库和语言中的工作方式。<br/>
```
import { Subject } from 'rxjs';
 
const subject = new Subject<number>();
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(1);
subject.next(2);
 
// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
```

**Every Subject is an Observer.** Subject具有next(v)、error(e)和complete()方法。要向Subject提供一个新值，只需调用next(theValue)，它将被多播给注册收听Subject的Observer。<br/>
由于Subject是观察者，这也意味着您可以提供一个Subject作为任何Observable的订阅参数
```
import { Subject, from } from 'rxjs';
 
const subject = new Subject<number>();
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
const observable = from([1, 2, 3]);
 
observable.subscribe(subject); // You can subscribe providing a Subject
 
// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```
Subject是使任何observable的执行共享给多个observer的唯一方法。<br/>

**Multicasted Observables** 使用Subject能够使多个observer看到相同的observable执行。observer订阅底层Subject，Subject订阅源observable。
```
import { from, Subject } from 'rxjs';
import { multicast } from 'rxjs/operators';
 
const source = from([1, 2, 3]);
const subject = new Subject();
const multicasted = source.pipe(multicast(subject));
 
// These are, under the hood, `subject.subscribe({...})`:
multicasted.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
multicasted.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
// This is, under the hood, `source.subscribe(subject)`:
multicasted.connect();
```
multicast返回的是一个ConnectableObservable，它只是有着connect()方法的一个observable。<br/>
connect()方法对于确定何时共享observable执行开始非常重要。因为connect()在后台执行`source.subscribe(subject)`，所以connect()返回一个Subscription，可以从中取消订阅以取消共享的observable执行。

* **BehaviorSubject**存储发送给消费者的最新值，并且每当新的Observer subscribe时，它将立即从BehaviorSubject接收当前值
```
import { BehaviorSubject } from 'rxjs';
const subject = new BehaviorSubject(0); // 0 is the initial value
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(3);
 
// Logs
// observerA: 0
// observerA: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```
* **ReplaySubject**记录observable执行的多个值(可配置)，并将它们重放到新subscribers中
```
import { ReplaySubject } from 'rxjs';
const subject = new ReplaySubject(3); // buffer 3 values for new subscribers
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(5);
 
// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
// observerA: 5
// observerB: 5
```
除了缓冲区大小之外，还可以指定以毫秒为单位的窗口时间，以确定记录的值的时间长度。在下面的示例中，使用了100的缓冲区大小，窗口时间参数为500毫秒：
```
import { ReplaySubject } from 'rxjs';
const subject = new ReplaySubject(100, 500 /* windowTime */);
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
let i = 1;
setInterval(() => subject.next(i++), 200);
 
setTimeout(() => {
  subject.subscribe({
    next: (v) => console.log(`observerB: ${v}`)
  });
}, 1000);
 
// Logs
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerA: 5
// observerB: 3
// observerB: 4
// observerB: 5
// observerA: 6
// observerB: 6
// ...
```
* **AsyncSubject** 在执行完成时,只会有observable执行的最后一个值被发送给其observer。类似于last()operator。
```
import { AsyncSubject } from 'rxjs';
const subject = new AsyncSubject();
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(5);
subject.complete();
 
// Logs:
// observerA: 5
// observerB: 5
```
* **Void subject**
`const subject = new Subject<void>();`<br/>
`const subject = new Subject(); // Shorthand for Subject<void>`<br/>
有时发送的值是不必要的。在版本7之前，subject值的默认类型是any。subject<any>不会对emit值的类型进行检查，而subject<void>防止意外访问emit值。

#### Schedulers (调度器)
Schedulers控制订阅何时启动以及何时发送通知，用来控制并发并且是中央集权的调度员。<br/>
**Schedulers是一种数据结构**它知道如何根据优先级或其他标准存储和排队任务。<br/>
**Schedulers是一个执行上下文**它表示任务在何时何地执行（例如立即执行，或在另一种回调机制中执行，如setTimeout或process.nextTick，或动画帧），允许定义observable将在什么执行上下文中向其observer发送通知。<br/>
**Schedulers有一个（虚拟）时钟**它通过调度器上的getter方法now（）提供时间的概念。在特定scheduler上计划的任务将仅遵循该时钟指示的时间。<br/>
Schedulers。<br/>

**Scheduler类型**
| SCHEDULER | PURPOSE |
-|-
| null | 不传递任何调度器的话，会以同步递归的方式发送通知。用于定时操作（constant-time operations）或尾递归操作（tail recursive operations） |
| queueScheduler | 当前事件帧中的队列调度 (蹦床调度器trampoline scheduler). 用于迭代操作（iteration operations）. |
| asapScheduler | 微任务的队列调度，它使用可用的最快速的传输机制，比如 Node.js 的 process.nextTick() 或 Web Worker 的 MessageChannel 或 setTimeout 或其他。该队列与用于promises的队列相同，基本上在当前job之后，但在下一个job之前。用于异步转换（asynchronous conversions）。 |
| asyncScheduler |	使用 setInterval 的调度，用于基于时间的操作符（time-based operations）。 |
| animationFrameScheduler | scheduler任务将在下一个浏览器内容重新绘制之前发生。用于创建平滑的浏览器动画。|
 
如果不提供Scheduler，RxJS将选择默认Scheduler，使用最小并发原则。但如果出于性能考虑，你想要引入并发，那么可以选择不同的调度器。
* 对于返回具有有限和少量消息的observable的operators，RxJS不使用scheduler，即null或未定义。
* 对于返回可能大量或无限多条消息的operators，使用queue Scheduler。
* 对于使用计时器timer的operators，使用async Scheduler。
要指定具体的调度器，可以使用那些采用调度器的操作符方法。
* **静态创建操作符通常可以接收调度器作为参数**，比如`from(array, scheduler)`可以让你指定调度器，当从array转换成要发送的每个通知时使用。
    * bindCallback
    * bindNodeCallback
    * combineLatest
    * concat
    * empty
    * from
    * fromPromise
    * interval
    * merge
    * of
    * range
    * throw
    * timer
* **使用 subscribeOn 来调度 subscribe() 调用在什么样的上下文中执行**，默认情况下，Observable 的 subscribe() 调用会立即同步地执行。然而，你可能会延迟或安排在给定的调度器上执行实际的 subscription，使用实例操作符（instance operator）`subscribeOn(scheduler)`，其中scheduler是你提供的参数。
* **使用 observeOn 来调度发送通知的的上下文**，`observeOn(scheduler)`在源Observer和目标Observer之间引入了一个中介Observer，中介Observer使用给定的调度器调度对目Observer进行调用。
* **实例操作符（instance operator）可能会接收调度器作为参数**，像bufferTime、debounceTime、delay、auditTime、sampleTime、throttleTime、timeInterval、timeout、timeoutWith、windowTime 这样时间相关的操作符全部接收调度器作为最后的参数，并且默认的操作是在asyncScheduler调度器上。<br/>
其他接收调度器作为参数的实例操作符还有：cache、combineLatest、concat、expand、merge、publishReplay、startWith。<br/>
注意，cache和publishReplay接收调度器都是因为它们使用了ReplaySubject。ReplaySubjects的构造函数接收一个可选的调度器作为最后的参数，因为ReplaySubject可能会处理时间，这只在调度器的上下文中才有意义。默认情况下，ReplaySubject使用queueScheduler来提供时钟。
```
import { Observable, asyncScheduler } from 'rxjs';
import { observeOn } from 'rxjs/operators';
 
const observable = new Observable((observer) => {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
}).pipe(
  observeOn(asyncScheduler)
);
 
console.log('just before subscribe');
observable.subscribe({
  next(x) {
    console.log('got value ' + x)
  },
  error(err) {
    console.error('something wrong occurred: ' + err);
  },
  complete() {
     console.log('done');
  }
});
console.log('just after subscribe');
    
// Logs:
// just before subscribe
// just after subscribe
// got value 1
// got value 2
// got value 3
// done
```
`observeOn(asyncScheduler)`在`new Observable`和最终的Observer之间其实是引入了一个代理Observer，为了更好的理解请参考下例。
```
import { Observable, asyncScheduler } from 'rxjs';
import { observeOn } from 'rxjs/operators';

var observable = new Observable((proxyObserver) => {
  proxyObserver.next(1);
  proxyObserver.next(2);
  proxyObserver.next(3);
  proxyObserver.complete();
}).pipe(
  observeOn(asyncScheduler)
);
 
var finalObserver = {
  next(x) {
    console.log('got value ' + x)
  },
  error(err) {
    console.error('something wrong occurred: ' + err);
  },
  complete() {
     console.log('done');
  }
};
 
console.log('just before subscribe');
observable.subscribe(finalObserver);
console.log('just after subscribe');
```
proxyObserver是在`observeOn(asyncScheduler)`中创建的，它的`next(val)`函数大概是下面这样子的：
```
const proxyObserver = {
  next(val) {
    asyncScheduler.schedule(
      (x) => finalObserver.next(x),
      0 /* 延迟时间 */,
      val /* 会作为上面函数所使用的 x */
    );
  },

  // ...
}
```
asyncScheduler操作符使用了setTimeout或setInterval，即使给定的延迟时间为0。因为在JavaScript中，我们知道setTimeout(fn, 0)会在下一次事件循环迭代的最开始运行fn。这也解释了为什么发送给 finalObserver的 got value 1 会发生在 just after subscribe 之后。
Scheduler的schedule()方法接收一个delay参数，它指的是相对于Scheduler内部时钟的一段时间。Scheduler的时钟不需要与实际的挂钟时间有任何关系。这也就是为什么像delay这样的时间操作符不是在实际时间上操作的，而是取决于调度器的时钟时间。
  

## 测试
#### 大理石图测试RxJS
大理石图为我们提供了一种直观的方式来表示Observable的行为。我们可以使用它们来断言一个特定的Observable的行为也可以mock [hot and cold Observables](https://benlesh.medium.com/hot-vs-cold-observables-f8094ed53339)来检查是否符合预期。[Reference](https://rxjs-dev.firebaseapp.com/guide/testing/marble-testing)



