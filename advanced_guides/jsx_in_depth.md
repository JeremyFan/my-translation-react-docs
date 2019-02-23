# 深入 JSX（JSX In Depth）

从根本上说，JSX 是 `React.createElement(component, props, ...children)` 这个函数的语法糖。以下这段 JSX 代码：
```js
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```
会被编译成：
```js
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```
如果没有 children，可以使用自闭合（self-closing）的标签。就像这样（so）：
```js
<div className="sidebar" />
```
会被编译成：
```js
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
```
如果你想测试指定的 JSX 转换成 JavaScript 是什么样子的，可以试试 [Babel 在线编译器]。

## 指定 React 元素类型
JSX 标签的开头决定了 React 元素的类型。

首字母大写表明 JSX 标签指向一个 React 组件。这些标签的引用被编译到一个指定的变量（These tags get compiled into a direct reference to the named variable），所以如果使用 JSX 的 `<Foo />` 表达式，`Foo` 必须在作用域中。

### React 必须在作用域中
因为 JSX 会被编译成调用 `React.createElement`，所以 `React` 库必须一直在 JSX 代码的作用域中。

比如，以下代码中的两个引入都是必须的，尽管 `React` 和 `CustomButton` 没有被 JavaScript 直接用到。
```js
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```
如果没有使用 JavaScript 打包工具，而是使用 `<script>` 标签加载 React，那么作用域中已经有 `React` 这个全局变量了。

### 在 JSX 中使用点号
在 JSX 中，也可以通过点号取到（refer to） React 组件。这样可以很方便的（取到）单个模块中导出的多个 React 组件。比如，如果 `MyComponents.DatePicker` 是个组件，那么你可以直接在 JSX 中这样用：
```js
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

### 自定义组件必须首字母大写
首字母小写的元素，是一个像 `<div>` 或 `<span>` 一样的内置（built-in）组件，（编译的）结果会把 `'div'` 或 `'span'` 传给 `React.createElement`。像 `<Foo />` 一样首字母大写的元素会被编译成 `React.createElement(Foo)`，相当于定义或从 JavaScript 文件中引入的组件。

我们推荐使用大写字母命名组件。如果你有一个小写字母开头的组件，先赋值给一个大写字母开头的变量再使用 JSX。

比如，以下代码并不会像预期一样：
```js
import React from 'react';

// Wrong! This is a component and should have been capitalized:
function hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Wrong! React thinks <hello /> is an HTML tag because it's not capitalized:
  return <hello toWhat="World" />;
}
```

把 `hello` 重命名为 `Hello` 然后使用 `<Hello />` 可以修复这个问题：
```js
import React from 'react';

// Correct! This is a component and should be capitalized:
function Hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Correct! React knows <Hello /> is a component because it's capitalized.
  return <Hello toWhat="World" />;
}
```

### 运行时选择（组件）类型
无法使用表达式选择 React 元素类型。如果想使用一般表达式表示使用哪种类型的元素，只要赋值给一个首字母大写的变量即可。这种场景常出现在要根据某个 prop 渲染不同的组件时：
```js
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Wrong! JSX type can't be an expression.
  return <components[props.storyType] story={props.story} />;
}
```

赋值给一个首字母大写的变量可以解决这个问题：
```js
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

## JSX 中的 Props
在 JSX 中有多种不同的方式定义 props。

### 表达式作为 Props
可以在 prop 中使用任何 JavaScript 表达式，以 `{}` 包裹。比如，在 JSX 中：
```js
<MyComponent foo={1 + 2 + 3 + 4} />
```
对于 `MyComponent`，`props.foo` 的值是 `10`，因为 `1 + 2 + 3 + 4` 会被计算。

`if` 语句和 `for` 循环在 JavaScript 中不是表达式，所以它们不能直接在 JSX 中使用。作为替代，可以把它们放到上下文（surrounding code）中。比如：
```js
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

可以在对应章节了解更多关于 [条件渲染] 和 [循环] 的内容。

### 字符串字面量
可以传递字符串字面量作为 prop。以下两个 JSX 表达式是等价的：
```js
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

如果传递了字符串字面量，它的值是未转义的 HTML（HTML-unescaped）。所以以下两个 JSX 表达式是等价的：
```js
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

这个行为通常无关紧要（usually not relevant）。这里只是为了完整性提及。

### Props 默认为 "True"
如果不给 prop 传值，那么它默认为 `true`。以下两个 JSX 表达式是等价的：
```js
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

通常我们不推荐这么用，因为会和 [ES6 的对象简写] `{foo}` 混淆，`{foo}` 是 `{foo: foo}` 的简写，而不是 `{foo: true}`。这种行为之所以存在是为了与 HTML 的行为统一。（ This behavior is just there so that it matches the behavior of HTML.）

### 展开属性
如果 props 是一个对象，在 JSX 中传递的时候，可以使用 `...` 作为扩展运算符（"spread" operator）传递整个 props 对象。以下两个组件是等价的：
```js
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```
可以挑出指定的 props 供组件使用（consume），同时使用扩展运算符传递其他所有的 props。

```js
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
};
```
在以上的例子中，`kind` prop 被安全使用而且没有传递到 `<button>` 元素中。所有其他的 props 通过 `...others` 对象传递，这让组件非常灵活。可以看到，传递了 `onClick` 和 `children` props。

展开属性很有用，但同时也容易把一些组件不关心的、无效的 HTML 属性等这些不必要的 props 传到组件。我们推荐少使用这种语法。


## JSX 中的 Children
对于同时包含起始标签（openning tag）和闭合标签的 JSX 表达式，标签之间的内容会作为一个特殊的 prop 传递：`props.children`。有几种不同的传递 children 的形式：

### 字符串字面量
可以在起始标签和闭合标签之间放一个字符串，这时候 `props.children` 就是这个字符串。这对于很多内置的 HTML 元素很有用。比如：
```js
<MyComponent>Hello world!</MyComponent>
```
这是有效的 JSX，`MyComponent` 中的 `props.children` 就直接是 `"Hello world!"`。HTML 是未转义的，所以通常可以像这样写 HTML 一样写 JSX：
```js
<div>This is valid HTML &amp; JSX at the same time.</div>
```

JSX 会删掉每行开头和最后的空白。JSX 还会删掉空行。挨着（adjacent to）标签的新行会被删掉；字符串字面量中的新行会被合并（condensed）成一个空格。所以以下这些渲染的结果是相同的：
```js
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX Children
可以提供更多 JSX 元素作为 children。这在展示嵌套组件时很有用：
```js
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```
可以混合不同类型的 children，可以同时使用字符串字面量和 JSX children。这是 JSX 另一个（another way）像 HTML 的地方，所以以下在 JSX 或 HTML 中都是有效的：
```js
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```

React 组件也可以返回一个包含多个元素的数组：
```js
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

### JavaScript 表达式作为 Children
任何 JavaScript 表达式都可以使用 `{}` 包裹后作为 children 传递。比如，以下表达式是等价的：

```js
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

这经常用来渲染任意数量的一系列 JSX 表达式。比如，这里渲染了一个 HTML 列表：

```js
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

JavaScript 表达式可以与其他类型的 children 混用。这经常用于替代字符串模板：

```js
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

### 函数作为 Children
通常，JSX 中插入的 JavaScript 表达式会计算成一个字符串，一个 React 元素，或两者混合的一个列表（a list of those things）。然而，`props.children` 就像其他 prop 一样可以传递任何类型的数据（any sort of data），而不仅仅是 React 知道如何渲染的类型。比如，可以在一个自定义组件中接收一个回调函数作为 `props.children`：

```js
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

传到自定义组件的 children 可以是任意类型，只要（as long as）组件渲染前把它们转换成 React 可以理解的东西就可以。这种用法并不常见，但如果想扩展 JSX 的功能，这样是可行的。

### 布尔类型，Null 和 Undefined 会被忽略
`false`，`null`，`undefined` 以及 `true` 是有效的 children，它们只是不渲染。以下 JSX 表达式会渲染相同的东西：

```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

这在有条件地渲染 React 元素时很有用。下面这个 JSX 只在 `showHeader` 是 `true` 的时候渲染 `<Header />`：

```js
<div>
  {showHeader && <Header />}
  <Content />
</div>
```

一个需要注意的地方（caveat）是一些 [“假”值]，比如数值 `0`，依然会被 React 渲染。比如，下面的代码可能不会像你想象的那样，因为档 `props.messages` 是空数组的时候，`0` 会渲染出来：

```js
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```

为了修复这个问题，需要确保 `&&` 之前的表达式一定是布尔值：

```js
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```

相反的，如果需要 `false`，`true`，`null` 或 `undefined` 这样的输出，必须先[转换成字符串]：

```js
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```

[Babel 在线编译器]: https://babeljs.io/repl/#?presets=react&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA
[条件渲染]: https://reactjs.org/docs/conditional-rendering.html
[循环]: https://reactjs.org/docs/lists-and-keys.html
[ES6 的对象简写]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015
[“假”值]: https://developer.mozilla.org/en-US/docs/Glossary/Falsy
[转换成字符串]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#String_conversion