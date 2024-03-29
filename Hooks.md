# React-hooks Abstract Components

Click on the [Public Number] (#Public Number) to get the latest updates to the document, and you can receive the ** Front End Interview Manual** and the most standard resume template** that are included in this guide.

The main content of this article is from the front-end intensive series [How to make wheels with React Hooks] (https://github.com/dt-fe/weekly/blob/v2/080.%E7%B2%BE%E8%AF%BB %E3%80%8A%E6%80%8E%E4%B9%88%E7%94%A8%20React%20Hooks%20%E9%80%A0%E8%BD%AE%E5%AD%90%E3 %80%8B.md)

## 1 Introduction

Last week's [Intensive Reading of React Hooks] (https://github.com/dt-fe/weekly/blob/master/79.%E7%B2%BE%E8%AF%BB%E3%80%8AReact% 20Hooks%E3%80%8B.md) Basic understanding of React Hooks has been implemented. Maybe you also looked at the basic implementation of React Hooks (that is, arrays), but can you understand the implementation principle? Learning is knowledge, but using skills, seeing other people's usage is like brushing sounds (wow, can you eat this way?), you will always have new gains.

This article takes this knowledge into practice and sees how the vast majority of the working people discover the potential of React Hooks.

First of all, from the perspective of use, to understand the characteristics of React Hooks is "very convenient Connect everything", so whether it is data stream, Network, or timer can be monitored, there is a little RXJS meaning, that is, you can use React Hooks, the React component is made: any change of things is the input source, when these sources change, it will re-trigger the render of the React component, you only need to pick which data source the component binds (which Hooks are used), and then just write render The function will do!

## 2 Intensive reading

After referring to some of the React Hooks components, I have classified them according to their functions.

> Because React Hooks are not very complex, they are not classified according to the technical implementation. After all, technology will be proficient one day, and it will have a long-lasting reference value according to functional classification.

###DOM Side Effects Modification / Monitoring

To make a web page, there are always some troubles that seem to have little to do with the components, such as modifying the page title (switching the page to remember to change to the default title), listening to the page size change (component destruction remembers to cancel the monitoring), prompting when disconnecting the network (a Layers of decorators have to be piled up into hills). React Hooks is especially good at doing these things, making these wheels the right size.

> Since React Hooks reduces the cost of using high-end components, the “judgment” that can be completed in a single lifecycle will be very simple.

Here are a few examples:

####Modify page title

Effect: The page title is set by calling the `useDocumentTitle` function in the component, and when the page is switched, the page title is reset to the default title "front-end intensive reading".

```tsx
useDocumentTitle("Personal Center");
```

Implementation: Assign directly with `document.title`, no more simple. Give a default title again when destroying. This simple function can be abstracted in the project tool function, and each page component needs to be called.

```tsx
Function useDocumentTitle(title) {
  useEffect(
    () => {
      Document.title = title;
      Return () => (document.title = "front-end intensive");
    },
    [title]
  );
}
```

[Online Demo] (https://codesandbox.io/s/lrnvnx866l)

#### Monitor page size changes, network disconnected

Effect: When the component calls `useWindowSize`, it gets the page size and automatically triggers the component update when the browser zooms.

```tsx
Const windowSize = useWindowSize();
Return <div> page height: {windowSize.innerWidth}</div>;
```

Realization: It is basically the same as the title idea. This time, you can get the page width and height directly from the API such as `window.innerHeight`. Note that you can use `window.addEventListener('resize')` to listen to the page size change. `setValue` will trigger the call to its own UI component renderer, it's that simple!

Finally, note that when destroying, `removeEventListener` logs out.

```tsx
Function getSize() {
  Return {
    innerHeight: window.innerHeight,
    innerWidth: window.innerWidth,
    outerHeight: window.outerHeight,
    outerWidth: window.outerWidth
  };
}

Function useWindowSize() {
  Let [windowSize, setWindowSize] = useState(getSize());

  Function handleResize() {
    setWindowSize(getSize());
  }

  useEffect(() => {
    window.addEventListener("resize", handleResize);
    Return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  Return windowSize;
}
```

[Online Demo] (https://codesandbox.io/s/j2rz2mj83)

#### Dynamic Injection css

Effect: Inject a class into the page, and remove the class when the component is destroyed.

```tsx
Const className = useCss({
  Color: "red"
});

Return <div className={className}>Text.</div>;
```

Implementation: As you can see, the convenience of Hooks is to remove side effects when components are destroyed, so we can safely use Hooks to do some side effects. It is natural to inject css, and destroy css. Just find the injected reference and destroy it. You can see this [code fragment] (https://github.com/streamich/nano-css/blob/c21413ddbed233777886f7c9aa1375af8a221f7b/addon) /pipe.js#L51).

> DOM side-effect modification/listening scenes have some ready-made libraries, as you can see from the name: [document-visibility](https://github.com/rehooks/document-visibility), [network-status]( Https://github.com/rehooks/network-status), [online-status](https://github.com/rehooks/online-status), [window-scroll-position](https://github. Com/rehooks/window-scroll-position), [window-size](https://github.com/rehooks/window-size), [document-title](https://github.com/rehooks/document- Title).

### Component Assistance

Hooks can also enhance component capabilities, such as getting and listening to the runtime and height of components.

#### Get component width and height

Effect: Get the width and height of a component ref instance by calling `useComponentSize`, and get the latest width and height when the width and height change.

```tsx
Const ref = useRef(null);
Let componentSize = useComponentSize(ref);

Return (
  <>
    {componentSize.width}
    <textArea ref={ref} />
  </>
);
```

Implementation: Similar to DOM listening, this time it uses the `ResizeObserver` to listen to the component ref, and destroys the listener when the component is destroyed.

The essence is to listen to some side effects, but through the delivery of ref, we can monitor and manipulate the component granularity.

```tsx
useLayoutEffect(() => {
  handleResize();

  Let resizeObserver = new ResizeObserver(() => handleResize());
  resizeObserver.observe(ref.current);

  Return () => {
    resizeObserver.disconnect(ref.current);
    resizeObserver = null;
  };
}, []);
```

[Online Demo] (https://codesandbox.io/s/zqxp3l9yrm), corresponding component [component-size](https://github.com/rehooks/component-size).

#### Get the value thrown by the component onChange

Effect: Get the value entered by the current user in the Input box via `useInputValue()` instead of manually listening to onChange and then throwing a `otherInputValue` and a callback function to write the heap logic in an unrelated place.

```tsx
Let name = useInputValue("Jamie");
// name = { value: 'Jamie', onChange: [Function] }
Return <input {...name} />;
```

As you can see, this not only does not occupy the component's own state, nor does it need to be handwritten with the onChange callback function, which is compressed into a row of use hooks.

Implementation: Read this should be roughly guessable, use `useState` to store the value of the component, and throw `value` and `onChange`, listen to `onChange` and modify `value` by `setValue`, you can The `onChange` triggers the call to the component's renderer.

```tsx
Function useInputValue(initialValue) {
  Let [value, setValue] = useState(initialValue);
  Let onChange = useCallback(function(event) {
    setValue(event.currentTarget.value);
  }, []);

  Return {
    Value,
    onChange
  };
}
```

It should be noted here that when we enhance the component, the callback of the ** component generally does not need to destroy the listener, and only needs to be listened once, which is different from the DOM listener, so for most scenarios, we need to use the `useCallback` package. And pass an empty array to ensure that it will only be listened for once, and there is no need to unregister the callback when the component is destroyed.

[Online Demo] (https://codesandbox.io/s/0xlk250l5l), right

Should be a component [input-value] (https://github.com/rehooks/input-value).

### Doing animation

Using React Hooks to animate, generally get some values ​​with elastic changes, we can assign values ​​to components such as progress bars, so that the progress of the changes is in line with some animation curve.

#### Get a value between 0-1 in a certain period of time

This is the most basic concept of animation, getting a linearly increasing value at some time.

Effect: The number between 0-1 that is constantly refreshed within t milliseconds is obtained by `useRaf(t)`, during which the component will be refreshed continuously, but the refresh rate is controlled by requestAnimationFrame (the UI will not be stuck).

```tsx
Const value = useRaf(1000);
```

Implementation: It is rather tedious to write, here is a brief description. Use `requestAnimationFrame` to give a value between 0-1 at a given time. Then, each time you refresh, just judge the ratio of the current refreshed time point to the total time, and then do the denominator, the numerator is 1.

[Online Demo] (https://codesandbox.io/s/n745x9pyy4), corresponding component [use-raf] (https://github.com/streamich/react-use/blob/master/docs/useRaf.md) .

#### Flexible animation

Effect: The animation value is obtained by `useSpring`, the component is refreshed at a fixed frequency, and the animation value is increased or decreased by an elastic function.

The actual calling method is generally to first get a value through `useState`, and then wrap this value through the animation function, so that the component will be refreshed from the original one, and then refreshed N times, and the value obtained is also along with the animation function. The rule changes, and finally the value will settle to the final input value (such as `50` in the example).

```tsx
Const [target, setTarget] = useState(50);
Const value = useSpring(target);

Return <div onClick={() => setTarget(100)}>{value}</div>;
```

Implementation: In order to achieve animation effects, you need to rely on the `rebound` library, which can be used to disassemble a target value into a function that conforms to the elastic animation function process. Then we need to use React Hooks to do the first time to receive the target value. Call `spring.setEndValue` to trigger the animation event, and do a one-time listener in `useEffect`, and then re-set `setValue` when the value changes.

The most amazing `setTarget` linkage `useSpring` recalculates the elastic animation part, which is implemented by the second parameter `useEffect`:

```tsx
useEffect(
  () => {
    If (spring) {
      spring.setEndValue(targetValue);
    }
  },
  [targetValue]
);
```

That is, when the target value changes, a new round of renderer will be performed, so `useSpring` does not need to listen to the `setTarget` at the call, it only needs to listen to the change of `target`, and clever use of `useEffect` The second parameter can do more with less.

[Online Demo] (https://codesandbox.io/s/yq0moqo8mv)

#### Tween Animation

Understand the principle of flexible animation, Tween animation is even simpler.

Effect: Get a value from 0 to 1 through `useTween`. The animation curve for this value is `tween`. As you can see, since the range of values ​​is fixed, we don't need to give the initial value.

```tsx
Const value = useTween();
```

Implementation: Get a linearly increasing value (interval is 0 to 1) through `useRaf`, and then map it to 0 to 1 through the `easing` library. Here we use the hook of the hook call hook (drive `useTween` via `useRaf`), and it can be used in other places.

```tsx
Const fn: Easing = easing[easingName];
Const t = useRaf(ms, delay);

Return fn(t);
```

### Send request

With Hooks, any request Promise can be encapsulated as an object with standard state: loading, error, result.

#### Universal Http package

Effect: Disassemble a Promise into three objects: loading, error, result by `useAsync`.

```tsx
Const { loading, error, result } = useAsync(fetchUser, [id]);
```

Implementation: In the initial setting of Promise loading, set result after the end, if error, set error, here can be packaged as `useAsyncState` to handle, here will not be released.

```tsx
Export function useAsync(asyncFunction) {
  Const asyncState = useAsyncState(options);

  useEffect(() => {
    Const promise = asyncFunction();
    asyncState.setLoading();
    Promise.then(
      Result => asyncState.setResult(result);,
      Error => asyncState.setError(error);
    );
  }, params);
}
```

The specific code can refer to [react-async-hook] (https://github.com/slorber/react-async-hook/blob/master/src/index.js). This function suggests only understanding the principle. Some boundary conditions need to be considered, such as the component isMounted to request the result accordingly.

#### Request Service

The business layer generally abstracts a `request service` to do a unified abstraction (such as a unified url, or a unified socket implementation, etc.). If the previous low method is:

```tsx
Async componentDidMount() {
  // setState: change isLoading state
  Try {
    Const data = await fetchUser()
    // setState: change isLoading, error, data
  } catch (error) {
    // setState: change isLoading, error
  }
}
```

Later, the request is placed in redux, and the way of injection via connect will change slightly:

```tsx
@Connect(...)
Class App extends React.PureComponent {
  Public componentDidMount() {
    this.props.fetchUser()
  }

  Public render() {
    // this.props.userData.isLoading | error | data
  }
}
```

Finally, you will find that Hooks are simple and clear:

```tsx
Function App() {
  Const { isLoading, error, data } = useFetchUser();
}
```

And `useFetchUser` can be easily written using the `useAsync` package above:

```tsx
Const fetchUser = id =>
  Fetch(`xxx`).then(result => {
    If (result.status !== 200) {
      Throw new Error("bad status = " + result.status);
    }
    Return result.json();
  });

Function useFetchUser(id) {
  Const asyncFetchUser = useAsync(fetchUser, id);
  Return asyncUser;
}
```

### Fill out the form

React Hooks is especially suitable for forms, especially [antd form] (https://ant.design/components/form-cn/). If Hooks is supported, it will be much easier to use:

```tsx
Function App() {
  Const { getFieldDecorator } = useAntdForm();

  Return (
    <Form onSubmit={this.handleSubmit} className="login-form">
      <FormItem>
        {getFieldDecorator("userName", {
          Rules: [{ required: true, message: "Please input your username!" }]
        })(
          <Input
            Prefix={<Icon type="user" style={{ color: "rgba(0,0,0,.25)" }} />}
            Placeholder="Username"
          />
        )}
      </FormItem>
      <FormItem>
        <Button type="primary" htmlType="submit" className="login-form-button">
          Log in
        </Button>
        Or <a href="">register now!</a>
      </FormItem>
    </Form>
  );
}
```

However, the `getFieldDecorator` is still based on the RenderProps idea. The thorough Hooks idea is to use the **component helper method described earlier to provide a set of component methods and pass them to the component **.

#### Hooks Thinking Form Components

Effect: Get the form value via `useFormState` and provide a series of **Component Assist** methods to control the component state.

```tsx
Const [formState, { text, password }] = useFormState();
Return (
  <form>
    <input {...text("username")} required />
    <input {...password("password")} required minLength={8} />
  </form>
);
```

The form value can be obtained at any time via `formState`, and some verification information is passed to the `input` component via `password("pwd")`, and the component is brought into a controlled state, and the input type is `password` The form key is `pwd`. And you can see that the `form` used is a native tag, and this form enhancement is quite decoupled.

Realization: A closer look at the structure, it is not difficult to find that we only need to combine the ** component auxiliary section to say
The idea of ​​the "values ​​thrown by the component onChange" section makes it easy to understand how `text`, `password` work on the `input` component and get its input state**.

To put it simply, just by merging these states, you can do this by aggregating `useReducer` to `formState`.

For the sake of simplicity, we only consider enhancements to `input`, which only takes 30 lines:

```tsx
Export function useFormState(initialState) {
  Const [state, setState] = useReducer(stateReducer, initialState || {});

  Const createPropsGetter = type => (name, ownValue) => {
    Const hasOwnValue = !!ownValue;
    Const hasValueInState = state[name] !== undefined;

    Function setInitialValue() {
      Let value = "";
      setState({ [name]: value });
    }

    Const inputProps = {
      Name, // add type: text or password to input
      Get value() {
        If (!hasValueInState) {
          setInitialValue(); // give the initialization value
        }
        Return hasValueInState ? state[name] : ""; // Assignment
      },
      onChange(e) {
        Let { value } = e.target;
        setState({ [name]: value }); // Modify the value of the corresponding Key
      }
    };

    Return inputProps;
  };

  Const inputPropsCreators = ["text", "password"].reduce(
    (methods, type) => ({ ...methods, [type]: createPropsGetter(type) }),
    {}
  );

  Return [
    { values: state }, // formState
    inputPropsCreators
  ];
}
```

The above 30 lines of code implement the setting of the `input` tag type, listen to `value` `onChange`, and finally aggregate to the big `values` as `formState`. Reading here should find that the application of React Hooks is invariable, especially the acquisition of component information, through deconstruction, and the internal aggregation of Hooks to complete the basic functions of the form component.

In fact, a complete wheel also needs to consider the compatibility of `checkbox` `radio` and the verification problem. These ideas are similar. The specific source code can be seen [react-use-form-state] (https://github.com/wsmd /react-use-form-state).

### Simulation life cycle

Sometimes React15's API is quite useful, and React Hooks can be used to simulate a full set.

#### componentDidMount

Effect: A callback function that is executed by the `useMount` to get the mount cycle.

```tsx
useMount(() => {
  // quite similar to `componentDidMount`
});
```

Implementation: `componentDidMount` is equivalent to the callback of `useEffect` (only once), so throw the callback function directly.

```tsx
useEffect(() => void fn(), []);
```

#### componentWillUnmount

Effect: A callback function that is executed by undo cycle with `useUnmount`.

```tsx
useUnmount(() => {
  // quite similar to `componentWillUnmount`
});
```

Implementation: `componentWillUnmount` is equivalent to the return value of the callback function of `useEffect` (only once), so you can directly throw the return value of the callback function.

```tsx
useEffect(() => fn, []);
```

#### componentDidUpdate

Effect: A callback function that is executed by the didUpdate cycle with `useUpdate`.

```tsx
useUpdate(() => {
  // quite similar to `componentDidUpdate`
});
```

Implementation: `componentDidUpdate` is equivalent to the logic of `useMount` every time it is executed, except for the first time. So use the mouting flag + unrestricted parameters to ensure that each rerender will be executed.

```tsx
Const mounting = useRef(true);
useEffect(() => {
  If (mounting.current) {
    Mounting.current = false;
  } else {
    Fn();
  }
});
```

#### Force Update

Effect: This is the most interesting. I hope to get a function `update`, which will force the current component to be refreshed each time it is called.

```tsx
Const update = useUpdate();
```

Implementation: We know that the `useState` subscript with 1 is used to update the data, and even if the data has not changed, the call will refresh the component, so we can return a `setValue` with no modified value, so that its The function is left with only the refresh component.

```tsx
Const useUpdate = () => useState(0)[1];
```

> For `getSnapshotBeforeUpdate`, `getDerivedStateFromError`, `componentDidCatch` Currently Hooks cannot be simulated.

#### isMounted

A long time ago React provided this API and later removed it because it can be derived via `componentWillMount` and `componentWillUnmount`. Since you have React Hooks, supporting isMount is a matter of minutes.

Effect: Get the `isMounted` status via `useIsMounted`.

```tsx
Const isMounted = useIsMounted();
```

Implementation: If you see this, you should already be familiar with this routine. `useEffect` is set to true on the first call, and false when the component is destroyed. Note that you can add a second argument to the empty array to optimize performance.

```tsx
Const [isMount, setIsMount] = useState(false);
useEffect(() => {
  If (!isMount) {
    setIsMount(true);
  }
  Return () => setIsMount(false);
}, []);
Return isMount;
```

[Online Demo] (https://codesandbox.io/s/5zwr1l1o1n)

### Save data

The previous article mentioned that Reuse Hooks' built-in `useReducer` can simulate Redux's reducer behavior, and the only thing that needs to be added is to persist the data. We consider the minimal implementation, which is the global Store + Provider part.

#### Global Store

Effect: Create a global Store via `createStore`, then inject `store` into the `context` of the child component via `StoreProvider`, and finally get and manipulate through two Hooks: `useStore` and `useAction`:

```tsx
Const store = createStore({
  User: {
    Name: "小明",
    setName: (state, payload) => {
      State.name = payload;
    }
  }
});

Const App = () => (
  <StoreProvider store={store}>
    <YourApp />
  </StoreProvider>
);

Function YourApp() {
  Const userName = useStore(state => state.user.name);
  Const setName = userAction(dispatch => dispatch.user.setName);
}
```

Implementation: The implementation of this example can separate an article, so the author analyzes the implementation of `StoreProvider` from the perspective of storing data.

Yes, Hooks does not solve the Provider problem, so the global state must have a Provider, but this Provider can be easily solved with React's built-in `createContext`:

```tsx
Const StoreContext = createContext();

Const StoreProvider = ({ children, store }) => (
  <StoreContext.Provider value={store}>{children}</StoreContext.Provider>
);
```

The rest is the problem of how `useStore` gets to the persistent store. Here we use `useContext` and the Context object we just created:

```tsx
Const store = useContext(StoreContext);
Return store;
```

More source code can be found in [easy-peasy] (https://github.com/ctrlplusb/easy-peasy), which is based on redux and provides a set of Hooks APIs.

### Encapsulating the original library

Is it necessary to rewrite all the libraries after React Hooks? Of course not, let's see how other libraries can be modified.

#### RenderProps to Hooks

Take the example of [react-powerplug](https://github.com/renatorib/react-powerplug) here.

For example, there is a renderProps library that I want to transform into Hooks:

```tsx
Import { Toggle } from 'react-powerplug'

Function App() {
  Return (
    <Toggle initial={true}>
      {({ on, toggle }) => (
        <Checkbox checked={on} onChange={toggle} />
      )}
    </Toggle>
  )
}
↓ ↓ ↓ ↓ ↓ ↓
Import { useToggle } from 'react-powerhooks'

Function App() {
  Const [on, toggle] = useToggle()
  Return <Checkbox checked={on} onChange={toggle} />
}
```

Effect: If I am the maintainer of `react-powerplug`, how do I support React Hook at the minimum cost? To be honest, there is no way to do this in one step, but it can be done in two steps.

```tsx
Export function Toggle() {
  // This is the source code for Toggle
  // balabalabala..
}

Const App = wrap(() => {
  // First step: package wrap
  Const [on, toggle] = useRenderProps(Toggle); // Step 2: Package useRenderProps
});
```

Implementation: First explain why you want to pack two layers. First, Hooks must follow React's specification. We must write a `useRenderProps` function to conform to the Hooks format. **The question is how to get Toggle to render 'on` and ` Toggle`? **The normal way should not be available, so the next best thing is to pass the Toggle from `useRenderProps` to `wrap`, ** let `wrap` construct the RenderProps execution environment to get `on` and `toggle` Call `useRenderProps` inside the `setArgs` function and let `const [on, toggle] = useRenderProps(Toggle)` implement the curve to save the country. **

```tsx
Const wrappers = []; // global storage wrappers

Export const useRenderProps = (WrapperComponent, wrapperProps) => {
  Const [args, setArgs] = useState([]);
  Const ref = useRef({});
  If (!ref.current.initialized) {
    Wrappers.push({
      WrapperComponent,
      wrapperProps,
      setArgs
    });
  }
  useEffect(() => {
    Ref.current.initialized = true;
  }, []);
  Return args; // Get the value by calling setArgs with the following wrap.
};
```

Since `useRenderProps` will be executed before `wrap`, wrappers will get Toggle first, and `wrap` will call `wrappers.pop()` directly to get the Toggle object. Then construct the execution environment of RenderProps:

```tsx
Export const wrap = FunctionComponent => props => {
  Const element = FunctionComponent(props);
  Const ref = useRef({ wrapper: wrappers.pop() }); // Get the Toggle provided by useRenderProps
  Const { WrapperComponent, wrapperProps } = ref.current.wrapper;
  Return createElement(WrapperComponent, wrapperProps, (...args) => {
    // WrapperComponent => Toggle, this step is to construct the RenderProps execution environment
    If (!ref.current.processed) {
      ref.current.wrapper.setArgs(args); // After getting on, toggle, pass the above args via setArgs.
      Ref.current.processed = true;
    } else {
      Ref.current.processed = false;
    }
    Return element;
  });
};
```

The above implementation scheme refers to [react-hooks-render-props] (https://github.com/dai-shi/react-hooks-render-props), and there is a need to be able to use it directly, but the implementation ideas can be referenced. The author's brain is quite big.

#### Hooks to RenderProps

Well, if you want Hooks to support RenderProps, you must be tempted to support both sets of syntax.

Effect: A set of code supports both Hooks and RenderProps.

Implementation: In fact, Hooks is the most convenient to wrap as RenderProps, so we use Hooks to write the core code, assuming we write the simplest `Toggle`:

```tsx
Const useToggle = initialValue => {
  Const [on, setOn] = useState(initialValue);
  Return {
    On,
    Toggle: () => setOn(!on)
  };
};
```

[Online Demo] (https://codesandbox.io/s/ppvrpnz80m)

The RenderProps component can then be easily encapsulated via the `render-props` library:

```tsx
Const Toggle = ({ initialValue, children, render = children }) =>
  renderProps(render, useToggle(initialValue));
```

[Online Demo] (https://codesandbox.io/s/249n3n4r30)

In fact, the second parameter of the `renderProps` component, in the Class form React component, receives the `this.state`, now we change the object returned by `useToggle`, which can also be understood as `state`, using the Hooks mechanism. Drive the Toggle component to renderer, thus let the child component renderer.

#### Encapsulation of the original setState enhanced library

Hooks are also particularly well-suited for encapsulating libraries that would otherwise work with setState , such as [immer](https://github.com/mweststrate/immer).

`useState` Although not `setState`, but can be understood as controlling the high-level components of `setState`, we can completely encapsulate a custom `useState`, and then built-in optimization of `setState`.

For example, the immer syntax is wrapped in `produce`, and the mutable code is passed to the immutable proxy via Proxy:

```tsx
Const nextState = produce(baseState, draftState => {
  draftState.push({ todo: "Tweet about it" });
  draftState[1].done = true;
});
```

Then this `produce` can be hidden by encapsulating a `useImmer`:

```tsx
Function useImmer(initialValue) {
  Const [val, updateValue] = React.useState(initialValue);
  Return [
    Val,
    Updater => {
      updateValue(produce(updater));
    }
  ];
}
```

How to use:

```tsx
Const [value, setValue] = useImmer({ a: 1 });

Value(obj => (obj.a = 2)); // immutable
```

## 3 Summary

This article lists the following uses of React Hooks and how they are implemented:

- DOM side effects modification / listening.
- Component assistance.
- Do animations.
- Send a request.
- Fill out the form.
- Simulate the life cycle.
- Save data.
- Encapsulate the original library.

Welcome everyone's continued supplement.

---

## No public
