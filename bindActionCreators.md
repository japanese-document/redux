# bindActionCreators(actionCreators, dispatch)

[action creator](https://japanese-document.github.io/redux/glossary.html#action-creator)を値として持つObjectを引数としてとります。そのObjectの値であるaction craetorをaction creatorを実行して戻り値(action)を[`dispatch`](https://redux.js.org/api/Store#dispatchaction)の引数として実行する関数(dispatch(actionCreator(args)))に置き換えます。(Objectのキーはそのまま)


(actionを適用する場合、)通常、直接[`Store`](https://redux.js.org/api/Store)の[`dispatch`](https://redux.js.org/api/Store#dispatchaction)関数を実行する必要があります。ReduxをReactと連携させる場合、[react-redux](https://github.com/gaearon/react-redux)が`dispatch`関数を提供するので、(dispatch関数を)直に実行することができます。

唯一の`bindActionCreators`の使用例は`dispatch`やReduxのStoreをコンポーネントに渡すことなく、Reduxを意識せずコンポーネントにaction creatorを渡したい場合です。

第1引数にaction creatorを渡すと、それをdispatchでラップした関数(dispatch(actionCreator(args)))を返します。

#### Parameters

1. `actionCreators` (_Function_ or _Object_): action creatorもしくはaction creatorを値に持つObject

2. `dispatch` (_Function_): `Store`の[`dispatch`](https://redux.js.org/api/Store#dispatchaction)関数

#### Returns

(_Function_ or _Object_): 第1引数にObjectが渡された場合、そのObjectの値をラップしてObjectの値の実行結果であるactionを第2引数のdispatchに渡して実行する関数(dispatch(actionCreator(args)))に置き換えて返します。第1引数に関数が渡された場合、同様に関数をラップして返します。


#### Example

#### `TodoActionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

#### `SomeComponent.js`

```js
import { Component } from 'react'
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'

import * as TodoActionCreators from './TodoActionCreators'
console.log(TodoActionCreators)
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  constructor(props) {
    super(props)

    const { dispatch } = props

    // 以下はbindActionCreatorsの使用例です。
    // 子コンポーネントにReduxを完全に意識させたくない場合、action creatorをdispatchにbindして子コンポーネントに渡します。

    this.boundActionCreators = bindActionCreators(TodoActionCreators, dispatch)
    console.log(this.boundActionCreators)
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }
  }

  componentDidMount() {
    // react-reduxよる注入
    let { dispatch } = this.props

    // 注意: これは動作しません。
    // TodoActionCreators.addTodo('Use Redux')

    // 単にaction createrを実行しているだけです。
    // actionをdispatchする必要があります。

    // これは動作します。
    let action = TodoActionCreators.addTodo('Use Redux')
    dispatch(action)
  }

  render() {
    // react-reduxよる注入
    let { todos } = this.props

    return <TodoList todos={todos} {...this.boundActionCreators} />

    // bindActionCreatorsを使う代わりにdispatch関数を子コンポーネントに渡す方法があります。
    // しかし、子コンポーネントはaction creatorをimportして、それを意識する必要があります。

    // return <TodoList todos={todos} dispatch={dispatch} />
  }
}

export default connect(state => ({ todos: state.todos }))(TodoListContainer)
```

## License

### Japanese part

Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)

Copyright (c) 2021 38elements

### Other

The MIT License (MIT)

Copyright (c) 2015-present Dan Abramov

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

<hr />

* [Redux](https://japanese-document.github.io/redux/)
