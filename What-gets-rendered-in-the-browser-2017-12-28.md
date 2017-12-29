# React在浏览器中到底渲染的是什么？

你也许不会喜欢这个答案，不巧的是，结果有点复杂。

> `element` 和组件这两个词难道不是一个意思吗？

```
class HelloMessage extends React.Component {
  render() {
    return (
      <div>
        Hello {this.props.name}
      <div/>
    )
  }
}

ReactDOM.render(
  <HelloMessage name="taylor" />,
  mountNode
)
```
从技术层面来讲，`ReactDOM` 在`DOM`中渲染的既不是*react组件* 也不是*react元素*。 **它渲染的是由组件的实例所支持的`DOM元素`**。这种说法是适用与由`class` 定义的组件。对函数式组件来说，`ReactDOM` 渲染的仅仅是`DOM`元素。函数式组件不能被实例化（class式的组件可以通过`this`访问到），因此当你使用一个函数式的组件时，`ReactDOM` 渲染的是由函数式组件返回的元素所生成的`DOM` 元素。

你需要了解的一点是`React元素` 和`DOM元素` 是不同的。`React 元素` 只是`HTML元素` 或 `React组件` 两者的一种描述。

一个好的面试问题可能是这样的

> 当你在`jsx`中使用像`<MyComponent/>`的组件时，它是一个组件，元素或者是实例。

答案是它是一个**元素** 但并不是一个`DOM` 元素，而是一个react元素。这里的线索是所有的`JSX` 标签都需要通过执行`React.createElement` 进行翻译。牢记住这个方法。**CREATE ELEMENT**

你也许会发现组件，元素，实例这些词可能混杂在react指引和教程里边。对这件事我表示抱歉，但是我认为一个`react` 的初学者需要理解这其中的重要区别。[react博客](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html) 中有相关的文章，但我认为它对初学者来说技术性太强了。

