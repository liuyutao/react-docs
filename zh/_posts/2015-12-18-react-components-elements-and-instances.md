---
title: "React 组件, 元素, 和 实例"
author: gaearon
---

 **组件, 它们的 实例, 和 元素** 之间的差别困扰了很多React新手. 渲染在屏幕上的某个元素为什么会存在三种不同的术语呢？
 
 如果你刚刚接触React,你可能之前仅仅用过组件类，和实例。例如，你可能创建一个类来声明一个 *组件* `Button`。当运行一个App时，在屏幕上你可能会拥有数个此组件类的 *实例* ，每一个 *实例* 都带有它自己的属性和内部状态。这就是传统的面向对象UI编程。为什么还要介绍 *元素* 呢？

在这个传统的UI模型中，需要由你来关注生成和销毁子组件实例的工作。如果一个 `Form` 组件想要渲染一个 `Button` 组件,它需要生成它的实例，并且当有新信息时维持它的更新。 


```js
class Form extends TraditionalObjectOrientedView {
  render() {
    // 读取传递给界面的一些数据
    const { isSubmitted, buttonText } = this.attrs;

    if (!isSubmitted && !this.button) {
      //表单尚未提交，创建按钮！
        this.button = new Button({
        children: buttonText,
        color: 'blue'
      });
      this.el.appendChild(this.button.el);
    }

    if (this.button) {
      // 按钮可见，更新它的文本！
      this.button.attrs.children = buttonText;
      this.button.render();
    }

    if (isSubmitted && this.button) {
      //表单已提交，销毁按钮!
      this.el.removeChild(this.button.el);
      this.button.destroy();
    }

    if (isSubmitted && !this.message) {
      // 表单已提交，显示成功信息!
      this.message = new Message({ text: 'Success!' });
      this.el.appendChild(this.message.el);
    }
  }
}
```

这是一段伪段码，但这大概就是当你用类似Backbone的库，以一种面向对象的方式去写复合的UI代码时，会不断遇到的情况吧。

每一个组件实例都必须要保持一些指向它的DOM节点以及子组件实例的引用，并且在合适的时间去创建，更新，销毁它们。随着组件可能的状态数量的增长，代码行数也将成平方级别增长，而且父级直接访问他们的子组件实例，使得很难在将来对其解耦。

所以，React有什么不同呢？

## 用元素来描述树

在React中， *元素* 的作用是， **元素是一个纯对象 *描述* 一个组件实例或DOM节点以及它们所需要的属性。** 它仅仅包含这些信息，它的组件类型（比如：`Button`),它的属性（比如：它的`color`）,和它所包含的子元素。


一个元素不是一个真实的实例。而是，告诉React你 *想要* 在屏幕上看到什么。你不能在元素上调用任何函数。它只是一个不可改变的描述对象，它有两个字段:`type: (string | ReactClass)` and `props: Object`[^1].


### DOM 元素

当一个元素的`type`是一个字符串，它表示一个有此标签名的DOM节点，`props`对应它的标签属性。这就是React要渲染的。比如：


```js
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

这个元素就是用一个纯对象的方式来表示以下的HTML：

```html
<button class='button button-blue'>
  <b>
    OK!
  </b>
</button>
```

注意元素们是怎么被嵌套的。按照约定，当我们想要创建一个元素的树的时候，我们会在父级元素的`children`属性上指定一个或多个子元素。

关键在于不管子元素还是父元素，它们都 *只是描述，而不是真正的实例*。它们不会指向屏幕上的任何东西。你可以创建或销毁它们，但这并没有什么影响。

React 元素很容易去遍历，不需要被解析，当然它们也比真实的DOM元素要轻量很多-它们只是对象而已！

### 组件元素

然而， 一个元素的`type`也可以是一个函数名，或一个React组件的类：

```js
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

这是React的核心观念。

**一个描述组件的元素仍然是一个元素，就像一个描述DOM节点的元素一样。它们可以互相被嵌套和混合。**

这个特性可以让你将一个具有特别`color`属性值的`Button`定义成一个`DangerButton`组件，而不需要担心`Button`会被渲染成是一个DOM的 `<button>`，或一个`<div>`, 还是完全的其它什么东西。

```js
const DangerButton = ({ children }) => ({
  type: Button,
  props: {
    color: 'red',
    children: children
  }
});
```

你可以混入和搭配DOM和组件元素在一个单元素树里：

You can mix and match DOM and component elements in a single element tree:

```js
const DeleteAccount = () => ({
  type: 'div',
  props: {
    children: [{
      type: 'p',
      props: {
        children: 'Are you sure?'
      }
    }, {
      type: DangerButton,
      props: {
        children: 'Yep'
      }
    }, {
      type: Button,
      props: {
        color: 'blue',
        children: 'Cancel'
      }
   }]
});
```

或者，你更喜欢JSX的话：

```js
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color='blue'>Cancel</Button>
  </div>
);
```

这样的混入和搭配方式可以使组件们之间相互解耦，因为通过这样特别的组成，可以同时表达出 *是一个* 和 *有一个* 的关系。

* `Button` 是一个 有特殊属性的 DOM `<button>`.
* `DangerButton` 是一个 有特殊属性的 `Button` .
* `DeleteAccount` 是在`<div>` 中包含一个 `Button` 和一个`DangerButton`.

### 包裹元素树的组件 

当React看到一个元素有一个函数或`type`类时，它知道要问 *那个* 组件，给定了对应`props`后，它要渲染成什么元素。


当它看到这个元素时：


```js
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

React 将问`Button` 它要渲染成什么。`Button`将返回这个元素：

```js
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

React将重复这个过程直到它把页面上每一个在组件下面潜在的DOM标签元素都搞清楚为止。

Reac 就像一个小孩子，当你告诉他“X 就是Y ”这类解释时，会不停的问"那什么是Y呢",直到他们能理解这世上的每一件小事情。

还记得上面的`Form` 例子吗？用React来写就是下面的样子[^1]:

```js
const Form = ({ isSubmitted, buttonText }) => {
  if (isSubmitted) {
    // Form submitted! Return a message element.
    return {
      type: Message,
      props: {
        text: 'Success!'
      }
    };
  }

  // Form is still visible! Return a button element.
  return {
    type: Button,
    props: {
      children: buttonText,
      color: 'blue'
    }
  };
};
```

就是这个样子，对于一个React组件来说，属性是输入，一个元素树就是输出。

**返回的元素树可以包括描述DOM节点的元素，和描述其它组件的元素。这样就允许你把UI的独立成员组合起来，而不用关心它们的内部DOM结构。**

我们让React创建，更新，销毁实例，我们用通过组件返回的元素来描述它们，然后，React来管理这些实例。

### 组件可以是类，也可以是函数

在上面的代码中， `Form`, `Message`, 和 `Button` 都是React 组件。它们也可以像上面一样写成函数，或写成继承自`React.Component`的类。这三种声明一个组件的方式基本上是等价的。

```js
// 1) As a function of props
const Button = ({ children, color }) => ({
  type: 'button',
  props: {
    className: 'button button-' + color,
    children: {
      type: 'b',
      props: {
        children: children
      }
    }
  }
});

// 2) Using the React.createClass() factory
const Button = React.createClass({
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
});

// 3) As an ES6 class descending from React.Component
class Button extends React.Component {
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
}
```

当一个组件用类的方式来定义时，它要比一个函数式组件强大一点的。当对应的DOM节点在创建或销毁时，它可以保存一些内部状态，应用自定义的逻辑。

函数式组件有点弱但是更简单，而且表现也跟类式组件一样只有一个`render`方法。除非你需要用到只有类式组件里才有的特性。我们推荐你使用函数式组件来代替类式组件。

**不管怎样，无论函数式还是类式，本质上它们都是React的组件。它们都以props作为输入，以元素作为它们的输出。**

### 从上至下的整合

当你执行：

```js
ReactDOM.render({
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}, document.getElementById('root'));
```

React 将询问`Form` 组件，给定这些`属性`，它能返回一个什么样的元素树呢。它将用更简单的基本类型的术语，来一步步地“精练”它对你的组件树的理解：


```js
// React: 你告诉我这些...
{
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}

// React: ...然后， Form又告诉我这些...
{
  type: Button,
  props: {
    children: 'OK!',
    color: 'blue'
  }
}

// React: ...并且， Button告诉我这些! 我想我已经搞定了。
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

这是当React调用[整合](/react/docs/reconciliation.html) 时过程中的一部分，而当你调用[`ReactDOM.render()`](/react/docs/top-level-api.html#reactdom.render) 或[`setState()`](/react/docs/component-api.html#setstate)时，就会开始这个过程。到了整合过程的结尾，React了解了DOM树的结果，一个渲染器如`react-dom` 或 `react-native` 将使用一个最小的必要的更新集，更新到DOM节点上（或者，在React-Native上，是特定平台的界面上）。

这个逐渐的提炼过程也是React 应用易于优化的原因。如果你的组件树的某些部分变得过于庞大，使得React访问它们变得效率降低时，你可以告诉它[如果相关属性未曾改变，则跳过对这个树的某些特定部分的“提炼”或差异比较](/react/docs/advanced-performance.html)。如果属性是不可变的话，计算属性是否改变将非常迅速，所以，React和不可变性要很紧密地工作，并且可以以最小的代价提价极大的优化空间。

你可能已经注意到本篇文章讲了很多关于组件和元素，对于实例却没有多少。实际上，与其它的大部分面向对象的UI框架相比，在React中的实例要不重要的多。

只有用类声明的组件拥有实例，而且你永远不会直接创建它们：React 会替你做这些。既然存在[父组件实例访问子组件实例的机制](/react/docs/more-about-refs.html)，那只有当必要的动作(比如给一个字段获取焦点)时才需要使用它们，一般情况下应该避免使用。

React 可以负责为每一个组件类创建实例，所以你可以使用方法与内部状态以面向对象的方式来写组件，除此之外，实例在React的编程模型中并不是非常重要，React可以自己管理。

## 总结

一个 *元素* 就是一个纯对象，它以DOM节点或其它组件的形式描述了你想在屏幕上显示什么。元素们也可以在它们的props中包含其它的元素们。创建React元素非常容易，一量创建好一个元素，那它就永远也不会改变。

一个 *组件* 可以用几种不同的方式来声明。它可以是具有`render()`方法的类。又或者，在简单点的场景下，可以用函数来定义它。不管哪种，它都使用props作为输入，元素树作为输出。

一个组件接收几个属性作为输入，

When a component receives some props as an input, it is because a particular parent component returned an element with its `type` and these props. This is why people say that the props flows one way in React: from parents to children.

An *instance* is what you refer to as `this` in the component class you write. It is useful for [storing local state and reacting to the lifecycle events](/react/docs/component-api.html).

Functional components don’t have instances at all. Class components have instances, but you never need to create a component instance directly—React takes care of this.

Finally, to create elements, use [`React.createElement()`](/react/docs/top-level-api.html#react.createelement), [JSX](/react/docs/jsx-in-depth.html), or an [element factory helper](/react/docs/top-level-api.html#react.createfactory). Don’t write elements as plain objects in the real code—just know that they are plain objects under the hood.

## 深度阅读

* [Introducing React Elements](/react/blog/2014/10/14/introducing-react-elements.html)
* [Streamlining React Elements](/react/blog/2015/02/24/streamlining-react-elements.html)
* [React (Virtual) DOM Terminology](/react/docs/glossary.html)

[^1]: All React elements require an additional ``$$typeof: Symbol.for('react.element')`` field declared on the object for [security reasons](https://github.com/facebook/react/pull/4832). It is omitted in the examples above. This blog entry uses inline objects for elements to give you an idea of what’s happening underneath but the code won’t run as is unless you either add `$$typeof` to the elements, or change the code to use `React.createElement()` or JSX.
