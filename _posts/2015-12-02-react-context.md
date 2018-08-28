---
layout: post
title: React Context
category: 技术
date: 2015-12-02
comments: true
---

了解下React Context这个不为人知的内容


####1、为什么引入？

若组件的层级关系较多，而底层想获取顶层的属性，而中间的组件不需要给属性

####2、是什么？

简单来说，context就是能从上层组件“隐式”的传向后代组件的一些属性，类似于props从父组件传数据到子组件

(目前为实验特性，将来的使用方法可能会改变)

***与props的区别：context可以链式的传向后代，只要后代定义了contextTypes***

```
 In brief, a context is an object that is implicitly passed
 from a component to its children, so by using contexts you 
 don’t have to explicitly pass around a whole bunch of props to 
 pass some contextual data.
 
 The difference between props and context is that context chains 
 to descendents, whereas props do not.
 
```

####3、怎么用：

外层组件需定义getChildContext方法和属性childContextTypes，子组件需定义contextTypes则可通过this.context来获取值，见例1

####4、缺点：
使数据的走向不明确，不易进行数据的追踪

####5、适用场景：

context类似于全局变量，因此像用户信息、当前语言和主题等更像是全局state可以使用context传至子组件中

*****************

###解答：React-redux库中的Provider是如何借助于react context将store传给connect的？？

Provider是一个container组件，有getChildContext方法，该方法返回{store：this.store}，且具有属性childContextTypes，指出store的类型为isRequired，***具备了getChildContext和childContextTypes***

只要它的后代组件中***定义了 contextTypes***，就可以在render中使用this.context获取到值

而使用了connect()的组件由于connect本身具备了contextTypes属性，因此可以获取到了provider提供的store
*****************

###注意：

(1) 类似于组件GrandFather-》Father-》child这样的传递过程中，若Father组件的shouldComponentUpdate返回为false，则child组件的this.context不会改变，见例2

(2) this.context的值来源于“创造它”的父类组件，不一定是parent，见例3


例1：
			
		var A = React.createClass({

            childContextTypes: {
                 name: React.PropTypes.string.isRequired
            },

            getChildContext: function() {
                 return { name: "Jonas" };
            },

            render: function() {
                 return <B />;
            }
        });

        var B = React.createClass({

            contextTypes: {
                name: React.PropTypes.string.isRequired   //若无该属性，则不会报错，但获取不到this.context.name
            },

            render: function() {
                return <div>My name is: {this.context.name}</div>;
            }
        });

        // Outputs: "My name is: Jonas"
        React.render(<A />, document.body);
        
例2

           var Bottom = React.createClass({
              contextTypes: {
                number: React.PropTypes.number.isRequired
              },

              render: function () {
                return <h1>{this.context.number}</h1>
              }
            });

            var Middle = React.createClass({
              shouldComponentUpdate: function (nextProps, nextState, nextContext) {
                return false;
              },

              render: function () {
                return <Bottom />;
              }
            });

            var Top = React.createClass({
              childContextTypes: {
                number: React.PropTypes.number.isRequired
              },

              getInitialState: function () {
                return { number: 0 };
              },

              getChildContext: function () {
                return { number: this.state.number };
              },

              componentDidMount: function () {
                setInterval(function () {
                  this.setState({
                    number: this.state.number + 1
                  });
                }.bind(this), 1000);
              },

              render: function() {
                return <Middle />;    
              }
            });

            React.render(<Top />, document.body);  //页面上显示0
            
例3

			var App = React.createClass({
              render: function() {
                return (
                  <Grandparent>
                    <Parent>
                      <Child/>
                    </Parent>
                  </Grandparent>
                );
              }
            });

            var Grandparent = React.createClass({  
              childContextTypes: {
                name: React.PropTypes.string.isRequired
              },
              getChildContext: function() {
                return {name: 'Jim'};
              },
              
              render: function() {
                return this.props.children;
              }
                
            });

            var Parent = React.createClass({
              render: function() {
                return this.props.children;
              }
            });
            
            var Child = React.createClass({
              contextTypes: {
                 name: React.PropTypes.string.isRequired
              },
              render: function() {
                 return <div>My name is {this.context.name}</div>;
              }
            });
            React.render(<App/>, document.body);
            
例3运行会出现：***Warning: owner-based and parent-based contexts differ (values: `undefined` vs `Jim`) for key (name) while mounting Child***

若将例3中的App下组件的包含关系，此时`<Child/>`组件的拥有着为`<Grandparent>`
，则可运行出正确的结果：My name is hello world，代码如下：

```
	        var App = React.createClass({
              render: function() {
                return (
                  <Grandparent>
                    { function() {
                      return (<Parent>
                        <Child/>
                      </Parent>)
                    }}
                  </Grandparent>
                );
              }
            });

            var Grandparent = React.createClass({  
              childContextTypes: {
                 name: React.PropTypes.string.isRequired
              },
              getChildContext: function() {
                return {name: 'hello world'};
              },
              
              render: function() {
                var children = this.props.children;
                children = children();
                return children;
              }
                
            });
            
            var Parent = React.createClass({
              render: function() {
                return this.props.children;
               }
            });

            var Child = React.createClass({
                contextTypes: {
                    name: React.PropTypes.string.isRequired
                },
              render: function() {
                 return <div>My name is {this.context.name}</div>;
               }
            });
            React.render(<App/>, document.body);
```

***参考网址：***

```
https://facebook.github.io/react/docs/context.html
https://www.tildedave.com/2014/11/15/introduction-to-contexts-in-react-js.html
https://medium.com/@skwee357/the-land-of-undocumented-react-js-the-context-99b3f931ff73#.qv4rflbfd
```
