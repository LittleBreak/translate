# React 模式：提取子组件避h函数绑定

在React中存在一个常见的情景，你正在遍历数组，需要每一项绑定点击事件和传递一些相关的数据。

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

在点击事件中，我用了箭头函数。**这意味着，每一次render函数被执行，都会被重新分配一个箭头函数。** 在大多数情况下，并没有什么大不了的。但是如果`render` 中存在一些子组件，因为`render` 函数中分配了新的函数，这些子组件即使相关数据没有发生变化也会重新渲染。

避免在`render` 函数中声明箭头函数或绑定以获得最佳性能。我的团队使用这种[eslint rule](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)规则帮助提醒我们注意到这个问题。

## 解决方式
所以我们如何避免在`render` 中避免绑定和箭头函数的使用呢，其中一种解决方式是抽离子组件。下面，我抽离这个列表到`UserListItem.js`

```
import React from 'react';
import PropTypes from 'prop-types';

class UserListItem extends React.Component {
  onDeleteClick = () => {
    // No bind needed since we can compose 
    // the relevant data for this item here
    this.props.onClick(this.props.user.id);
  }

  // No arrow func in render! 
  render() {
    return (
      <li>
        <input 
          type="button" 
          value="Delete" 
          onClick={this.onDeleteClick} 
        /> 
        {this.props.user.name}
      </li>
    );
  }
}

UserListItem.propTypes = {
  user: PropTypes.object.isRequired,
  onClick: PropTypes.func.isRequired
};

export default UserListItem;
```
这样的话，父组件的渲染函数将变得简单，并且不需要包含箭头函数。它在`renderUserListItem` 这个新函数中通过属性将相关的上下文传递到列表的每一项中。

```
import React from 'react';
import { render } from 'react-dom';
import UserListItem from './UserListItem';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [{ id: 1, name: 'Cory' }, { id: 2, name: 'Sherry' }],
    };
  }

  deleteUser = id => {
    this.setState(prevState => {
      return { users: prevState.users.filter(user => user.id !== id) };
    });
  };

  renderUserListItem = user =>
    <UserListItem key={user.id} user={user} onClick={this.deleteUser} />;

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
          {this.state.users.map(this.renderUserListItem)}
        </ul>
      </div>
    );
  }
}

render(<App />, document.getElementById('root'));
```

注意一点，我们调用一个在`render` 函数外19行声明的新函数来替换在`render` 中遍历时用的箭头函数。在每次渲染中将不再会重新分配函数。


## 利与弊

这种模式通过消除重复的函数分配来提高性能，所以在以下情况中运用到你的组件中是非常有用的。
1. `render` 函数频繁的执行
2. 渲染子节点成本昂贵

需要承认的一点是，我所建议的抽离子组件的方式需要额外的工作。需要更多的组合，更多的代码。所以如果你没有性能的上的问题，可以说是过早的性能优化。
