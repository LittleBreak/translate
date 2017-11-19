# 如何在React中构建组件

编程可以说是相当复杂的任务，尤其构建清晰的代码十分困难。我们需要关注很多的元素，变量的命名，函数作用域，处理异常，确保安全，监控性能问题等等。**在编程中如果仍要说出最难解决的问题的话，我觉得是编写低耦合高内聚的组件。** 这和我们讨论面向对象或者是函数式编程没有什么关联。构建系统是最难的事情，并且它会影响到整个项目。在设计软件架构上需要花费数年的时间才可以精通（也许一个人永远不会掌握它，在这样一个快速发展的行业中精通者总是会领先一步，总会有一些方法去提高设计）。

我十分享受`react` 上的工作，并且我认为最大的优点是`React` 特别简易。简易并不同于简单[simple&easy](https://www.infoq.com/presentations/Simple-Made-Easy). 我真的认为`React` 是简易的。当然你需要花费一些时间去了解它。但是当你理解一些核心的理念后，剩下的就变得顺其自然了。最难的一部分如下。

## 耦合 & 内聚
这里有个指标或多或少可用来描述在代码中去改变行为是多么困难。耦合和内聚的概念源于面向对象编程并且适用与一些class。我们使用他们到`React` 组件中，既然规则同样适用。

耦合是元素之间的联系或是一种依赖。如果盖面一个元素*需要* 修改另外的元素，我们可以说是紧耦合。如果元素是低耦合的,改变一个元素不意味着另外的元素需要做更改。举个例子，我们来看一下展示银行转账金额。如果显示知道比率是怎么计算的，那么无论何时内部转账结构发生变化，显示代码也需要更新。如果我们设计一个系统是低耦合的，依赖一个元素的接口，那么转账的变化就不会导致展示层的改变。低耦合的组件会更易管理和维护。

内聚告诉我们如果元素的职责构成一个组件。该指标与单责原则相联系或是`unix` 原则：`do one thing and di it well`. 如果账户余额的格式化程序也计算利率和检查允许显示的历史，那么它有很多的职责并且并不相关。也许这里应该由不同的组件来控制权限或是计算利率。另一方面，如果有不同的组件，一个用来负责整数部分，一个用来负责小数部分，一个用来负责货币类型，那么每次程序想要展示余额，需要找到所有的元素。面对的挑战是创建高内聚的组件。

## 构建组件

我们可以用许多中方式来构建组件。我们希望组件是可以复用的，但只能达到合理的程度。我们希望构建可以用来构建更大概念的小组件。理想情况下，我们希望构建低耦合高内聚的组件，这样我们的系统会更易维护和发展。在`React` 组件中属性可以作为函数的参数所以可以用来构建无状态的组件。**我们如何在组件中定义属性，定义一个组件如何去复用。**

我们使用费用管理并且我们来分析费用明细格式化。让我们假设费用模型如下
```
type Expense {
  description: string
  category: string
  amount: number
  doneAt: moment
}
```

这里有几种可能去模拟费用明细格式化组件

* 无属性控制
*  传递费用相关对象
*  只传递需要用的属性
*  传递属性的数组
*  传递格式作为children

我们来讨论上面的每一项运用过程中的优缺点。**牢记没有最好的只有最适合的，所有模式都依赖与系统。**

## 无属性控制

最简单的解决方式并且这种方式通常为最开始的时候来构建一个组件用硬编码的数据。

```
const ExpenseDetails = () => (
  <div className='expense-details'>
     <div>Category: <span>Food</span></div>
     <div>Description: <span>Lunch</span></div>
     <div>Amount: <span>10.15</span></div>
     <div>Date: <span>2017-10-12</span></div>
  </div>
)
```

不传递任何属性，当然组件将只适用于一个场景切没有给我们提供任何灵活性。当然，在这个花费明细的组件中我们为你可以看到在最开始的时候组件需要接受哪一些参数。然而，有些场景下不接受属性是一个好的解决方式。首先，我们可以使用像logo,公司信息，徽章这些不接收属性的静态组件。

```
const Logo = () => (
  <div className='logo'>
   <img src='/logo.png' alt='DayOne logo'/>
  </div>
)
```


## 传递费用的对象

在花费明细明确的情况下，我们需要传递数据到组件中去。首先我们先来看看传递费用对象的情况。
```
const ExpenseDetails = ({ expense }) => (
  <div className='expense-details'>
     <div>Category: <span>{expense.category}</span></div>
     <div>Description: <span>{expense.description}</span></div>
     <div>Amount: <span>{expense.amount}</span></div>
     <div>Date: <span>{expense.doneAt}</span></div>
  </div>
)
```

传递费用对象到花费明细组件中感觉十分合适。花费明细的格式化是高内聚的，它展示乐费用的数据。无论是么时候我们想修改展示的格式，只需要在一个地方做出修改。同时改变费用明细格式化不引入任何副作用到消费对象本身。

组件紧密的和费用对象绑定在一起，这样会有什么问题吗。当然没有，但是我们必须意识到这会如何影响到我们的系统。把费用对象作为属性传递，结果导致花费明细组件依赖以费用对象的内部结构。只要我们修改乐费用对象的内部结构，我们就需要修改组件。当然，我们只需要需要一个地方。

这种设计会如果影响将来的改变和拓展呢？如果我们像添加，修改或者移除一个字段,我们只需要修改一个组件。如果我们想添加一个不同的日期的格式化呢，我们就需要为了日期的格式化添加另一个属性。
```
const ExpenseDetails = ({ expense, dateFormat }) => (
  <div className='expense-details'>
     <div>Category: <span>{expense.category}</span></div>
     <div>Description: <span>{expense.description}</span></div>
     <div>Amount: <span>{expense.amount}</span></div>
     <div>Date: <span>{expense.doneAt.format(dateFormat)}</span></div>
  </div>
)
```

我们通过添加额外的属性使组件更加灵活。只要这里只有少许的选择，一切ok。在系统不断增长，为了不同的应用场景下我们需要很多的属性来进行控制。
```
const ExpenseDetails = ({ expense, dateFormat, withCurrency, currencyFormat, isOverdue, isPaid ... })
```

添加属性使组件变得更加的复用，但是这同样是一个信号，这个组件在完成各种不同的职责。这种规律同样适用于函数。我们可以创建一个接受很多参数的函数，但是只要参数的数量超出3-4个，这个函数就开始负责很多的事情。也许是时候把函数分离成更小的函数。

随着组件属性的增长，我们可以决定是否把组件分割成更容易定义的组件，就像`OverdueExpenseDetails, PaidExpenseDetails` 等等。

## 仅传递需要的属性

为了减少组件和费用对象的关联，我们可以仅传递需要的属性。

```
const ExpenseDetails = ({ category, description, amount, date }) => (
  <div className='expense-details'>
     <div>Category: <span>{category}</span></div>
     <div>Description: <span>{description}</span></div>
     <div>Amount: <span>{amount}</span></div>
     <div>Date: <span>{date}</span></div>
  </div>
)
```

我们分别传递每个属性，所以我们将责任转移到使用组件的人身上。如果费用对象的内部结构改变乐的话，它并不影响花费明细格式化组件本身，但是或许会影响到所有用到这个组件的地方，因为传递的属性必须进行修改。当作为单独的属性进行传递时，组件将会更加的**抽象**。

分离传递属性又会怎么影响未来额设计呢？添加/更新/删除一些属性变得不是很容易。当我们想添加一个字段时，我们不仅需要修改花费明细组件的内部实现还需要修改每一个用到组件的地方。

```
const ExpenseDetails = ({ category, description, amount, date, account, comment, case ... }) => ( ... )
```
另一方面，支持多种日期的格式化几乎是在组件外完成的，既然我们可以把日期作为属性传递进来，我们同样可以传递格式化后的日期。

```
<ExpenseDetails 
  category={expense.category} 
  description={expense.description}
  amount={expense.amount}
  date={expense.doneAt.format('YYYY-MM-DD')}
/>
```
决定如何展示特定的字段掌握在使用组件的人手中。这不再是花费明细组件实现的情况。

## 传递属性的数组或映射

组件将会变得更加抽象化，我们可以传递属性的映射。
```
const ExpenseDetails = ({ expense }) => (
  <div class='expense-details'>
  {
    _.reduce(expense, (acc, value, key) => {
      acc.push(<div>{key}<span>{value}</span></div>)
    }, [])
  }
  </div>
)
```
使用组件的人将控制花费明细组件的格式。传递到组件中的对象必须进行适当的格式化。
```
const expense = {
  "Category": "Food",
  "Description": "Lunch",
  "Amount": 10.15,
  "Date": 2017-10-12
}
```
这种解决方式存在很多问题。我们稍微控制了组件会如何展示。`reduce` 的规则并没有被详细说明，所以我们需要添加一些规则。我们可以传递一个有相关对象的数组替换掉映射，来克服这种问题，但是还是存在一些缺点。

传递映射或数组作为属性和花费对象没有关联在一起，同时也完全没有内聚相关。添加或删除属性将只涉及到修改属性，但是在组件本身将失去对格式化的控制。如果我们想修改类别的格式化，在这个方案中时不能解决的。（*精确的说，总有办法去改变一些东西，比如传递另一个携带这格式化配置信息的属性，当然这种解决方案不再清晰和直观。* ）

## 把格式化数据作为child传递

```
const ExpenseDetails = ({ children }) => (
  <div class='expense-details'>
    { children }
  </div>
)
```
在这种情况下，花费明细组件仅仅只是提供结构和样式的容器。使用组件去展示详情必须提供所有的数据。
```

<ExpenseDetails>
  <div>Category: <span>{expense.category}</span></div>
  <div>Description: <span>{expense.description}</span></div>
  <div>Amount: <span>{expense.amount}</span></div>
  <div>Date: <span>{expense.doneAt}</span></div>
</ExpenseDetails>
```

既然我们需要做很多重复性的工作，或许这种解决方式对花费明细组件来说不是一种好的解决方案。但仍然有很大的灵活性并且有了很多不同的格式化展示的可能性。添加，删除，更新字段只涉及到使用组件的更改。日期格式化同样如此。我们失去了内聚，但是这个代价是我们必须要付出的。

## 应用场景才是王道

正如你所看到的，我们在交换不同的长处和可能性。哪一种时最好的呢？这取决于

* 项目本身
* 项目的阶段
* 组件——我们是想要更明确的组件
* 个人偏好
* 需求——组件是修改的更频繁好事使用的更频繁

**这里没有唯一的最好的解决方案，一种方式不会适合所有的应用场景.** 我们如何构建组件将会极大的影响我们如何维护和扩展系统。所有的一切都取决于应用场景。值得庆幸的时有很多解决方式供我们选择。组件化是一种很好的抽象去构建或大或小的系统。这只是选择正确解决方案的一个例子。

## 原文链接

>[How to structure components in React?](https://reallifeprogramming.com/how-to-structure-components-in-react-54fc43e71546)