## 二、react 生命周期 ##
[钩子函数](https://www.cnblogs.com/fightjianxian/p/12457433.html)

 1. 生命周期
     - Initialization -->  Mounting --> updation --> Unmounting
         -   初始化阶段    --> 虚拟DOM挂载阶段 --> 组件更新阶段 --> 组件卸载阶段
 2. 生命周期阶段
     - <font color=red>Initialization</font>:属性和状态的初始化，在es6的constructor构造器里完成
    ```
       class Xiao extends Componet {
         constructor(props){
            super(props);
            this.state={}
          }
    }
    ```

     - <font color=red>Mounting</font>:
         - componentWillMount --> render --> componentDidMount
              -  ![title](/api/file/getImage?fileId=60861c0909eb7d0d960001b9)

     - <font color=red>updation</font>:
          - state的变化： shouldComponentUpdate --> componentWillUpdate --> render --> componentDidUpdate
          - props 的变化： componentWillReceiveProps --> shouldComponentUpdate --> componentWillUpdate --> render --> componentDidMount

          ![title](/api/file/getImage?fileId=6086210c09eb7d0d960001bb)
         - 总结：该生命周期触发需要满足两个条件：
             `1.组件第一次存在于DOM中，函数不会被执行`
             `2.如果已经存在于DOM里，第二次发生变化时，函数才会被执行`

     - <font color=red>unMounting</font>:
          - componentWillUnmount

-----------

##二、基本问题

 1. 问题一：<font color=red> setState到底是同步的还是异步的？</font>
     - （1）示例查看结果：
```
    <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <link href="css/bootstrap.min.css" rel="stylesheet">
            <script src="https://cdn.staticfile.org/react/16.4.0/umd/react.development.js"></script>
            <script src="https://cdn.staticfile.org/react-dom/16.4.0/umd/react-dom.development.js"></script>
            <script src="https://cdn.staticfile.org/babel-standalone/6.26.0/babel.min.js"></script>
            <title>查看setState 示例</title>
        </head>
        <body>

        <div class="well div" id="example"></div>

        <script type="text/babel">
            class MyComponent extends React.Component{
                constructor(props) {
                    super(props)
                    this.state ={
                        value:0
                    }
                }
                handleClick = () => {
                    this.setState({value:1})
                    console.log('在handleClick里输出' + this.state.value);
                }

                handleStateChange1 = () => {
                    this.setState({value:1})
                    console.log('在handleClick里输出' + this.state.value);
                }
                handleStateChange2 = () => {
                    this.setState({value:2})
                    console.log('在handleClick里输出' + this.state.value);
                }
                handleStateChange3 = () => {
                    this.setState({value:3})
                    console.log('在handleClick里输出' + this.state.value);
                }
                // handleClick = () => {
                //     this.handleStateChange1();
                //     this.handleStateChange2();
                //     this.handleStateChange3();
                // }

                render(){
                    console.log('在render()里输出' + this.state.value);
                    return(
                        <div>
                            <button onClick ={this.handleClick}> 按钮 </button>
                            <h2 >{this.state.value}</h2>
                        </div>
                    );
                }
            }

            ReactDOM.render(
                < MyComponent />,
                document.getElementById('example')
            );
        </script>

        </body>
        </html>
  ```

     - （2）当点击按钮时，调用handleClick函数，首先调用this.setState()设置value,随即把this.state.value输出，但结果是，虽然调用了setState({value:1})，但this.state.value并不会马上变成0，而是直到render()函数调用时，setState()才真正被执行；如下所示
         - ![验证结果](/api/file/getImage?fileId=607fca7e09eb7d0d960000f0)
     - （3）<font color=red>要是在render()前多次调用this.setState()改变同一个值呢？</font>
         - 对handleClick做一些修改，让它变得复杂一点，在调用handleClick的时候，依次调用handleStateChange1 ，handleStateChange2，handleStateChange3，它们会调用setState分别设置value为1,2,3并且随即打印;结果如下所示。
         - ![验证结果](/api/file/getImage?fileId=607fccd109eb7d0d960000f1)
     - （4）setState渲染流程
     !![title](/api/file/getImage?fileId=607fd62409eb7d0d960000f6)

     - 总结： <font color=red>setSate大部分的时候是异步执行的，但是，在react本身监听不到的地方，如原生js的监听里，setInterval,setTimeout里，setState就是同步更新的</font>




 2. 问题二:<font color=red>如何在子组件中改变父组件的state？</red>
      - （1）示例查看：

             class Son extends React.Component{
                render(){
                    console.log('儿子：在render()里输出' + this.props.value);
                    return(
                        <div>
                            <div onClick = {this.props.handleClick}>
                                {this.props.value}
                            </div>
                        </div>

                    )
                }
            }
            class Father extends React.Component{
                constructor(props){
                    super(props)
                    this.state ={
                        value:'a',
                    }

                }
                handleClick = () => {
                    this.setState({value:'b'})
                }
                render(){
                    console.log('父亲：在render()里输出' + this.state.value);
                    return (
                        <div style ={{margin:50}}>
                            <button onClick ={this.handleClick}>按钮</button>
                            <Son value = {this.state.value}  />
                        </div>
                    );
                }
            }

            ReactDOM.render(
                <Father />,
                document.getElementById("example")
            )
        - （2）子组件改变父组件的state可以采用`props`,如上述代码所示，结果如下：
        - ![title](/api/file/getImage?fileId=607fd30009eb7d0d960000f5)


 3. context的运用
      - （1） 原始代碼使用
    ```
     class Son extends React.Component{
            render(){
                console.log('儿子：在render()里输出' + this.props.gene);
                return (
                    <div onClick = {this.props.handleClick} style={{marginLeft:'20px',marginTop:'10px'}}>
                        <h3 style ={{marginTop:30}}>我从我的爷爷那里得到了基因--
                            <span style={{color:'red'}}>{this.props.gene}</span>
                        </h3>
                    </div>
                )
            }
        }
        class Father extends React.Component{
            render(){
                console.log('爸爸：在render()里输出' + this.props.gene);
                return (<Son gene = {this.props.gene}/>)
            }
        }
        class GrandFather extends React.Component{
            constructor(props) {
                super(props)
                this.state ={
                    gene:'[爷爷的基因]'
                }
            }
            handleClick = () =>{
                this.setState({gene:'奶奶的基因'})
            }
            render(){
                console.log('爷爷：在render()里输出' + this.state.gene);
                return (
                    <div>
                        <button onClick ={this.handleClick}  style={{marginLeft:'20px',marginTop:'10px',backgroundColor:'yellow'}}>按钮</button>
                        <Father gene = {this.state.gene} />
                    </div>

                )
            }
        }
    ```


      - （2）更新代码如下：
      ```
      class Son extends React.Component{
            render(){
                console.log('儿子：在render()里输出' + this.context.gene);
                return (
                    <div onClick = {this.props.handleClick} style={{marginLeft:'20px',marginTop:'10px'}}>
                        <h3 style ={{marginTop:30}}>我从我的爷爷那里得到了基因--
                            <span style={{color:'red'}}>{this.context.gene}</span>
                        </h3>
                    </div>
                )
            }
        }
        Son.contextTypes ={
            gene:React.PropTypes.string
        }
        class Father extends React.Component{
            render(){
                console.log('爸爸：在render()里输出' + this.context.gene);
                return (<Son gene = {this.context.gene}/>)
            }
        }
        class GrandFather extends React.Component{
            getChildContext(){
                return {gene:'爷爷的基因'}
            }

            render(){
                console.log('爷爷：在render()里输出' + this.context.gene);
                return (
                    <div>
                        <button onClick ={this.handleClick}  style={{marginLeft:'20px',marginTop:'10px',backgroundColor:'yellow'}}>按钮</button>
                        <Father gene = {this.context.gene} />
                    </div>

                )
            }
        }
        GrandFather.childContextTypes = {
            gene: React.PropTypes.string
        };
      ```

      - 解释下代码：
        > getChildContext()是你在顶层组件中定义的钩子函数，这个函数返回一个对象——你希望在后代组件中取用的属性就放在这个对象中，譬如这个例子中我希望在Son组件中通过this.context.gene取属性，所以在getChildContext()中返回{gene:'[爷爷的基因]'}
GrandFather.childContextTypes和Son.contextTypes 用于规定顶层组件和取顶层组件context的后代组件的属性类型

   
   - （3） context 和 props之间的区别：
          ![title](/api/file/getImage?fileId=607fec7309eb7d0d96000131)
   
   - （4）新版本react的写法：
                 ```
                    const PropTypes = require("Prop-Types");
                    GrandFather.childContextTypes = {
                         gene: PropTypes.string
                    }
                 ```









如要仔细查看demo的流程讲解，请到地址查看：https://github.com/BingerCoder/react-demo
