今天想着重讨论一下在代码中遇到的那些循环引起的bug

一个是springboot中的循环引用，一个是react中的循环渲染

在过去刚开始写代码的日子里，我经常会无意间写出上述的这些bug，直到编译后开始运行，遇到内存泄漏运行卡顿等报错的时候，才发现这些bug的存在。不只是人类，如今的AI编程也是一样，它也会写出这些bug，在IDE中看似没有问题的代码，在运行时却会出现报错。

① springboot中的循环引用

简而言之，两个或多个Bean相互依赖，形成一个闭环的依赖链。

而且Spring的自动装配机制容易掩盖依赖关系，业务逻辑层和数据访问层等边界模糊。

举一个非常简单而经典的例子：

```java
@Service
public class A {
    @Autowired
    private B b;
}

@Service
public class B {
    @Autowired
    private A a;
}
```

最好的解决方法就是：重构代码，理清业务逻辑，消除循环依赖

虽然AI会给出一些建议，比如：添加@Lazy注解，但是这些建议往往是只是暂时解决了问题，会一定程度上影响性能。

② react中的循环渲染

简而言之，一个组件在渲染过程中，无限制地触发自身或其他组件的重新渲染，会多次调用自身的render方法

出现循环渲染会导致页面卡顿甚至程序崩溃

举一个非常简单而经典的例子：

```js
function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

在这个例子中，App组件的render方法会被调用两次，一次是在初始渲染时，一次是在点击按钮时。

正确使用react钩子，添加合理的渲染条件判断，共享状态提升等都是很好的解决方法。

③ react钩子

 useState 是最基本的 Hook，它让函数组件能够拥有自己的状态。

 useEffect 是一个 Hook，它可以让你在函数组件中执行副作用操作。

 useContext 是一个 Hook，它可以让你在函数组件中访问 Context。

 useRef 是一个 Hook，它可以让你在函数组件中访问 DOM 节点。

 useReducer 是一个 Hook，它可以让你在函数组件中使用 Redux。

 useCallback 是一个 Hook，它可以让你在函数组件中缓存函数。

 useMemo 是一个 Hook，它可以让你在函数组件中缓存计算结果。

④ context

Context 是 React 提供的一种在组件树中共享数据的方式，无需通过 props 手动传递数据到每一个需要它的组件。

在 React 应用中，数据通常是通过 props 自上而下（从父组件到子组件）传递的。但对于某些类型的属性（如地区偏好、UI 主题、用户认证信息等），这些属性需要在应用的许多组件之间共享，使用 props 传递会变得非常繁琐。Context 提供了一种在组件之间共享这些值的方式，而不必显式地通过组件树的每个层级传递 props。