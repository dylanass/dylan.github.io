---
title: 深入React函数组件的re-render原理及优化
date: 2022-03-26 21:07:19
tags: 
    - React
---

# 深入React函数组件的re-render原理及优化

对于函数组件的 **re-render** 大致分为以下三种情况

- 组件本身使用 **useState**  或者 **useReducer** 更新,引起的 **re-render**
- 父组件的 **props** 引起的 **re-render**
- 组件本身使用了 **useContext** , **context** 更新引起的 **re-render**

下面详细讨论下这些情况

<!-- more -->

## 1.useState

### 1.1常规使用

```
const Counter = () => {
  console.log('counter render');
  const [count, addCount ] = useState(0);
  return (
    <div className="counter">
      <div className="counter-num">{count}</div>
      <button onClick={() => {addCount(count + 1)}}>add</button>
    </div>
  )
}
```

每次  `click` 都会 `re-render`

### 1.2 immutation state

 将state改成引用类型

```
const Counter = () => {
  console.log("counter render");
  const [count, addCount] = useState({ num: 0, time: Date.now() });
  const clickHandler = () => {
    count.num++;
    count.time = Date.now();
    addCount(count);
  };
  return (
    <div className="counter">
      <div className="counter-num">
        {count.num}, {count.time}
      </div>
      <button onClick={clickHandler}>add</button>
    </div>
  );
};
```

[栗子链接(codesandbox)](https://codesandbox.io/s/cocky-voice-0p5z3?file=/src/App.js)

真实原因在于,更新state的时候,会有一个新老state的比较,用的是 `Object.is` 比较的,如果为 `true` 则不更新 --------- [MDN Object.is()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 

```javascript
Object.is('foo', 'foo');     // true
Object.is(window, window);   // true

Object.is('foo', 'bar');     // false
Object.is([], []);           // false

var foo = { a: 1 };
var bar = { a: 1 };
Object.is(foo, foo);         // true
Object.is(foo, bar);         // false

Object.is(null, null);       // true
```

> 所以更新state的时候要注意 `state` 为不可变数据,每次更新都需要一个新值才会有效



### 1.3 focreUpdate (强制更新)

类组件有一个 `forceUpdate` 方法,函数组件是没有该方法的,但是可以自己写一个,由于 `Object.is({},{})` 总是 `false` ,所以总能引起更新



说完 `useState` , `useReducer` 就不用说了,因为源码里面 `useState` 的更新其实调用的就是 `useReducer` , 只是传入了自带的 `reducer`   

```
function updateState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  return updateReducer(basicStateReducer, (initialState: any));
}
```



## 2.props更新引起的子组件 re-render

### 2.1 常规使用

父组件做一个计数器,子组件做一个单纯的渲染组件,无关计数.

```
const Hello = ({ name }) => {
  console.log("hello render");
  return<div>hello {name}</div>;
};

const App = () => {
  console.log("app render");
  const [count, addCount] = useState(0);
  return (
    <div className="app">
      <Hello name="react" />
      <div className="counter-num">{count}</div>
      <button
        onClick={() => {
          addCount(count + 1);
        }}
      >
        add
      </button>
    </div>
  );
};
```

> [demo 栗子](https://codesandbox.io/s/stupefied-bhabha-bimdg?file=/src/App.js)
>
> 每次执行 `addCount` 时, `Hello` 组件都会一起执行

对于这种不必要的 **re-render**,我们有手段可以优化

### 2.2 优化组件设计

#### 2.2.1 将更新部分抽离成单独的组件

将计数部分抽离成 `Counter` 组件

```
const App = () => {
  console.log("app render");
  return (
    <div className="app">
      <Hello name="react" />
      <Counter />
    </div>
  );
};
```

#### 2.2.2 将不需要re-render的部分,以插槽形式渲染(children)

```
// App 组件预留 children 位
const App = ({ children }) => {
  console.log("app render");
  const [count, addCount] = useState(0);
  return (
    <div className="app">
        {children}
      <div className="counter-num">{count}</div>
      <button
        onClick={() => {
          addCount(count + 1);
        }}
      >
        add
      </button>
    </div>
  );
};

// 使用
<App>
  <Hello name="react" />
</App>
```

> [demo 栗子](https://codesandbox.io/s/angry-christian-pglr7?file=/src/App.js)

#### 2.2.3 React.memo

- 对于是否需要re-render,类组件提供了两种方法: `PureComponent` 组件和 `shouldComponentUpdate` 生命周期方法

- 对于函数组件来说,有一个 `React.memo` 方法,可以用来决定是否需要 **re-render**,如下我们将 `Hello` 组件 memo化,除非 `Hello` 组件的 `props` 更新:

  

```
const Hello = React.memo(({ name }) => {
  console.log("hello render");
  return<div>hello {name}</div>;
});

const App = () => {
  console.log("app render");
  const [count, addCount] = useState(0);
  return (
    <div className="app">
      <Hello name="react" />
      <div className="counter-num">{count}</div>
      <button
        onClick={() => {
          addCount(count + 1);
        }}
      >
        add
      </button>
    </div>
  );
};
```



memo 方法的源码定义简略如下:

```
exportfunction memo<Props>(
  type: React$ElementType, // react 自定义组件
  compare?: (oldProps: Props, newProps: Props) => boolean, // 可选的比对函数，决定是否 re-render
) {
    ...
    const elementType = {
    $$typeof: REACT_MEMO_TYPE,
    type,
    compare: compare === undefined ? null : compare,
  };
    ...
  
  return elementType;
}
```

memo 的关键对比逻辑: 如果有 `compare` 函数则使用 `compare` 函数决定是否需要 **re-render**,

否则使用浅比较 `shallowEqual` 决定是否需要 **re-render**

如果我们给 `Hello` 组件添加一个点击事件,这时间我们发现 `Hello` 组件又开始 **re-render** 了:

```
// 新增 onClick 处理函数
const Hello = memo(({ name, onClick }) => {
  console.log("hello render");
  return<div onClick={onClick}>hello {name}</div>;
});

const App = ({ children }) => {
  console.log("counter render");
  const [count, addCount] = useState(0);
  
  // 新增处理函数
  const clickHandler = () => {
    console.log("hello click");
  };

  return (
    <div className="counter">
      <Hello name="react" onClick={clickHandler} />
      <div className="counter-num">{count}</div>
      <button
        onClick={() => {
          addCount(count + 1);
        }}
      >
        add
      </button>
    </div>
  );
};
```

这是因为每次点击计数,都会重新定义 `clickHandler` 处理函数,这样 `shallowEqual` 浅比较发现 `onClick` 属性不同(引用不同),于是将会进行 **re-render** 

#### 2.2.4 useCallback

这时候我们可以使用 `useCallback` 将定义的函数缓存起来,如下就不会引起 **re-render**了

```
// 新增处理函数，使用 useCallback 缓存起来
const clickHandler = useCallback(() => {
  console.log("hello click");
}, []);
```

`useCallback` 的原理主要是在挂载的时候,将定义的 `callback` 函数及 `deps` 依赖挂载该 `hook` 的 `memoizedState` ,当更新时,将依赖进行对比,如果依赖没变,则直接返回老的 `callback` ,否则则更新新的 `callback` 函数,及依赖:

```
//挂载时
function mountCallback(callback, deps) {
  var hook = mountWorkInProgressHook();
  var nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}

//更新时
function updateCallback(callback, deps) {
  var hook = updateWorkInProgressHook();
  var nextDeps = deps === undefined ? null : deps;
  var prevState = hook.memoizedState;

  if (prevState !== null) {
    if (nextDeps !== null) {
      var prevDeps = prevState[1];

      // 如果依赖未变，则直接返回老的函数
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  // 否则更新新的 callback 函数
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

看起来好像没问题了,但是如果我们在刚才 `callback` 中使用的 `count` 这个 `state` 呢?

```
// 新增处理函数，使用 useCallback 缓存起来
// 在 callback 函数中使用 count
const clickHandler = useCallback(() => {
  console.log("count: ", count);
}, []);
```

当我们点击了几次计数,然后再点击 `Hello` 组件时,发现 `count` 还是初始挂载的值,最不是最新的,其实,都是闭包惹的祸

所以,为了让 `callback` 中的 `state` 中可以使用最新的,我们还需要将 `state` 放入 `deps` 依赖,但是这样,依赖更新了,`callback` 函数也更新了,于是 `Hello` 组件又将会 *re-render* ,又回到了从前

这样我们得出一个结论:

当 `callback` 函数需要使用 state 的值时,如果是state更新引起的更新 ,`useCallback` 其实是没有效果的



#### 2.2.5 useRef & useEffect

为了解决刚才闭包的问题,我们换一个方式 ,引入 `useRef` 和 `useEffect` 来解决:

```
const App = ({ children }) => {
  console.log("counter render");
  const [count, addCount] = useState(0);
  
  // 1、创建一个 countRef 
  const countRef = useRef(count);
  
  // 2、依赖改成 countRef
  // 浅对比 countRef 时，将不会引起 callback 函数更新
  // callback 函数又中可以读取到 countRef.current 值，即 count 的最新值
  const clickHandler = useCallback(() => {
    console.log("count: ", countRef.current);
  }, [countRef]);
  
  // 3、当 count 更新时，更新 countRef 的值
  useEffect(() => {
      countRef.current = count;
  }, [count]);

  return (
    <div className="counter">
      <Hello name="react" onClick={clickHandler} />
      <div className="counter-num">{count}</div>
      <button
        onClick={() => {
          addCount(count + 1);
        }}
      >
        add
      </button>
    </div>
  );
};
```

> 像上面用 `useEffect` 更新 `current` 或者 `setState` 回中更新,像下面的栗子
>
> [demo 栗子](https://codesandbox.io/s/stupefied-benz-speei?file=/src/App.js)



**useRef** 保存原理如下:

```
// 挂载 ref
function mountRef(initialValue) {
  var hook = mountWorkInProgressHook();
  
  // 创建一个 ref 对象，将值挂在 current 属性上
  var ref = {
    current: initialValue
  };

  {
    Object.seal(ref);
  }
    
  // 将 ref 挂到 hook 的 memoizedState 属性上，并返回
  hook.memoizedState = ref; 
  return ref;
}

// 更新 ref
function updateRef(initialValue) {
  var hook = updateWorkInProgressHook();
  return hook.memoizedState; // 直接返回 ref
}
```

相当于修改的是 `ref` 对象上的 `current` 属性



##  context 更新,引起的 re-render

其实关于 context，我们平时都有在用，如 react-redux，react-router 都运用了 context 来进行状态管理。

[React Context 源码浅析](https://juejin.cn/post/6936421130081140744)



