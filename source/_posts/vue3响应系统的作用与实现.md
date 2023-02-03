---
title: vue3响应系统的作用与实现
date: 2023-02-01 14:10:39
tags: JavaScript
description: 响应式是vue的一个重要组成部分，它实现了数据与视图的绑定......
---

响应式是vue的一个重要组成部分，它实现了数据与视图的绑定。接下我们就从相应数据跟副作用函数开始一步步了解vue的响应系统的设计与实现
## 副作用函数
副作用函数是指会产生副作用的函数
````
function effect(){
     document.body.innerText = "hellow";
}
````
当effect执行时，会改变body的内容，但除了effect，其他函数也可以访问到body的文本内容，所以effect函数会影响到其他函数读取body的内容，即说effect是一个副作用函数。
````
let a = 1
function effect(){
    var a = 2
}
````
副作用函数非常常见，像这种effect改变了其他函数也可以访问到的全局变量，影响了其他函数对它的读取，这也是一种副作用函数。可以理解为**某个函数改变了其他函数对某个内容的访问结果，即这个函数是副作用函数**

## 响应式数据
````
const obj = {text:'hellow'}
function effect (){
     document.body.innerText = obj.text;
}
````
当effect函数执行时，会改变body的文本内容。如果obj的值变化的时候都能触发effect，从而改变body的文本内容达到改变视图的效果，那我们则称obj是一个响应数据。
改变obj的时候，我们会触发到它的**set**操作，而读取的时候会触发到**get**操作。

如果我们可以拦截这两步操作，在读取obj.text的时候把副作用函数effect存储到一个桶里，在修改obj.text的时候再把这个副作用函数拿出来执行，便可以达到每当修改obj.text的时候都能改变视图的效果了。

在vue2中，vue采用了[Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)实现了响应系统，但**Object.defineProperty只能监听指定key的value的变化，所以需要遍历整个对象从而监听所有key**。vue3使用了[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)代替了defineProperty解决了这一痛点。
这里我们用proxy来实现
````
  //用于存储副作用函数的桶
  const bucket = new Set();
  //原始数据
  const data = { text: "hellow" };
  //对原始数据的代理
  const obj = new Proxy(data, {
    get(target, key) {
      //将副作用函数丢进bucket
      if (effect) {
        bucket.add(effect);
      }
      //返回读取的值
      return target[key];
    },
    set(target, key, newVal) {
      //设置新属性值
      target[key] = newVal;
      //把副作用函数从桶取出并执行
      bucket.forEach((fn) => fn());
      //设置成功
      return true;
    },
  });
````
这时候执行effect之后，再改变obj.text的值的时候，会发数据响应视图，视图改变了。成功实现了一个响应数据。
## 注册副作用函数 
在上面代码中，我们使用effect作为副作用函数，但实际中副作用函数不一定这么命名，甚至可能是一个匿名函数，所以我们需要一个注册副作用函数的流程
````
  //用一个全局变量存储被注册的副作用函数
  let activeEffect;
  //用于注册副作用函数
  function effect(fn) {
    activeEffect = fn;
    fn();
  }
  
  const obj = new Proxy(data, {
    get(target, key) {
      //将副作用函数丢进bucket
      if (activeEffect) {
        bucket.add(activeEffect);
      }
      //返回读取的值
      return target[key];
    },
    set(target, key, newVal) {
      //设置新属性值
      target[key] = newVal;
      //把副作用函数从桶取出并执行
      bucket.forEach((fn) => fn());
      //设置成功
      return true;
    },
  });
  
  effect(() => {
    document.body.innerText = obj.text;
  });

  
````
 这里我们定义了一个activeEffect，它的作用是用来存储被注册的副作用函数，再将effect修改为一个用来注册副作用函数的工具，这样一来无论副作用函数是什么都可以通过注册副作用函数注册后由activeEffect来表示。
当执行effect时，会把匿名函数赋值给activeEffect，接着执行传入的fn函数，会触发obj.text的get，从而将副作用函数放到桶中.
## 完善存放副作用函数的桶的数据结构
以上步骤一直围绕着obj.text做操作，我们的副作用函数是与之绑定的，**如果我们设置了一个原先不存在的属性的话，由于它与副作用函数是没联系的，所以理论上不应该触发这个副作用函数，但实际上当我们对obj设置一个新属性的时候它触发了这个与它没联系的副作用函数**。
````
  effect(() => {
    console.log('effect')
    document.body.innerText = obj.text;
  });
  
  obj.a = 1
````
触发obj.a = 1的时候可以看到副作用函数被触发了，这是不合理的。导致该问题的原因就是我们没有在副作用函数中明确副作用函数与被操作的字段建立联系。这会导致无论当前读取的是哪个属性，都会把副作用函数丢到桶里，设置的时候也是如此，无论是设置哪个属性都会把桶里的副作用函数拿出来执行。所以对这个桶的结构我们需要重新设计下。先看下执行副作用函数的时候其内部的关系。
````
function effect(){
    document.body.innerText = obj.text
}
````
在这段代码中有三个角色
- 代理对象obj
- 字段名text
- 副作用函数effect

把target看成一个代理对象，key是被操作的字段，effectfn是副作用函数，那么它们的关系如下

![请添加图片描述](https://img-blog.csdnimg.cn/6444860aedc048829280a5353b78b14a.jpeg)


如果是有两个副作用函数作用于一个字段
````
  effect(function effect1() {
    obj.text;
  });

  effect(function effect2() {
    obj.text;
  });
````
那么关系如下

![请添加图片描述](https://img-blog.csdnimg.cn/2fa829f9a8ac47578f809fa9de9583f2.jpeg)


如果一个副作用函数中读取了多个属性

````
effect(function effect(){
    obj.text1
    obj.text2
})
````

![请添加图片描述](https://img-blog.csdnimg.cn/73a73d0dfb584e3e847d8217fc0dc7c6.jpeg)


如果在不同副作用函数读取不同属性的话
````
  effect(function effect1() {
    obj.text1;
  });

  effect(function effect2() {
    obj.text2;
  });
````

![请添加图片描述](https://img-blog.csdnimg.cn/b1da665fd27f4044b85145d552623a19.jpeg)


按这个关系来看，便可以解决了我们先前改变任何属性都会触发副作用函数的问题。例如这里我们读取或者设置了text1，只会触发effect1，而不会导致effect2执行。接着就来实现这个桶。

首先我们使用weakMap代替先前的set作为桶的数据结构。因为需要一个key与effectFn的映射关系。
这里说一下WeakMap跟Map的区别
````
  const map = new Map();
  const weakmap = new WeakMap();
  (function () {
    const foo = { foo: 1 };
    const bar = { bar: 1 };
    map.set(foo, 1);
    weakmap.set(bar, 2);
  })();
````
我们创建了map，weakmap两个实例，在立即执行函数内部定义了foo，bar两个对象并且将这两个对象作为map，weakmap的key。函数执行结束后foo还作为key被map引用，因此它没有被垃圾回收器移除，还可以通过map.keys打印出来。而weakmap是弱引用，它不影响垃圾回收器的运作，所以执行完后bar会被垃圾回收器回收，我们也无法通过weakmap获取到bar。
所以使用weakmap常用于存储那些只有当key引用对象存在时才有价值的信息，如果使用map的话，绑定的数据不再被引用了，也不会被回收，最终可能会造成**内存泄漏**。

````
  const bucket = new WeakMap();
  const obj = new Proxy(data, {
    get(target, key) {
      track(target, key);
      return target[key];
    },
    set(target, key, newVal) {
      target[key] = newVal;
      trigger(target, key);
    },
  });

 function track(target, key) {
    //如果没有副作用函数直接return
    if (!activeEffect) {
      return;
    }
    //根据target从桶中取出desMap
    let depsMap = bucket.get(target);
    //如果desMap不存在那么创建一个Map并与target关联
    if (!depsMap) {
      depsMap = new Map();
      bucket.set(target, depsMap);
    }
    //再根据key从desMap中取得deps，它里面存储着所有与当前key关联的副作用函数 effects
    let deps = depsMap.get(key);
    //如果deps不存在，同样创建一个Set并与key关联
    if (!deps) {
      deps = new Set();
      depsMap.set(key, deps);
    }
    //最后将当前激活的副作用函数添加到桶里
    deps.add(activeEffect);
  }

  function trigger(target, key) {
    //根据target从桶中取出depsMap
    const depsMap = bucket.get(target);
    if (!depsMap) {
      return;
    }
    //根据key取得所有副作用函数effects
    const effects = depsMap.get(key);
    //执行副作用函数
    effects && effects.forEach((fn) => fn());
  }
````

这段代码可以看出它的桶设计如下

![请添加图片描述](https://img-blog.csdnimg.cn/186661b1f6164154801f6386257e0a4e.jpeg)

## 分支切换跟cleanup
````
  effect(function effectFn(){
    document.body.innerText = obj.ok ? obj.text : "not";
  });
````
如果我们注册了这么一个副作用函数，当obj.ok为true时，会读取obj.text的值，所以这时候副作用函数会触发ok，text两个属性的读取操作，effectFn会与text,ok建立联系
![请添加图片描述](https://img-blog.csdnimg.cn/6b41a7de060d4a82ae4d9b260cab39ff.jpeg)

但当ok的值为false的时候，不会去读取obj.text，所以这个时候effectFn不应该被text的set收集，但在我们刚刚实现的响应系统里面，obj.ok变成false之后，effectFn还是在text的set里面。
![请添加图片描述](https://img-blog.csdnimg.cn/fb7aa0c63e6c49949f5e2014c0abf669.jpeg)

这种遗留的副作用函数会导致没必要的更新，obj.ok为false，无论text再怎么修改界面都是显示not，但text修改的时候却会触发effecnFn，这是多余的。为了解决这个问题，我们尝试副作用每次执行的时候把相关联的副作用函数从集合中删除。然后当副作用函数执行的时候，会重新建立联系，但新的联系中不会再包含遗留的没必要的集合。
要将一个副作用函数从所有与之有关联的集合中移除，就需要明确哪些集合中有它，因此我们需要重新设计下副作用函数注册器。
````
  function effect(fn) {
    const effectFn = () => {
        //当effectFn执行时，将其设置为当前激活的副作用函数
      activeEffect = effectFn;
      fn();
    };
    //activeEffect.deps用来存储所有与该副作用函数有关联的集合
    effectFn.deps = [];
    effectFn();
  }
````
然后再在track函数中，将集合push到effectFn.deps
````
  function track(target, key) {
    if (!activeEffect) {
      return;
    }
    let depsMap = bucket.get(target);
    if (!depsMap) {
      depsMap = new Map();
      bucket.set(target, depsMap);
    }
    let deps = depsMap.get(key);
    if (!deps) {
      deps = new Set();
      depsMap.set(key, deps);
    }
    deps.add(activeEffect);
    //deps就是一个与当前副作用函数关联的集合，把它push到activeEffect.deps中
    activeEffect.deps.push(deps);
  }

````
有了副作用函数的集合对象，我们就可以在触发副作用函数的时候把该副作用函数从集合中移除了。下面通过实现一个cleanup函数来处理。
````
  function cleanup(effectFn) {
    for (let i = 0; i < effectFn.deps.length; i++) {
      //将effectFn从集合中删除
      const deps = effectFn.deps[i];
      deps.delete(effectFn);
    }
    effectFn.deps.length = 0;
  }
  function effect(fn) {
    const effectFn = () => {
      cleanup(effectFn);
      activeEffect = effectFn;
      fn();
    };
    effectFn.deps = [];
    effectFn();
  }
````
这个时候运行代码会发现，副作用函数无限循环执行，这是因为trigger中我们遍历effects集合时，会调用cleanup清除，实际就是从effects中将当前正在调用的副作用函数清除，但副作用函数执行会让导致又被重新收集到集合中。
解决的方法也很简单，构造另一个Set集合遍历它即可。
````
  function trigger(target, key) {
    const depsMap = bucket.get(target);
    if (!depsMap) {
      return;
    }
    const effects = depsMap.get(key);
    //new一个新的集合
    const effectsToRun = new Set(effects);
    effectsToRun && effectsToRun.forEach((fn) => fn());
  }
````
再执行就没有上面循环的问题了。
## 嵌套的effect与effect栈
````
effect(function effect1(){
   function effect2(){
       ...
   }
})
````
这段代码中effect1嵌套了effect2，effect1，effect1的执行会导致effect2的执行。在vue中这种现象是很常见的，例如组件嵌套。
````
const Bar = ()=>{    
    render(){}
}
const Foo = ()=>{
  render(){
      return <Bar/>
  }
}
````
此时就发生了嵌套，相当于
````
effect(()=>{
    Foo.render()
    effect(()=>{
        Bar.render()
    })
})
````
我们可以通过以下代码测试我们至今完成的响应系统是不是支持嵌套的
````
  let t1,t2
  effect(() => {
    console.log("eff1");
    effect(() => {
      console.log("eff2");
      t2 = obj.ok;
    });
    t1 = obj.text;
  });
````
这种情况下我们希望修改obj.text的时候会打印eff1，接着再触发内部的副作用函数打印eff2。当实际上打印出来的是eff2。

![请添加图片描述](https://img-blog.csdnimg.cn/afe73fa9f2014467adc764289c146717.jpeg)

可以看到，当前text的集合中没有副作用函数。

这是因为我们通过activeEffect存储当前的副作用函数，**这只能存储一个副作用函数，当发生嵌套副作用函数的时候，后面的副作用函数会把先前的副作用函数覆盖掉。** 导致在注册完副作用函数后，text的集合其实是内部的副作用函数，所以在对其进行修改后，set操作中执行的副作用函数不会对text做get操作，cleanup之后不会重新add副作用函数到集合中，所以它的集合是空的。

为了解决这个问题我们创建一个副作用函数栈effectStack，在副作用函数执行时，把当前的副作用函数压入栈中，并且始终让activeEffect指向顶部的副作用函数。
````
  const effectStack = [];
    function effect(fn) {
    const effectFn = () => {
      cleanup(effectFn);
      activeEffect = effectFn;
      //在调用副作用函数之前将其压入栈中
      effectStack.push(effectFn);
      fn();
      //当前副作用函数执行完成后，将其弹出栈，并让activeEffect还原到之前的值
      effectStack.pop();
      activeEffect = effectStack[effectStack.length - 1];
    };
    effectFn.deps = [];
    effectFn();
  }
````

![请添加图片描述](https://img-blog.csdnimg.cn/b90d82c6ca0d4dc18701954f77536322.jpeg)


修改obj.text的时候，触发外部副作用函数，将其压入栈底，再执行内部副作用函数压入到栈顶，执行后再将其弹出。

![请添加图片描述](https://img-blog.csdnimg.cn/1c0ea4985e974565a1da31ab1cee4567.jpeg)


这样一来，响应式数据的属性就只会收集与其绑定的副作用函数。

![请添加图片描述](https://img-blog.csdnimg.cn/99a539f9ad5c4211a9929f98e913aeab.jpeg)
