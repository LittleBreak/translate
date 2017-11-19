# React16 protal功能的炫酷实现

React16已经发布，其中最令人感兴趣的功能就是`portal`。

`portal` 可以让你在父组件外面渲染一些`React` 的`DOM`节点。[React文档](https://reactjs.org/docs/portals.html) 使用了一个弹层的例子做了很好的解释。在`tooltip` 下同样也有很好的实现。（[demo](https://codepen.io/davidgilbertson/pen/ooXVyw)）

但是这些例子都不是很让人感兴趣，让我们来做一些奇怪的东西...

既然`portal` 功能可以把一个元素添加到另外的元素之下，你不需要仅限制在当前的文档对象之下。你可以把元素附加到不同的文档对象下的`body` 元素之下，也许这个文档对象是一个完全不同的 窗口。

在下方，我有一个简单的页面（左侧），页面上有一个计数器和一个深红色的按钮，还有一个窗口是同一个`React` 应用。

在上方图片你所能看到的所有东西只是下方的这一个组件。
```
class App extends React.PureComponent {
  constructor(props) {
    super(props);
    
    this.state = {
      counter: 0,
      showWindowPortal: false,
    };
    
    this.toggleWindowPortal = this.toggleWindowPortal.bind(this);
  }

  componentDidMount() {
    window.setInterval(() => {
      this.setState(state => ({
        ...state,
        counter: state.counter + 1,
      }));
    }, 1000);
  }
  
  toggleWindowPortal() {
    this.setState(state => ({
      ...state,
      showWindowPortal: !state.showWindowPortal,
    }));
  }
  
  render() {
    return (
      <div>
        <h1>Counter: {this.state.counter}</h1>
        
        <button onClick={this.toggleWindowPortal}>
          {this.state.showWindowPortal ? 'Close the' : 'Open a'} Portal
        </button>
        
        {this.state.showWindowPortal && (
          <MyWindowPortal>
            <h1>Counter in a portal: {this.state.counter}</h1>
            <p>Even though I render in a different window, I share state!</p>
            
            <button onClick={() => this.setState({ showWindowPortal: false })} >
              Close me!
            </button>
          </MyWindowPortal>
        )}
      </div>
    );
  }
}
```
你会发现`<MyWindowPortal>` 组件有些特殊，组件内的所有元素都会在另一个窗口中渲染出来。

更明确的说，`<MyWindowPortal>` 组件需要做两件事

1. 当组件渲染时打开一个新的浏览器窗口
2.  创建一个传送门把`props.children` 附加到新窗口下的`body` 下。

是不是很酷！！！

下方的代码就是上边`body` 下展示的组件。在11行`ReactDom.createPortal` zhezhong`React16` 的新写法才是实现这种魔法的关键。

```
class MyWindowPortal extends React.PureComponent {
  constructor(props) {
    super(props);
    // STEP 1: create a container <div>
    this.containerEl = document.createElement('div');
    this.externalWindow = null;
  }
  
  render() {
    // STEP 2: append props.children to the container <div> that isn't mounted anywhere yet
    return ReactDOM.createPortal(this.props.children, this.containerEl);
  }

  componentDidMount() {
    // STEP 3: open a new browser window and store a reference to it
    this.externalWindow = window.open('', '', 'width=600,height=400,left=200,top=200');

    // STEP 4: append the container <div> (that has props.children appended to it) to the body of the new window
    this.externalWindow.document.body.appendChild(this.containerEl);
  }

  componentWillUnmount() {
    // STEP 5: This will fire when this.state.showWindowPortal in the parent component becomes false
    // So we tidy up by closing the window
    this.externalWindow.close();
  }
}
```

这样合理吗？组件没有返回一些dom结构，而是在其他地方展示出来。

换种思路来考虑就像是这样：正常情况下，一个父组件给他的子组件说，(｡･∀･)ﾉﾞ嗨，渲染一些`DOM`,并把结果附加到我身上，结果这个子组件就按父组件所说的来实现。但是在这种情况下，这个任性的子组件说，不，我要把东西渲染到其他的窗口上，并且写个博客发布上去。

也许你在考虑另外一件事：如果一个空白窗口没有样式那么把一些`DOM` 注入到这下面有什么好处呢？如果这个网站是`craigslist`或者`wikipedia`，可能没有人会注意到这件事，但是你的网站要漂亮多了。

同志们，好消息！

起初我希望有一种简单的方式可以把样式拷贝到一个新的窗口上。然后我想起我的生活不仅仅是一系列毫无意义的任务来填满这些时间和时间，它们的唯一目的是让我远离内心深处的咆哮的空虚。

```
function copyStyles(sourceDoc, targetDoc) {
  Array.from(sourceDoc.styleSheets).forEach(styleSheet => {
    if (styleSheet.cssRules) { // for <style> elements
      const newStyleEl = sourceDoc.createElement('style');

      Array.from(styleSheet.cssRules).forEach(cssRule => {
        // write the text of each rule into the body of the style element
        newStyleEl.appendChild(sourceDoc.createTextNode(cssRule.cssText));
      });

      targetDoc.head.appendChild(newStyleEl);
    } else if (styleSheet.href) { // for <link> elements loading CSS from a URL
      const newLinkEl = sourceDoc.createElement('link');

      newLinkEl.rel = 'stylesheet';
      newLinkEl.href = styleSheet.href;
      targetDoc.head.appendChild(newLinkEl);
    }
  });
}
```
[实例程序](https://codepen.io/davidgilbertson/pen/xPVMqp)

## 原文链接

>[Using a React 16 Portal to do something cool](https://hackernoon.com/using-a-react-16-portal-to-do-something-cool-2a2d627b0202)