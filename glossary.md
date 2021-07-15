# 用語

ここではReduxでよく使われる用語をその型と併せて説明します。その型は[Flowの表記](https://flowtype.org/docs/quick-reference.html)を使って説明します。

## State

```js
type State = any
```

_State_ (別名 _state tree_ )はいろいろな意味を持ちます。通常、Redux APIでは、Storeによって管理され、[`getState()`](https://redux.js.org/api/store#getstate)で返される単独の状態を意味します。 それはReduxを使ったアプリケーション全体の状態を表します。多くの場合、その状態は複雑なObjectです。

慣習ではState自体はObjectやMapのようなkey-value collectionであることが多いですが、技術的には任意の型で問題ありません。それでも、Stateをシリアライズできるようにするべきです。簡単にJSONに変換できない物はStateの中に入れないようにしてください。

## Action

```js
type Action = Object
```

_Action_ はStateを変更する注文を表す素のObjectです。ActionはデータをStoreに渡す唯一の方法です。つまり、UIイベントやネットワークコールバックやWebSocketのような他のソースからのデータは最終的にActionとしてdispatchする必要があります。

Actionにそれの種類を示す`type`フィールドを設定する必要があります。`type`フィールドの値は定数として定義したり、他のモジュールからimportすることができます。`type`フィールドの値は[Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol)より文字列の方が適しています。なぜなら、シリアライズできるからです。

`type`フィールド以外のActionオブジェクトの構造は特に決まっていません。それに興味がある場合は、[Flux Standard Action](https://github.com/acdlite/flux-standard-action)が参考になると思います。

以下の[Async Action](#async-action)も見ましょう。

## Reducer

```js
type Reducer<S, A> = (state: S, action: A) => S
```

_reducer_ (別名 _reducing function_) は累算結果と値を受け取って、新しい累算結果を返す関数です。 これはコレクションの値を1つの値に畳み込むために使われます。

ReducerはReduxだけの概念ではありません。関数型プログラミングの基本的な概念です。JavaScriptのようなほとんど関数型言語でない言語でも、reduceするためのAPIを備えています。JavaScriptのそれは[`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)です。

Reduxでは累算結果はStateオブジェクトです。そして、累算される値はActionです。Reducerは既存のStateとActionを受け取って新しいStateを算出します。Reducerは _純粋関数(Pure Function)_ である必要があります。つまり、同じ入力に対して完全に同じ出力を返す必要があります。

hot reloadingやtime travelのような素晴らしい機能を動作させるために、Reducerは副作用(side-effects)を含んでいてはいけません。

ReducerはReduxの最も重要なコンセプトです。

_API呼び出しをReducerに入れないでください。_

## Dispatching Function

```js
type BaseDispatch = (a: Action) => Action
type Dispatch = (a: Action | AsyncAction) => any
```

_Dispatching Function_ (または、simply dispatch function )は、1つのActionもしくは1つの[Async Action](#async-action)を受け取る関数です。この関数は1つのActionもStoreへdispatchしない場合と1つまたは複数のActionをStoreへdispatchする場合があります。

一般的なDispatching FunctionとStoreインスタンスのMiddlewareを通さないBase [`Dispatch`](https://redux.js.org/api/Store#dispatchaction) Functionは異なります。

Base Dispatch Functionは常に同期的なActionをStoreのReducerへ送ります。そのActionとReducerと既存のStoreのStateを使って、新しいStateを算出します。このActionは素のObjectにReducer用の値をセットした物です。

[Middleware](#middleware)はBase Dispatch Functionをラップします。これによって、dispatch関数はActionに加えて[Async Action](#async-action)を取り扱うことが可能になります。(Dispatching Functionになる。)MiddlewareはActionとAsync Actionを次のMiddlewareに渡す前に、それらを変換したり、遅延したり、無視したり、それ以外の処理を加える場合があります。詳しい説明は以下にあります。

## Action Creator

```js
type ActionCreator<A, P extends any[] = any[]> = (...args: P) => Action | AsyncAction
```

_Action Creator_ は端的に言うとActionを生成する関数です。もう一度言いますが、ActionはReducerへ送信されるデータ本体でAction CreaterはActionを生成する関数です。この2つを混同しないように注意してください。Action Creatorを実行するとActionが生成されるだけで、Actionはdispatchされません。Stateを変更したい場合は、Storeの [`dispatch`](https://redux.js.org/api/Store#dispatchaction)関数を実行する必要があります。Actionを生成して、それを特定のStoreインスタンスにdispatchするAction Creatorを _Bound Action Creator_ と呼びます。

Action Creatorが現在のStateを読み込んだり、APIを読み込んだり、RouterのTransitionのような副作用を発生させる必要がある場合は、Actionの代わりに[Async Action](#async-action)を返す必要があります。

## Async Action

```js
type AsyncAction = any
```

_Async Action_ はDispatching Functionに渡される値ですが、そのままではReducerで処理することができません。Async ActionはBase [`Dispatch`](https://redux.js.org/api/Store#dispatchaction) Functionに渡される前に、[Middleware](#middleware)で1つまたは複数のActionに変換されます。Async Actionは適用するMiddlewareごとに型が異なります。たいてい、Async Actionは、Promiseやthunkのような非同期処理を実行するものです。Async ActionはReducerにそのまま渡されません。Middlewareで、それらの処理が完了した後、Actionがdispatchされます。

## Middleware

```js
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```

MiddlewareはDispatching Functionを関数合成して新しい[Dispatching Function](#dispatching-function)を返す高階関数(higher-order function)です。たいてい、それは [Async Action](#async-action)をActionに変換します。

Middlewareは関数合成(function composition)で生成されます。
これはログを出力やRoutingのような副作用の実行や非同期のAPIの呼び出しに対応するMiddlewareを合成することで、それぞれを(非同期の)Actionに変換する1つのMiddlewareにすることを可能にします。詳しくは[`applyMiddleware(...middlewares)`](https://redux.js.org/api/applyMiddleware)を見てください。

## Store

```js
type Store = {
  dispatch: Dispatch
  getState: () => State
  subscribe: (listener: () => void) => () => void
  replaceReducer: (reducer: Reducer) => void
}
```

Storeはアプリケーションのstate treeを保存するオブジェクトです。そのstate treeはReducerで構成されます。Reduxを使っているアプリケーションではStoreは1つのみ存在してるべきです。

- [`dispatch(action)`](https://redux.js.org/api/Store#dispatchaction)は上記のBase Dispatch Functionです。
- [`getState()`](https://redux.js.org/api/Store#getState)はStoreの現在のStateを返します。
- [`subscribe(listener)`](https://redux.js.org/api/Store#subscribelistener)Stateが変化したときに実行される関数を登録します。
- [`replaceReducer(nextReducer)`](https://redux.js.org/api/Store#replacereducernextreducer)はhot reloadingやcode splittingの実装に使うことができます。これを使うことはほとんどないと思います。

詳しく知りたい場合は[Store APIリファレンス](https://redux.js.org/api/store#dispatchaction)を見てください。

## Store Creator

```js
type StoreCreator = (reducer: Reducer, preloadedState: ?State) => Store
```

Store CreatorはStoreを生成する関数です。Dispatching Functionのように、Base Store Creator(Reduxパッケージからexportされる[`createStore(reducer, preloadedState)`](https://redux.js.org/api/createStore))とStore Enhancerで生成されるStore Creatorを区別する必要があります。

## Store Enhancer

```js
type StoreEnhancer = (next: StoreCreator) => StoreCreator
```

Store EnhancerはStore Creatorを関数合成して新しい改良されたStore Creatorを返す高階関数です。これはStoreとのやり取りを関数合成で変更できることがMiddlewareと似ています。

Store EnhancerはReactの[higher-order components](https://reactjs.org/docs/higher-order-components.html)とほとんど同じコンセプトです。だから、“component enhancers”と呼ばれることもあります。

Storeは(何らかのクラスの)インスタンスではなく関数を素のObjectにまとめたものなので、簡単に複製することができます。そして、元のStoreを変更することなく改善することができます。
[`compose`](https://redux.js.org/api/compose)のドキュメントにこの例があります。

たいてい、Store Enhancerを書くことはありませんが、[developer tools](https://github.com/reduxjs/redux-devtools)で提供されているStore Enhancerを使うことはあるかもしれません。それはアプリケーションに影響を与えずにtime travelを可能にします。面白いことに、[Redux middlewareの実装](https://redux.js.org/api/applyMiddleware)はそれ自体がStore Enhancerです。


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
