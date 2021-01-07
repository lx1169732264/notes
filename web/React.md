

| on[Event]     | 事件的监听         |
| ------------- | ------------------ |
| handle[Event] | 处理事件的监听方法 |
|               |                    |
|               |                    |





**JSX**，JS的语法扩展。可以很好地描述 UI 应该呈现出它应有交互的本质形式

 渲染逻辑本质上与其他 UI 逻辑内在耦合, 比如，在 UI 中需要绑定处理事件、在某些时刻状态发生变化时需要通知到 UI，以及需要在 UI 中展示准备好的数据 

 React  将 *标记与逻辑*  共同存放在称之为“组件”的松散耦合单元之中 







**根DOM 节点**	根节点下的所有内容都将由 React DOM 管理

```react
const element = <h1>Hello, world</h1>;
//将元素渲染到根节点上
ReactDOM.render(element, document.getElementById('root'));
```



**React 元素**	元素一旦创建，无法修改子元素或者属性 

元素更新的唯一方式是创建新的元素,并ReactDOM.render()



# 组件



允许将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思,能接受任意的入参（props），并返回用于描述页面展示内容的 React 元素 

**组件名必须以大写开头** , React 会将以小写字母开头的组件视为原生 DOM 标签



**函数组件**	 本质上就是 JS 函数 ,接收入参并返回一个React 元素

```react
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
==
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```



**渲染组件**

```react
//会在页面上渲染 “Hello, Sara”
function Welcome(props) { return <h1>Hello, {props.name}</h1>; }

//对于用户自定义的组件,将JSX所接收的属性（attributes）以及子组件（children）转换为单个对象传递给组件，这个对象被称之为 “props”
const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,document.getElementById('root')
);
```



**组合组件**

```react
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
//App组件可以多次渲染Welcome组件
function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```





**提取组件**

```react
//该组件接收 author（对象），text（字符串）以及 date（日期）作为props
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl} alt={props.author.name}
        />
        <div className="UserInfo-name">  {props.author.name} </div>
      </div>
      <div className="Comment-text">  {props.text}	</div>
      <div className="Comment-date">  {formatDate(props.date)}	</div>
    </div>
  );
}

//首先提取 Avatar 组件	->
function Avatar(props) {
  return (
    //
    <img className="Avatar" src={props.user.avatarUrl} alt={props.user.name} />
  );
} 
```



同时获取多个子组件的数据/子组件之间的通信 从技术上可以实现,但代码难以维护,可以将子组件的state提升至父组件,之后父组件通过props将状态数据传递到子组件



**受控组件**	子组件在触发事件时从父组件接收值,并通知父组件

state成为组件唯一的数据源, 渲染表单的 React 组件还控制着用户输入过程中表单发生的操作

==受控组件上指定了value,将阻止用户更改输入== ( `value` 设置为 `undefined` 或 `null` 时不再受控)









**函数组件**	组件只包含render(),并不需要state或state由父组件管理,就可以使用函数组件

 接收 `props` 作为参数，然后返回需要渲染的元素 

```react
class Square extends React.Component {
  render() {
    return (
      <button className="square" 
        onClick={()=>this.props.onClick()}>
        {this.props.value}
      </button>
    );
  }
}
->
function Square(props){
  return(
    <button className="square" onClick={props.onClick}> //此处去掉了this和括号
      {props.value}
    </button>
  );
}
```





## 生命周期



**挂载**		组件实例被创建并插入 DOM 中

- constructor()
- static getDerivedStateFromProps()
- render()
- componentDidMount()











## input



```react
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.name === 'isGoing' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          参与:
          <input
            //给input增加name属性
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          来宾人数:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```



对于文件的input标签	type="file" 它的value是只读的,为**非受控组件**

```react
<input type="file" />
```

 



## textarea



`this.state.value` 初始化于构造函数中，因此文本区域默认有初值。 

```react
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {      value: '请撰写一篇关于你喜欢的 DOM 元素的文章.'    };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {    this.setState({value: event.target.value});  }
  handleSubmit(event) {
    alert('提交的文章: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          <textarea value={this.state.value} onChange={this.handleChange} />        							</label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```











## Select



```react
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    //多选	this.state = {value: ['coconut', 'lime'],flag:true,};
    this.state = {value: 'coconut'};
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {    this.setState({value: event.target.value});  }
  handleSubmit(event) {
    alert('你喜欢的风味是: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
					<!--多选multiple={this.state.flag}-->
          <select value={this.state.value} onChange={this.handleChange}>            									<option value="grapefruit">葡萄柚</option>
            <option value="lime">酸橙</option>
            <option value="coconut">椰子</option>
            <option value="mango">芒果</option>
          </select>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```







# 状态提升



 多个组件需要反映相同的变化数据 ,可以将共享状态提升到最近的共同父组件中去 



```react
//BoilingVerdict组件接受 celsius 温度作为prop,并对prop进行条件判断
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;  
  }
  return <p>The water would not boil.</p>;
}

//Calculator组件渲染 input,并将input的value保存在this.state.temperature中
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    return (
      <fieldset>
        <input
          value={temperature}
          //根据当前用户输入值进行重新渲染,渲染受React控制
          onChange={this.handleChange} />
        <BoilingVerdict  celsius={parseFloat(temperature)} />
      </fieldset>
    );
  }
}
```



然而这种情况只有1个input,为了支持多个input,需要将其抽离,



```react
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
};

function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}

function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}

function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  //1.调用 TemperatureInput 组件的 handleChange()
  handleChange(e) {
    //2.调用Calculator传入的props 中的onTemperatureChange()
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}

class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};
  }

  //4.通过setState改变值,进而render()重新渲染,2个输入框的值都将被重新计算
  handleCelsiusChange(temperature) {  this.setState({scale: 'c', temperature});}
  handleFahrenheitChange(temperature) {  this.setState({scale: 'f', temperature});}

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        //5.render()过程中也调用了2个子组件的render(),子组件也被重新渲染
        <TemperatureInput
          scale="c"
          temperature={celsius}
          //3.onTemperatureChange()具体可以为2种实现
          onTemperatureChange={this.handleCelsiusChange} />
        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />
        //6.重新计算水是否沸腾
        <BoilingVerdict
          celsius={parseFloat(celsius)} />
      </div>
    );
  }
}

ReactDOM.render(
  <Calculator />,
  document.getElementById('root')
);
```



state的数据流应当是自顶向下的,而不是在各个组件中进行同步state

虽然提升 state 方式比双向绑定方式需要编写更多的“样板”代码，但带来的好处是，排查和隔离 bug 所需的工作量将会变少。由于“存在”于组件中的任何 state，仅有组件自己能够修改它，缩小 bug 排查范围





## 组合



 用 `children` prop 将子组件传递到渲染结果 	(不推荐用继承)



```react
//当需要对WelcomeDialog进行外层的包装时
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1>	Welcome	</h1>
    </FancyBorder>
  );
}

function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}	<!--组合,将WelcomeDialog的所有内容作为{props.children}传入-->
    </div>
  );
}
```



对于多个组件的组合

```react
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <!--预留插槽-->
      <div>  {props.left}     </div>
      <div>  {props.right}    </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={  <Contacts />   }
      right={  <Chat />      }
    />
  );
}
```















 ## 事件



使用 JSX 语法时需要传入函数，而不是字符串 

```react
<button onclick="activateLasers()">  Activate Lasers	</button>
->
<button onClick={activateLasers}>  Activate Lasers	</button>
```



```react
preventDefault()	//终止事件,无法用return
```



 在 JavaScript 中，class 的方法默认不会绑定 `this`。如果忘记绑定 `this.handleClick` 并把它传入了 `onClick`，当调用时 `this` 的值为 `undefined` 

```react
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    // 为了在回调中使用 `this`，需要绑定this
    this.handleClick = this.handleClick.bind(this);  
  }

  handleClick() {}

  render() {
    return (
      <button onClick={this.handleClick}> </button>
    );
  }
}
//如果要去掉绑定this
->
  handleClick = () => {
  console.log('this is:', this);
}
//或者
->
  return (
      <button onClick={() => this.handleClick()}></button>
    );
```



### 向事件处理程序传递参数

```react
//事件对象e
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>	//箭头函数需要显示传递e
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>	//普通函数通过bind
```





## 条件渲染



~~if 不使用~~

```react
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;
    let button;
    //决定了button绑定哪个点击事件函数
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;    
    } else {      
      button = <LoginButton onClick={this.handleLoginClick} />;    
    }
    return (
      <div><Greeting isLoggedIn={isLoggedIn} /> {button}  </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```



&&

```react
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      {unreadMessages.length > 0 &&  <h2>You have unread messages.</h2>}    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```

在 JavaScript 中

`true && expression` 返回 `expression`	使得组件被渲染

`false && expression` 返回 `false` 



?:	三目

当条件过于复杂时,需要考虑提取组件

```react
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.    </div>
  );
}
```



阻止渲染

```react
function WarningBanner(props) {
  //直接返回null
  if (!props.warn) {    return null;  }
  return (
    <div className="warning">
      Warning!
    </div>
  );
}
```





## 列表&key



通过map()将数组转化为元素

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>  <li>{number}</li>);
```





**key的定义应当在就近的数组上下文**

```react
function ListItem(props) {
  //此处无需指定key  
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  // key 应该在数组的上下文中被指定
  const listItems = numbers.map((number) => <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

并且key只需要在兄弟节点间唯一,而不必全局唯一



由于JSX 允许在大括号中嵌入任何表达式,可以优化为

```react
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) => 
                   <ListItem key={number.toString()}      
                     value={number} />      )}   
    </ul>
  );
}
```































# 井字棋



```react
function Square(props){
  return(
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}

class Board extends React.Component {
  renderSquare(i) {
    return (
      <Square value={this.props.squares[i]}
        onClick={() => this.props.onClick(i)}
        />
    );
  }

  render() {
    return (
      <div>
        <div className="board-row">
          {this.renderSquare(0)}{this.renderSquare(1)}{this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}{this.renderSquare(4)}{this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}{this.renderSquare(7)}{this.renderSquare(8)}
        </div>
      </div>
    );
  }
}

class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      //存放历史记录,从Board提升至Game
      history:[{squares: Array(9).fill(null),}],
      flag:true,
      stepNumber: 0,
    };
  }


  handleClick(i){
    //此处无法使用this.state.history,如果发生回退,再走一步时会导致“未来”历史记录不正确
    // const history = this.state.history;
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i]=this.state.flag?'X':'O';
    this.setState({
      //concat()不会修改原数组
      history: history.concat([{
        squares: squares
      }]),
      stepNumber: history.length,
      flag: !this.state.flag,
    });
  }

  //回退  重置stepNumber与flag
  jumpTo(step) {
    this.setState({
      stepNumber: step,
      flag: (step % 2) === 0,
    });
  }

  render() {
    const history=this.state.history;
    //根据当前stepNumber展示数据,避免回退导致的异常
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step,move)=>{
      const desc =move? 'Go to move #' + move :'Go to game start';
      return (
        <li key={move}>
          <button onClick={() => this.jumpTo(move)}>{desc}</button>
        </li>
      );
    });


    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Next player: " + (this.state.xIsNext ? "X" : "O");
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board squares={current.squares}
            onClick={i => this.handleClick(i)} />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
        </div>
      </div>
    );
  }
}

// ========================================
ReactDOM.render(
  <Game />,
  document.getElementById('root')
);

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```



















[^Warning: Each child in an array or iterator should have a unique “key” prop. Check the render method of “XXX”.]: 需要给每个元素key,以便区别不同的元素以及父子元素

当重新渲染时,会根据每个元素的key检索出上次渲染时的key

1. key不存在,创建新组件
2. key缺少,销毁组件
3. key彼岸花,销毁再用新的state重建



 `key` 是 React 中特殊保留属性（还有一个是 `ref`，拥有更高级的特性）

当 React 元素被创建时，会提取出 `key` 并存储在返回的元素上

无法通过 `this.props.key` 来获取 `key`。React 会通过 `key` 来自动判断哪些组件需要更新。组件是不能访问 `key`  

**key只需要在同级元素之前保证唯一**















# 高阶组件HOC



 **参数为组件，返回值为新组件的函数** , 组件是将 props 转换为 UI，而高阶组件是将组件转换为另一个组件 









## memo



```
const MyComponent = React.memo(function MyComponent(props) {
  /* 使用 props 渲染 */
});
```



适用于 相同 props 时渲染得到相同的组件,通过memo 记忆组件渲染结果,提高渲染性能

memo仅检查 props 变更。如果函数组件被 `React.memo` 包裹，且其实现中拥有 useState / useContext 的 Hook，当 context 发生变化时，仍会重新渲染 

 默认做浅层对比，可以将自定义比较函数通过第二个参数传入来实现 

































