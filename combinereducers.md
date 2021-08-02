# `combineReducers(reducers)`

アプリケーションが複雑になって、[reducing function](https://japanese-document.github.io/redux/glossary.html#reducer)を[state](https://japanese-document.github.io/redux/glossary.html#state)のキーごとに関数に分割したいとします。

`combineReducers`関数はobjectが持つ複数のreducing functionを[`createStore`](https://redux.js.org/api/createstore)へ渡すことができる1つの関数に変換します。

`combineReducers`関数の戻り値のreducerは各子reducerを実行して、その結果を1つのstate objectにまとめます。**`combineReducers()`が生成するreducerが生成するstateは、`combineReducers()`に渡されたobjectのkeyをkeyにします。そして、objectの各keyに格納されているreducerの結果をそのkeyの値にします。**

例:

```js
rootReducer = combineReducers({potato: potatoReducer, tomato: tomatoReducer})
// rootReducerは以下のstate objectを生成します。
{
  potato: {
    // ... potatoReducerが生成したstate
  },
  tomato: {
    // ... tomatoReducerが生成したstate
  }
}
```

`combineReducers()`に渡すobjectのkey名を指定することで、stateのkey名を指定することができます。例えば、`combineReducers({ todos: myTodosReducer, counter: myCounterReducer })`を実行した場合、stateを形状は`{ todos, counter }`になります。

A popular convention is to name reducers after the state slices they manage, so you can use ES6 property shorthand notation:
一般的に、reducerの関数名はそれが管理するstateの該当部分の名前にします。そうすると、[ES 2015のshorthand記法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Object_initializer#new_notations_in_ecmascript_2015)を使って次のように記述することができます。
`combineReducers({ counter, todos })`。これは`combineReducers({ counter: counter, todos: todos })`と等価です。

#### Arguments

1. `reducers` (_Object_): 複数のreducing functionを持つobjectです。reducing functionは`combineReducers()`によって1つにまとめられます。objectに格納されるreducing functionの注意点が以下にあります。

#### Returns

(_Function_): 引数の`reducers` objectに格納されているすべてのreducerを実行するreducerで、`reducers` objectと同じ形状のstate objectを生成するreducer

#### 注意点

`combineReducers`に渡されるreducerは以下のルールを満たす必要があります。

- reducerの対象外のactionの場合、reducerの第1引数のstateを返す必要があります。

- `undefined`を返してはいけません。単なる`return`文で間違えることはよくあることです。reducerが`undefined`を返した場合、errorをthrowします。

- reducerの第1引数の`state`の値が`undefined`である場合、reducerはinital stateを返す必要があります。前述のルールに従うとinitial stateは`undefined`ではない必要があります。[ES 2015のデフォルト引数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Default_parameters)を使うことをお勧めしますが、明示的に第1引数が`undefined`か確認することもできます。

`combineReducers`はこれらのルールの一部に関して渡されたreducerが適合しているか確認しますが、これらを覚えて、それに従うようにしてください。`combineReducers`は渡されたreducerにundefinedを渡して確認します。これはinitial stateを`Redux.createStore(combineReducers(...), initialState)`のように指定した場合でも行われます。だから、必ず、意図せずreducerの第1引数のstateが`undefined`になった場合でも、正常に動作するようにreducerを実装してください。

#### Example

#### `reducers/todos.js`

```js
export default function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
  }
}
```

#### `reducers/counter.js`

```js
export default function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

#### `reducers/index.js`

```js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({
  todos,
  counter
})
```

#### `App.js`

```js
import { createStore } from 'redux'
import reducer from './reducers/index'

const store = createStore(reducer)
console.log(store.getState())
// {
//   counter: 0,
//   todos: []
// }

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
console.log(store.getState())
// {
//   counter: 0,
//   todos: [ 'Use Redux' ]
// }
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
* [Redux Thunk](https://japanese-document.github.io/redux-thunk/)
