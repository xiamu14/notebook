# React 中阻止事件冒泡的问题
🛴 转载自《https://www.cnblogs.com/Wayou/p/react_event_issue.html》
在正式开始前，先来看看 JS 中事件的触发与事件处理器的执行。

## **JS 中事件的监听与处理**

**事件捕获与冒泡**
DOM 事件会先后经历 **捕获** 与 **冒泡** 两个阶段。捕获即事件沿着 DOM 树由上往下传递，到达触发事件的元素后，开始由下往上冒泡。

**事件处理器**
默认情况下，事件处理器是在事件的冒泡阶段执行，无论是直接设置元素的 `onclick` 属性还是通过 `[EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)` 来绑定，后者在没有设置 `useCapture` 参数为 `true` 的情况下。
考察下面的示例：

```html
<button onclick="btnClickHandler(event)">CLICK ME</button>
<script>
  document.addEventListener("click", function(event) {
    console.log("document clicked");
  });

  function btnClickHandler(event) {
    console.log("btn clicked");
  }
</script>
```

输出:

```js
btn clicked
document clicked
```

**阻止事件的冒泡**
通过调用事件身上的 `stopPropagation()` 可阻止事件冒泡，这样可实现只我们想要的元素处理该事件，而其他元素接收不到。

```html
<button onclick="btnClickHandler(event)">CLICK ME</button>
<script>
  document.addEventListener(
    "click",
    function(event) {
      console.log("document clicked");
    },
    false
  );

  function btnClickHandler(event) {
    event.stopPropagation();
    console.log("btn clicked");
  }
</script>
```

输出：

    btn clicked

**一个阻止冒泡的应用场景**
常见的弹窗组件中，点击弹窗区域之外关闭弹窗的功能，可通过阻止事件冒泡来方便地实现，而不用这种方式的话，会引入复杂的判断当前点击坐标是否在弹窗之外的复杂逻辑。

```js
document.addEventListener("click", () => {
  // close dialog
});

dialogElement.addEventListener("click", event => {
  event.stopPropagation();
});
```

但如果你尝试在 React 中实现上面的逻辑，一开始的尝试会让你怀疑人生。

## **React 下事件执行的问题**

了解 JS 中事件的基础后，会觉得一切都没什么复杂。但在引入 React 后，事情开始起变化。将上面阻止冒泡的逻辑在 React 里实现一下，代码大概像这样：

```js
function App() {
  useEffect(() => {
    document.addEventListener("click", documentClickHandler);
    return () => {
      document.removeEventListener("click", documentClickHandler);
    };
  }, []);

  function documentClickHandler() {
    console.log("document clicked");
  }

  function btnClickHandler(event) {
    event.stopPropagation();
    console.log("btn clicked");
  }

  return <button onClick={btnClickHandler}>CLICK ME</button>;
}
```

输出:

    btn clicked
    document clicked

document 上的事件处理器正常执行了，并没有因为我们在按钮里面调用 `event.stopPropagation()` 而阻止。
那么问题出在哪？
**React 中事件处理的原理**
考虑下面的示例代码并思考点击按钮后的输出。

```jsx
import React, { useEffect } from "react";
import ReactDOM from "react-dom";

window.addEventListener("click", event => {
  console.log("window");
});

document.addEventListener("click", event => {
  console.log("document:bedore react mount");
});

document.body.addEventListener("click", event => {
  console.log("body");
});

function App() {
  function documentHandler() {
    console.log("document within react");
  }

  useEffect(() => {
    document.addEventListener("click", documentHandler);
    return () => {
      document.removeEventListener("click", documentHandler);
    };
  }, []);

  return (
    <div
      onClick={() => {
        console.log("raect:container");
      }}
    >
      <button
        onClick={event => {
          console.log("react:button");
        }}
      >
        CLICK ME
      </button>
    </div>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));

document.addEventListener("click", event => {
  console.log("document:after react mount");
});
```

现在对代码做一些变动，在 body 的事件处理器中把冒泡阻止，再思考其输出。

```js
document.body.addEventListener("click", event => {
+  event.stopPropagation();
  console.log("body");
});
```

下面是剧透环节，如果你懒得自己实验的话。
点击按钮后的输出：

    body
    document:bedore react mount
    react:button
    raect:container
    document:after react mount
    document within react
    window

bdoy 上阻止冒泡后，你可能会觉得，既然 body 是按钮及按钮容器的父级，那么按钮及容器的事件会正常执行，事件到达 body 后， body 的事件处理器执行，然后就结束了。 document 上的事件处理器一个也不执行。
事实上，按钮及按钮容器上的事件处理器也没执行，只有 body 执行了。
输出：

    body

通过下面的分析，你能够完全理解上面的结果。
**SyntheticEvent**
React 有自身的一套事件系统，叫作 [SyntheticEvent](https://reactjs.org/docs/events.html)。叫什么不重要，实现上，其实就是通过在 document 上注册事件代理了组件树中所有的事件（[facebook/react#4335](https://github.com/facebook/react/issues/4335#issuecomment-120269153)），并且它监听的是 document 冒泡阶段。你完全可以忽略掉 SyntheticEvent 这个名词，如果觉得它有点让事情变得高大上或者增加了一些神秘的话。
除了事件系统，它有自身的一套，另外还需要理解的是，界面上展示的 DOM 与我们代码中的 DOM 组件，也是两样东西，需要在概念上区分开来。
所以，当你在页面上点击按钮，事件开始在原生 DOM 上走捕获冒泡流程。React 监听的是 document 上的冒泡阶段。事件冒泡到 document 后，React 将事件再派发到组件树中，然后事件开始在组件树 DOM 中走捕获冒泡流程。
现在来尝试理解一下输出结果：

- 事件最开始从原生 DOM 按钮一路冒泡到 body，body 的事件处理器执行，输出 `body`。注意此时流程还没进入 React。为什么？因为 React 监听的是 document 上的事件。
- 继续往上事件冒泡到 document。
    - 事件到达 document 之后，发现 document 上面一共绑定了三个事件处理器，分别是代码中通过 `document.addEventListener` 在 `ReactDOM.render` 前后调用的，以及一个隐藏的事件处理器，是 [ReactDOM 绑定的](https://github.com/facebook/react/blob/e8857918422b5ce8505ba5ce4a2d153e509c17a1/packages/react-dom/src/events/ReactBrowserEventEmitter.js#L105-L173)，也就是前面提到的 React 用来代理事件的那个处理器。
    - 同一元素上如果对同一类型的事件绑定了多个处理器，会按照绑定的顺序来执行。
    - 所以 `ReactDOM.render` 之前的那个处理器先执行，输出 `document:before react mount`。
    - 然后是 React 的事件处理器。此时，流程才真正进入 React，走进我们的组件。组件里面就好理解了，从 button 冒泡到 container，依次输出。
    - 最后 `ReactDOM.render` 之后的那个处理器先执行，输出 `document:after react mount`。
- 事件完成了在 document 上的冒泡，往上到了 window，执行相应的处理器并输出 `window`。

理解 **React 是通过监听 document 冒泡阶段来代理组件中的事件**，这点很重要。同时，区分原生 DOM 与 React 组件，也很重要。并且，React 组件上的事件处理器接收到的 `event` 对象也有别于原生的事件对象，不是同一个东西。但这个对象上有个 `nativeEvent` 属性，可获取到原生的事件对象，后面会用到和讨论它。
紧接着的代码的改动中，我们在 body 上阻止了事件冒泡，这样事件在 body 就结束了，没有到达 document，那么 React 的事件就不会被触发，所以 React 组件树中，按钮及容器就没什么反应。如果没理解到这点，光看表象还以为是 bug。
进而可以理解，如果在 `ReactDOM.render()` 之前的的 document 事件处理器上将冒泡结束掉，同样会影响 React 的执行。只不过这里需要调用的不是 `event.stopPropagation()`，而是 `event.stopImmediatePropagation()`。

```js
document.addEventListener("click", event => {
+  event.stopImmediatePropagation();
  console.log("document:bedore react mount");
});
```

输出：

    body
    document:bedore react mount

`stopImmediatePropagation` 会产生这样的效果，即，如果同一元素上同一类型的事件（这里是 `click`）绑定了多个事件处理器，本来这些处理器会按绑定的先后来执行，但如果其中一个调用了 `stopImmediatePropagation`，不但会阻止事件冒泡，还会阻止这个元素后续其他事件处理器的执行。
所以，虽然都是监听 document 上的点击事件，但 `ReactDOM.render()` 之前的这个处理器要先于 React，所以 React 对 document 的监听不会触发。
**解答前面按钮未能阻止冒泡的问题**
如果你已经忘了，这是相应的代码及输出。
到这里，已经可以解答为什么 React 组件中 button 的事件处理器中调用 `event.stopPropagation()` 没有阻止 document 的点击事件执行的问题了。因为 button 事件处理器的执行前提是事件达到 document 被 React 接收到，然后 React 将事件派发到 button 组件。既然在按钮的事件处理器执行之前，事件已经达到 document 了，那当然就无法在按钮的事件处理器进行阻止了。

## **问题的解决**

要解决这个问题，这里有不止一种方法。
**用** `**window**` **替换** `**document**`
来自 [React issue 回答](https://github.com/facebook/react/issues/4335#issuecomment-421705171)中提供的这个方法是最快速有效的。使用 window 替换掉 document 后，前面的代码可按期望的方式执行。

```jsx
function App() {
  useEffect(() => {
+    window.addEventListener("click", documentClickHandler);
    return () => {
+      window.removeEventListener("click", documentClickHandler);
    };
  }, []);

  function documentClickHandler() {
    console.log("document clicked");
  }

  function btnClickHandler(event) {
    event.stopPropagation();
    console.log("btn clicked");
  }

  return <button onClick={btnClickHandler}>CLICK ME</button>;
}
```

这里 button 事件处理器上接到到的 event 来自 React 系统，也就是 document 上代理过来的，所以通过它阻止冒泡后，事件到 document 就结束了，而不会往上到 window。
`**Event.stopImmediatePropagation()**`
组件中事件处理器接收到的 `event` 事件对象是 React 包装后的 SyntheticEvent 事件对象。但可通过它的 `nativeEvent` 属性获取到原生的 DOM 事件对象。通过调用这个原生的事件对象上的 `[stopImmediatePropagation()](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopImmediatePropagation)` 方法可达到阻止冒泡的目的。

```js
function btnClickHandler(event) {
+  event.nativeEvent.stopImmediatePropagation();
  console.log("btn clicked");
}
```

至于原理，其实前面已经有展示过。React 在 render 时监听了 document 冒泡阶段的事件，当我们的 `App` 组件执行时，准确地说是渲染完成后（`useEffect` 渲染完成后执行），又在 document 上注册了 click 的监听。此时 document 上有两个事件处理器了，并且组件中的这个顺序在 React 后面。
当调用 `event.nativeEvent.stopImmediatePropagation()` 后，阻止了 document 上同类型后续事件处理器的执行，达到了想要的效果。
但这种方式有个缺点很明显，那就是要求需要被阻止的事件是在 React render 之后绑定，如果在之前绑定，是达不到效果的。
**通过元素自身来绑定事件处理器**
当绕开 React 直接通过调用元素自己身上的方法来绑定事件时，此时走的是原生 DOM 的流程，都没在 React 的流程里面。

```jsx
function App() {
  const btnElement = useRef(null);
  useEffect(() => {
    document.addEventListener("click", documentClickHandler);
    if (btnElement.current) {
      btnElement.current.addEventListener("click", btnClickHandler);
    }

    return () => {
      document.removeEventListener("click", documentClickHandler);
      if (btnElement.current) {
        btnElement.current.removeEventListener("click", btnClickHandler);
      }
    };
  }, []);

  function documentClickHandler() {
    console.log("document clicked");
  }

  function btnClickHandler(event) {
    event.stopPropagation();
    console.log("btn clicked");
  }

  return <button ref={btnElement}>CLICK ME</button>;
}
```

很明显这样是能解决问题，但你根本不会想要这样做。代码丑陋，不直观也不易理解。

## **结论**

注意区分 React 组件的事件及原生 DOM 事件，一般情况下，尽量使用 React 的事件而不要混用。如果必需要混用比如监听 document，window 上的事件，处理 `mousemove`，`resize` 等这些场景，那么就需要注意本文提到的顺序问题，不然容易出 bug。

