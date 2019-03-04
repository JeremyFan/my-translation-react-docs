# 不包含 JSX 的 React

对于 React 来说，JSX 并不是必须的。如果不想在构建环境中支持编译，使用不包含 JSX 的 React 是非常方便的。

每个 JSX 元素都只是调用 `React.createElement(component, props, ...children)` 方法的语法糖。所以，任何 JSX 能做的事纯 JavaScript 也能做。

比如这些 JSX 代码：
```js
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.toWhat}</div>;
  }
}
ReactDOM.render(
  <Hello toWhat="World" />,
  document.getElementById('root')
);
```

可以被编译成下面这段未使用 JSX 的代码：
```js
class Hello extends React.Component {
  render() {
    return React.createElement('div', null, `Hello ${this.props.toWhat}`);
  }
}
ReactDOM.render(
  React.createElement(Hello, {toWhat: 'World'}, null),
  document.getElementById('root')
);
```

如果很好奇想要看到更多 JSX 如何编译成 JavaScript 的例子，可以试试 [Babel 在线编译器]。

组件可以是：一个字符串、一个 `React.Component` 的子类、一个作为无状态组件的普通函数。

如果觉得写太多 `React.createElement` 很累，一个通用的模式是使用缩写：

```js
const e = React.createElement;
ReactDOM.render(
  e('div', null, 'Hello World'),
  document.getElementById('root')
);
```

如果像这样使用 `React.createElement` 的简写形式，在不使用 JSX 的情况下使用 React 也几乎同样方便。

另一种选择是使用提供更简洁语法（terser syntax）的社区项目，比如 [react-hyperscript] 和 [hyperscript-helpers]。



[Babel 在线编译器]: https://babeljs.io/repl/#?presets=react&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA
[react-hyperscript]: https://github.com/mlmorg/react-hyperscript
[hyperscript-helpers]: https://github.com/ohanhi/hyperscript-helpers