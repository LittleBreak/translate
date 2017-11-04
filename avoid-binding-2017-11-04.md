# React 模式：提取子组件避免绑定

在React中存在一个常见的情景，你正在遍历数组，并且你需要每一项绑定点击时间和传递一些相关的数据。
这里有一个例子，我正在遍历一个用户信息的列表，把`userId` 传递到`deleteUser`函数中去删除指定的用户。

```
import React from 'react';

class App extends React.Component {
  constructor() {
    this.state = {
      users: [
        { id: 1, name: 'Cory' }, 
        { id: 2, name: 'Meg' }
      ]
    };
  }
  
  deleteUser = id => {
    this.setState(prevState => {
      return { users: prevState.users.filter( user => user.id !== id)}
    })
  }

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
        { 
          this.state.users.map( user => {
            return (
              <li key={user.id}>
                <input 
                  type="button" 
                  value="Delete" 
                  onClick={() => this.deleteUser(user.id)} 
                /> 
                {user.name}
              </li>
            )
          })
        }
        </ul>
      </div>
    );
  }
}

export default App;
```

## 哪里存在问在问题呢？

在点击事件中，我用了箭头函数。**这意味着，每一次render函数被执行，都会被重新分配一个新的箭头函数。** 在大多数情况下，并没有什么大不了的。但是如果`render` 中存在一些子组件，因为`render` 函数中分配了新的函数，导致这些子组件会重新渲染即使相关数据没有发生变化。

避免在`render` 函数中声明箭头函数或绑定以获得最佳性能。我的团队使用这种[eslint rule](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)规则帮助提醒我们注意到这个问题。

## 解决方式
