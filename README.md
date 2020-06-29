```
// 首先我们先创建一个类Fn
class Fn {
  // 添加一个属性，和构造方法
  public val: any = "";
  constructor(val: any) {
    this.val = val;
  }

  // 我们添加一个方法
  use(something: any) {
    return new Fn(something);
  }
}

// 我们要调用use得这么做
let FnObj = new Fn("first demo");
FnObj.use("the first").use("the second");

// 这就构成了一个最基本的链式表达式

/**
 * 我们再看看函数式编程的一些特点：（以下摘自百度百科）
 * 1. λ演算：通过回调函数作为入参或者出参来计算
 * 2. 闭包和高阶函数：这里指的是编程语言需要具备这两个特性，
 *  其一：需要支持闭包，也就是说在一个函数中可以嵌套子函数，面向对象语言也属于这种特性，我们可以把class类看作特殊的函数。
 *  其二：高阶函数指的就是λ演算，有很多编程语言其实不支持回调函数，也就不能用作函数式编程。
 * 3. 惰性计算：也属于λ演算，利用回调函数来处理一些逻辑，所以在调用函数时是不会马上执行的，它在等回调函数的执行结果。
 * 4. 递归：最理想的函数式编程是自身返回自身对象，从逻辑上这就，是自身调用自身，也就是递归调用，这一点从上面的例子就能看得出来。
 * 5. 函数是”第一等公民“，只用"表达式"，不用"语句"：
 *  这里涉及到一个概念”纯函数“，也就是只有表达式没有变量的函数。
 *  举个例子：
 *    if(true) let a = 'first'; 这是一个表达式，我们可以下面这种方式编程编程表达式。
 *    a = true?'first':''; 这样就不需要表达式了，我们也可以直接返回后面的内容。
 *  下面我再举个更复杂点的例子：
 */
let PureFunc = {
  height: 13,
  set: function(height: any) {
    return (PureFunc.height = height), PureFunc;
  }
};
// 上面我没用到任何的变量，
console.log(PureFunc.set(15).height);
/**
 *  在eslint中，表达式直间的逗号可能不被允许，但在原生js中，却可以用这种方式把多条语句构建成一个表达式。
 *  这就解释了函数是”第一等公民“的观点。
 * 6. 没有副作用，不修改状态，引用透明性：
 *  因为没有变量，函数内也不会用到外部变量，就不会对外部产生任何影响（状态会用到变量）。
 */

// 接下来我们把最上面的例子改造一下
class Fn2 {
  public val: any = "";
  constructor(val: any) {
    this.val = val;
  }

  // 这里是一个自身实例化的过程
  static me = new Fn2("");
  // 我们再添加一个函数
  set(_val: any) {
    //这也是一种逻辑转表达式的方式
    return ((this.val = _val) && false) || this;
  }
  show() {
    return ((console.log(this.val) as any) && false) || this;
  }
  // 接下来我要把函数变得更自由些，加入回调
  transform(callback: Function) {
    return ((this.val = callback(this.val)) && false) || this;
  }
}

// 然后我们就可以这么使用
Fn2.me.set("123").show(); // 控制台打印出：123
// 也可以这么调用
Fn2.me
  .set("123")
  .show()
  .transform((oldVal: string) => {
    return isNaN(parseInt(oldVal)) ? "string" : "number";
  })
  .show(); // 控制台先打印出：123 ， 然后打印出：number

/**
 * 假如，我们把Fn2的transform中的callback函数的入参改成回调函数。
 * 我们就可以这么使用：
 *  Fn2.me.set("123").transform(resolve=>{resolve()})
 * 可能有些抽象，通过下面的例子，你就能明白了。
 */

// 我们再创建一个Fn3
class Fn3 {
  public val: any = "";
  constructor(val: any) {
    this.val = val;
  }

  // 这里是一个自身实例化的过程
  static me = new Fn3("");
  // 我们再添加一个函数
  set(_val: any) {
    //这也是一种逻辑转表达式的方式
    return ((this.val = _val) && false) || this;
  }
  show() {
    return ((console.log(this.val) as any) && false) || this;
  }
  // 接下来我要把函数变得更自由些，加入回调
  transform(callback: Function) {
    return ((this.val = callback(this.val)) && false) || this;
  }
  // 这是一个异步回调
  asynTransform(callback: Function) {
    return (
      ((this.val = callback({ show: this.show, set: this.set })) && false) ||
      this
    );
  }
}

// 我们再来调用一下Fn3
Fn3.me
  .asynTransform(({ show, set }: any) => {
    setTimeout(() => {
      set("one second latter");
      show();
    }, 1000);
    return "first content";
  })
  .show();
/**
 * 结果先输出： first content
 * 一秒钟以后输出： one second latter
 */

/**
 * 这样来看，我们可操作行的空间就大多了，这也满足了上面的惰性计算的要求。
 * 其实这只是一个简单的例子，在我们周围函数式编程无处不在，下面的例子大家肯定看过。
 */
let arr: Array<number> = [49, 52, 78, 11, 23, 90, 12];
let _arr: Array<any> = arr
  .sort((a: number, b: number) => a - b)
  .filter((v: number) => v > 50)
  .map((v: number) => ({ v: v }));
console.log(_arr);
// 结果： [{v:52},{v:78},{v:90}]
/**
 * 我们先定义一个数组，然后排序，再过滤，再重组，这三个操作是连起来的，好比自然语言。
 * 这也是函数式编程的方便之处，而函数式编程最常见的应用上面这种”链式调用“。
 */
```
