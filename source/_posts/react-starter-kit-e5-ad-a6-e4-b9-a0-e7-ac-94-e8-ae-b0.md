---
title: React Starter Kit 学习笔记
url: 173.html
id: 173
categories:
  - 前端
date: 2018-03-24 00:02:44
tags:
---

根据[http://reactjs.cn/react/docs/...](http://reactjs.cn/react/docs/getting-started.html)页面中Starter Kit 15.3.1中的例子汇总修改而成。

* * *

HTML代码

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>React test</title>
    <link rel="stylesheet" href="react_transition.css">
    <script src="lib/react-with-addons.js"></script>
    <script src="lib/react-dom.js"></script>
    <script src="lib/browser.min.js"></script>
</head>

<body>
    <div id="container1"></div>
    <div id="container2"></div>
    <div id="container3"></div>
    <div id="container4"></div>
    <div id="container5"></div>
    <div id="container6"></div>
    <div id="container7"></div>
    <div id="container8"></div>
    <div id="container9"></div>
    <div id="container10"></div>
    <script src="react_test.js" type="text/babel"></script>
</body>

</html>

* * *

JS代码

const container1 = document.getElementById('container1')
const container2 = document.getElementById('container2')
const container3 = document.getElementById('container3')
const container4 = document.getElementById('container4')
const container5 = document.getElementById('container5')
const container6 = document.getElementById('container6')


//1 'Hello' + name
function Welcome(props) {
    return <h1 > Hello, { props.name } < /h1>;
}

const element = < Welcome name = "Diary" / > ;
ReactDOM.render(
    element,
    container1
);

//2 流逝时间计时器
var Elapsed = React.createClass({
    render: function () {
        var elapsed = Math.round(this.props.elapsed / 100);
        var seconds = elapsed / 10 + (elapsed % 10 ? '' : '.0');
        var message =
            'React has been successfully running for ' + seconds + ' seconds.';

        return <p > { message } < /p>;
    }
});

var start = new Date().getTime();
setInterval(function () {
    ReactDOM.render( < Elapsed elapsed = { new Date().getTime() - start }
        / \> ,
        container2
    );
}, 50);

//3 按钮点击计数

var Counter = React.createClass({
    getInitialState: function () {
        return { 
            count: 2,
            sum: 100
        };
    },
    handleClick: function () {
        this.setState({
            count: this.state.count * this.state.count,
        });
    },
    handleDoubleClick: function () {
        this.setState({
            count: 2,
            sum: 1000
        })
    },
    render: function () {
        return ( 
            < button onClick = { this.handleClick } onDoubleClick = { this.handleDoubleClick }>
            sum - count: { this.state.sum - this.state.count } < /button>
        );
    }
});
ReactDOM.render( < Counter / > ,
    container3
);


//4 流逝时间计时器（ES6写法）

class ExampleApplication extends React.Component {
    render() {
        var elapsed = Math.round(this.props.elapsed / 100);
        var seconds = elapsed / 10 + (elapsed % 10 ? '' : '.0');
        var message =
            \`React has been successfully running for ${seconds} seconds.\`;

        return <p > { message } < /p>;
    }
}
var start = new Date().getTime();
setInterval(() => {
    ReactDOM.render( < ExampleApplication elapsed = { new Date().getTime() - start }/>,
        container4
    );
}, 50);


//5 实时求解一元二次方程

var QuadraticCalculator = React.createClass({
  getInitialState: function() {
    return {
      a: 1,
      b: 3,
      c: -4
    };
  },

  /\*\*
   \* This function will be re-bound in render multiple times. Each .bind() will
   \* create a new function that calls this with the appropriate key as well as
   \* the event. The key is the key in the state object that the value should be
   \* mapped from.
   */
  handleInputChange: function(key, event) {
    var partialState = {};
    partialState\[key\] = parseFloat(event.target.value);
    this.setState(partialState);
  },

  render: function() {
    var a = this.state.a;
    var b = this.state.b;
    var c = this.state.c;
    var root = Math.sqrt(Math.pow(b, 2) - 4 * a * c);
    var denominator = 2 * a;
    var x1 = (-b + root) / denominator;
    var x2 = (-b - root) / denominator;
    return (
      <div>
        <strong>
          <em>ax</em><sup>2</sup> + <em>bx</em> + <em>c</em> = 0
        </strong>
        <h4>Solve for <em>x</em>:</h4>
        <p>
          <label>
            a: <input type="number" value={a} onChange={this.handleInputChange.bind(null, 'a')} />
          </label>
          <br />
          <label>
            b: <input type="number" value={b} onChange={this.handleInputChange.bind(null, 'b')} />
          </label>
          <br />
          <label>
            c: <input type="number" value={c} onChange={this.handleInputChange.bind(null, 'c')} />
          </label>
          <br />
          x: <strong>{x1}, {x2}</strong>
        </p>
      </div>
    );
  }
});

ReactDOM.render(
  <QuadraticCalculator />,
  container5
);


//6 调用React组件CSSTransitionGroup制作动画效果

var CSSTransitionGroup = React.addons.CSSTransitionGroup;
var INTERVAL = 2000;

var AnimateDemo = React.createClass({
getInitialState: function() {
  return {current: 0};
},

componentDidMount: function() {
  this.interval = setInterval(this.tick, INTERVAL);
},

componentWillUnmount: function() {
  clearInterval(this.interval);
},

tick: function() {
  this.setState({current: this.state.current + 1});
},

render: function() {
  var children = \[\];
  var colors = \['#fac', '#cdc', '#36d', '#ca0', '#0aa'\];
  for (var i = this.state.current, pos=0; i < this.state.current + colors.length; i++, pos++) {
    var style = {
      left: pos * 128,
      background: colors\[i % colors.length\]
    };
    children.push(<div key={i} className="animateItem" style={style}>{i}</div>);
  }
  return (
    <CSSTransitionGroup
      className="animateExample"
      transitionEnterTimeout={250}
      transitionLeaveTimeout={250}
      transitionName="example">
      {children}
    </CSSTransitionGroup>
  );
}
});

ReactDOM.render(
<AnimateDemo />,
container6
);

* * *

CSS代码（仅在例6中使用）

.example-enter,
.example-leave {
  -webkit-transition: all .25s;
  transition: all .25s;
}

.example-enter,
.example-leave.example-leave-active {
  opacity: 0.01;
}

.example-leave.example-leave-active {
  margin-left: -128px;
}

.example-enter {
  margin-left: 128px;
}

.example-enter.example-enter-active,
.example-leave {
  margin-left: 0;
  opacity: 1;
}

.animateExample {
  display: block;
  height: 128px;
  position: relative;
  width: 384px;
}

.animateItem {
  color: white;
  font-size: 36px;
  font-weight: bold;
  height: 128px;
  line-height: 128px;
  position: absolute;
  text-align: center;
  -webkit-transition: all .25s; /* TODO: make this a move animation */
  transition: all .25s; /* TODO: make this a move animation */
  width: 128px;
}