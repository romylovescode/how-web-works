- 透彻》 https://juejin.cn/post/7100103691474059272

- 这篇文章来自向谭鹏飞同学的约稿，他结合自己丰富的实践经验，为我们介绍了Angular的生命周期全套知识


在Angular中，实现组件的生命周期hook接口方法是使用Angular框架实现插入业务的时机点。而不同的生命周期hook适合不同的初始化业务，深入了解生命周期钩子并了解其与其他特性的组合可以帮助大家更好的实现业务，少写隐藏性bug。官方章节内容是基于生命周期本身去讲的，而实际使用过程生命周期的执行过程会与依赖注入时机， 数据流传递，变化检测，父子组件数据变更，组件继承等等诸多特性结合使用，在结合后，执行顺序是怎样的对我们来说有点不那么容易弄明白，只有通过大量思考以及实践后才能得知并总结出其规律。希望本文能给大家一些启示。

生命周期hooks的意义
在Angular中，实现组件的生命周期hook接口方法是使用Angular框架实现插入业务的时机点。
官方文档中生命周期都有哪些内容
在Angular的官方文档中有一个章节Lifecycle hooks来专门讲解生命周期的机制与如何来观察组件生命周期。
从官方章节中，我们能了解到生命周期hook方法 的方方面面并且基本不会有出入。
我列举几点从官方收获到的：

在组件中的写法；要实现 OnInit 方法，在组件中添加 ngOnInit 方法, 注意多了俩个字母， 每个hook方法均是如此。
生命周期事件顺序；
如何去观察生命周期事件；
使用ngOnInit作为业务插入点，而不应该在constructor中放入业务代码；
ngOnDestroy的执行时机
全部生命周期事件触发顺序
ngOnChanges的执行机制
...等等

这基本上覆盖到了有组件生命周期的方方面面；
本文重点内容
官方章节内容是基于生命周期本身去讲的，而实际使用过程生命周期的执行过程会与依赖注入时机， 数据流传递，变化检测，父子组件数据变更，组件继承等等诸多特性结合使用，在结合后，执行顺序是怎样的对我们来说有点不那么容易弄明白，只有通过大量思考以及实践后才能得知并总结出其规律。
与官方的切入点不同，本文希望从实践中来总结规律。
而本文选取了如下几种场景，进行实践演示，并尝试得出规律：

基本的生命周期介绍
数据流传递的时机是在哪个具体的事件执行。
父子组件中父子组件的生命周期是如何执行的。
继承组件中生命周期又是如何执行的。
模板中绑定的变量在获取值时与这些生命周期有没有关系？

希望通过本文的阅读，能帮你拨开一些面纱，让你能够随我一起进一步掌握Angular的特性，去思考Angular中生命周期设定去窥探Angular的运行机制；理解Angular的思路、思想以及其为开发者塑造的思考模式。
只有我们能够按照Angular的方式思考，当遇到复杂的业务参与了复杂的数据流，复杂的组件关系，复杂的类关系后，能通过思考快速了解知识盲区，快速组织获取相关知识的关键词，这样茫茫Angular的概念以及新知识的海洋里你就变得如鱼得水，也会成为我们开发业务代码时，优化重构，解决问题时最锋利的矛。
场景一：回顾生命周期钩子的基本用法
由于官方文档中已经把生命周期的顺序介绍并演示了，我们这里就做个简单的验证和总结，当做一次复习。
首先来看一个实际执行效果图

注意：有仔细的小伙伴可能会注意到这个图中，没有看到ngOnChanges事件。不是忘了 根组件不会触发ngOnChanges事件，因为跟组件没有@Input变量；
源码如下
kotlin 代码解读复制代码@Component({  selector: 'my-app',  templateUrl: './app.component.html',  styleUrls: ['./app.component.css'],})export class AppComponent implements OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked,    AfterViewInit, AfterViewChecked, OnDestroy {  name = 'Angular ' + VERSION.major;  messages = [];  subject = window['subject'];  constructor() {    this.subject.subscribe((item) => {      this.messages.push(item);    });    this.subject.next({ type: 'constructor exec', content: 'AppComponent class instance' });  }  ngOnChanges() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngOnChanges' });  }  ngOnInit() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngOnInit' });  }  ngDoCheck() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngDoCheck' });  }  ngAfterContentInit() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterContentInit' });  }  ngAfterContentChecked() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterContentChecked' });  }  ngAfterViewInit() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterViewInit' });  }  ngAfterViewChecked() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterViewChecked' });  }  ngOnDestroy() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngOnDestroy' });  }}

下图是正常组件，并带有@Input属性

源码如下
css 代码解读复制代码@Component({  selector: 'my-app',  templateUrl: './app.component.html',  styleUrls: ['./app.component.css'],})export class AppComponent implements OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked,    AfterViewInit, AfterViewChecked, OnDestroy {  name = 'Angular ' + VERSION.major;  messages = [];  subject = window['subject'];  constructor() {    this.subject.subscribe((item) => {      this.messages.push(item);    });    this.subject.next({ type: 'constructor exec', content: 'AppComponent class instance' });  }}// HelloComponent 是AppComponent的子视图组件...@Component({  selector: 'hello',  template: `<h1>Hi, {{name}}!</h1>`,  styles: [`h1 { font-family: Lato; }`],})export class HelloComponent implements OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked,    AfterViewInit, AfterViewChecked, OnDestroy {      _name: string = '';  subject = window['subject'];  @Input() set name(n: string) {    this.subject.next({ type: '@input', content: 'set name update' });    this._name = n;  }  get name() {    // this.subject.next({ type: 'template binding variable get', content: 'get name update' }); 仅演示调用    return this._name;  }  messages = [];  constructor() {    this.subject.next({ type: 'constructor exec', content: 'class instance, 访问@input属性name=' + this.name });  }  ngOnChanges() {    this.subject.next({ type: 'lifecycle', content: 'ngOnChanges, 访问@input属性name=' + this.name });  }  ngOnInit() {    this.subject.next({ type: 'lifecycle', content: 'ngOnInit, 访问@input属性name=' + this.name });  }  ngDoCheck() {    this.subject.next({ type: 'lifecycle', content: 'ngDoCheck, 访问@input属性name=' + this.name });  }  ngAfterContentInit() {    this.subject.next({ type: 'lifecycle', content: 'ngAfterContentInit, 访问@input属性name=' + this.name });  }  ngAfterContentChecked() {    this.subject.next({ type: 'lifecycle', content: 'ngAfterContentChecked, 访问@input属性name=' + this.name });  }  ngAfterViewInit() {    this.subject.next({ type: 'lifecycle', content: 'ngAfterViewInit, 访问@input属性name=' + this.name });  }  ngAfterViewChecked() {    this.subject.next({ ype: 'lifecycle', content: 'ngAfterViewChecked, 访问@input属性name=' + this.name });  }  ngOnDestroy() {    this.subject.next({ type: 'lifecycle', content: 'ngOnDestroy, 访问@input属性name=' + this.name });  }}

解释并归纳
我们结合上面顺序可以先粗略画一个图如下：

解释下这个图中的每个钩子的含义：
注意：

浅灰色名字的事件，在组件的生命周期中只会触发一次，而绿色的随着相应的逻辑变化会多次触发。
这里我将组件构造函数的执行也加入到了观察序列中，因为在业务中，经常会有小伙伴会在constructor中插入业务代码。

所有的方法执行顺序如下：
 代码解读复制代码Construction, OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked, AfterViewInit, AfterViewChecked, OnDestroy

Construction
构造函数执行时， 执行一次。
OnChanges
当指令（组件的实现继承了指令）的任何一个可绑定属性发生变化时调用。
要点：

从上面根组件的执行效果来看，这个hook不一定会被调用，只有@Input属性变化才会触发；
@Input属性的变更次数是没有要求的，所以它的回调次数也是没有限制的。
这块要特别注意父级改变传递值的情况，会导致ngOnChange在生命周期的任何时刻都可能被再次调用。
参数是一个输入属性的变更处理器，会包含所有变更的输入属性的旧值和新值；
如果至少发生了一次变更，则该回调方法会在默认的变更检测器检查完可绑定属性之后、视图子节点和内容子节点检查完之前调用。
属性的setter能够替代在这个钩子中执行逻辑。

OnInit
在 Angular 初始化完了该指令的所有数据绑定属性之后调用；
要点：

在默认的变更检测器首次检查完该指令的所有数据绑定属性之后，任何子视图或投影内容检查完之前。
它会且只会在指令初始化时调用一次
定义 ngOnInit() 方法可以处理所有附加的初始化任务。

DoCheck
在变更检测期间，默认的变更检测算法会根据引用来比较可绑定属性，以查找差异。 你可以使用此钩子来用其他方式检查和响应变更。
要点：

除了使用默认的变更检查器执行检查之外，还会为指令执行自定义的变更检测函数
默认变更检测器检查更改时，他会触发OnChanges()的执行（如果有），而不在乎你是否进行了额外的变更检测。
不应该同时使用DoCheck和OnChanges来响应在同一个输入上发生的更改。
默认的变更检测器执行之后调用，并进行变更检测。
参见 KeyValueDiffers 和 IterableDiffers，以实现针对集合对象的自定义变更检测逻辑。
在DoCheck中你可以实现监控那些OnChanges无法捕获的变更，检测的逻辑需要自行实现。
由于DoCheck可以监控出特定变量的何时发生了变化，但这却非常昂贵。Angular 在页面的其它地方渲染不相关的数据也会触发这个钩子，所以你的实现必须自行保证用户体验。

AfterContentInit
它会在 Angular 初始化完该指令的所有内容之后立即调用。
要点：

在指令初始化完成之后，它只会调用一次。
可以用来处理一些初始化任务

AfterContentChecked
在默认的变更检测器对该指令下的所有内容完成了变更检测之后立即调用。
AfterViewInit

在 Angular 完全初始化了组件的视图后调用。 定义一个 ngAfterViewInit() 方法来处理一些额外的初始化任务。

AfterViewChecked
在默认的变更检测器对组件视图完成了一轮变更检测周期之后立即调用。
OnDestroy
在指令、管道或服务被销毁时调用。 用于在实例被销毁时，执行一些自定义清理代码。
进一步说明：
因为我们关心的是什么时候使用这些钩子，大家回到上面这些回调钩子定义处，仔细观察带有Init的钩子内容，可以看到OnInit ， AfterContentInit， AfterViewInit 这三个，他们都只执行一次，都可以做一些初始化任务，这三个钩子的区别，就像定义中的描述都是有差别的。
大家有需要可以在文章下面留言，看实际情况是否需要仔细解释下，这三个钩子所适用的差异化场景。
没有仔细研究过这三者不同的小伙伴，常常对于应该把自己要实现的异步初始化业务放到哪个钩子中晕头转向，所以随便选一个，要么放在OnInit, 要么AfterViewInit，如果发现放到一个里不行，就换另一个，直到问题解决或耗费很长时间问题解决不了或者留下偶现的bug(之所以偶现是因为没有从技术上保证异步的顺序性执行)，再次排查也相当费劲。
所以这三个Init非常值得写异步业务比较多的小伙伴关注。
从第一个场景下，我们回顾了每一个生命周期钩子都有哪些内容。
接下来我们看一下带有@Input的场景：
场景二：生命周期钩子与输入属性（@Input）
源码如下
xml 代码解读复制代码//AppComponent html<h1>Hi, Angular 13!</h1><h3>- 演示生命周期钩子函数调用顺序<br /></h3><p>Start editing to see some magic happen :)</p><ul>  <li *ngFor="let message of messages">    <span class="message-type">{{ message.type }}</span>    =>    <span class="message-content">{{ message.content }}</span>  </li></ul><hello [name]="name"></hello>

css 代码解读复制代码// AppComponent@Component({  selector: 'my-app',  templateUrl: './app.component.html',  styleUrls: ['./app.component.css'],})export class AppComponent{  name = 'Angular ' + VERSION.major;  messages = [];  subject = window['subject'];  constructor() {    this.subject.subscribe((item) => { this.messages.push(item); });    this.subject.next({ type: 'constructor exec', content: 'AppComponent class instance, 访问@input属性name=' + this.name });  }}// HelloComponent 是AppComponent的子视图组件...@Component({  selector: 'hello',  template: `<h1>Hi, {{name}}!</h1>`,  styles: [`h1 { font-family: Lato; }`],})export class HelloComponent implements OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked,    AfterViewInit, AfterViewChecked, OnDestroy {      _name: string = '';  subject = window['subject'];  @Input() set name(n: string) {    this.subject.next({ type: '@input', content: 'set name update' });    this._name = n;  }  get name() {    // this.subject.next({ type: 'template binding variable get', content: 'get name update' }); 仅演示调用    return this._name;  }  messages = [];  constructor() {    this.subject.next({ type: 'constructor exec', content: 'class instance, 访问@input属性name=' + this.name });  }  ngOnChanges() {    this.subject.next({ type: 'lifecycle', content: 'ngOnChanges, 访问@input属性name=' + this.name });  }  ngOnInit() {    this.subject.next({ type: 'lifecycle', content: 'ngOnInit, 访问@input属性name=' + this.name });  }  ngDoCheck() {    this.subject.next({ type: 'lifecycle', content: 'ngDoCheck, 访问@input属性name=' + this.name });  }  ngAfterContentInit() {    this.subject.next({ type: 'lifecycle', content: 'ngAfterContentInit, 访问@input属性name=' + this.name });  }  ngAfterContentChecked() {    this.subject.next({ type: 'lifecycle', content: 'ngAfterContentChecked, 访问@input属性name=' + this.name });  }  ngAfterViewInit() {    this.subject.next({ type: 'lifecycle', content: 'ngAfterViewInit, 访问@input属性name=' + this.name });  }  ngAfterViewChecked() {    this.subject.next({ ype: 'lifecycle', content: 'ngAfterViewChecked, 访问@input属性name=' + this.name });  }  ngOnDestroy() {    this.subject.next({ type: 'lifecycle', content: 'ngOnDestroy, 访问@input属性name=' + this.name });  }}

执行结果

解释并归纳
我们给Hello组件添加了输入属性，父组件AppComponent初始化时给name赋值了初始值，仅在angular的处理下输入属性的绑定是发生在Hello组件初始化之前（当然在Hello组件生命周期调用的过程中，父组件随时可能改变name）。

注意在第一次OnChanges触发之后，也就是传递变量的初始值给完后，有些情况我们会通过逻辑在父组件中调整传递变量的值，这时就会立即再次触发OnChanges的回调，并且这个回调与HelloComponent组件的OnInit，AfterContentInit等的回调是按时间顺序依次调用的。也就是OnChanges的触发与AfterContentInit， AfterViewInit是否已经完成一次执行无关。

我们在刚刚的AppComponent组件中也加入生命周期的执行，结果会怎么样呢？
案例三：生命周期钩子与父子视图带输入属性
改动源码如下
kotlin 代码解读复制代码export class AppComponent implements OnChanges, OnInit, DoCheck, AfterContentInit, AfterContentChecked,    AfterViewInit, AfterViewChecked, OnDestroy {  name = 'Angular ' + VERSION.major;  messages = [];  subject = window['subject'];  constructor() {    this.subject.subscribe((item) => { this.messages.push(item); });    this.subject.next({ type: 'constructor exec', content: 'AppComponent class instance, 访问@input属性name=' + this.name });  }  ngOnChanges() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngOnChanges' });  }  ngOnInit() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngOnInit' });  }  ngDoCheck() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngDoCheck' });  }  ngAfterContentInit() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterContentInit' });  }  ngAfterContentChecked() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterContentChecked' });  }  ngAfterViewInit() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterViewInit' });  }  ngAfterViewChecked() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngAfterViewChecked' });  }  ngOnDestroy() {    this.subject.next({ type: 'lifecycle', content: 'AppComponent ngOnDestroy' });  }}

执行结果

解释并归纳
现象：

父组件AppComponent的生命周期回调被子组件分为了俩部分。
constuctor的执行顺序是先父组件，接下来是子组件，接下来才是生命周期的其他函数回调。
父组件的ngOnChanges（如果有，第一次）, ngOnInit，ngDoCheck，ngAfterContentInit，ngAfterContentChecked会执行较早。
父组件name值传递到子组件，触发子组件的OnChanges。
子组件的生命周期执行，接下来父组件的ngAfterViewInit， ngAfterViewChecked执行。

要点：


父组件将绑定值传入到子组件是在父组件的生命周期执行到ngAfterContentChecked时触发，这一点很重要

意味着，如果在子组件的生命周期（比如：OnInit）中有处理依赖传递变量的逻辑，那么可能得不到最新的传递值。（由于这一点，小伙伴经常陷入困惑中，这也与不了解Init钩子的适用场景有关）



父子组件中，AfterViewInit会等到所有的子组件的生命周期执行完成才执行，（这一点特性应该被充分发挥并利用）。


接下来我们看看存在继承组件的场景下，Angular会怎么处理生命周期回调。

作者：OpenTiny社区
链接：https://juejin.cn/post/7100103691474059272
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
