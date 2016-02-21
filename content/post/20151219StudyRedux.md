+++
date = "2015-12-19T22:08:33+09:00"
draft = true
title = "ReduxをFluxに当てはめながら考える"

+++
Reactと組み合わせて使うFluxフレームワークとして、Reduxが主流になりつつあるので使えようになりたい。
下記のチュートリアルを読んで学んだ過程をメモしたい。
https://github.com/happypoulp/redux-tutorial
ちなみに私は今までFacebookが提供していたDispatcherだけ使って自分の好きなようにFluxを構成していた。
このチュートリアルはFluxがMVCとどう違うのかとか、
Reduxの部品はFluxの何に当たるのかとか書いてあったので私にとってわかりやすいだろうと思ってこれを選んだ。

## MVCとFlux
### MVC
1. User clicks on button "A"
2. A click handler on button "A" triggers a change on Model "A"
3. A change handler on Model "A" triggers a change on Model "B"
4. A change handler on Model "B" triggers a change on View "B" that re-renders itself

どのViewもどのModelを参照することができ、どのModelも他のModelを参照することができる。
Modelに対してデータが色んな所から流れ込んで来るから、バグが紛れ込んだら治すのは大変なことになる。

### Flux
1. user clicks on button "A"
2. a handler on button "A" triggers an action that is dispatched and produces a change on Store "A"
3. since all other stores are also notified about the action, Store B can react to the same action too
4. View "B" gets notified by the change in Stores A and B, and re-renders

StoreはActionのみによって変更され得る。データの流れがはっきりしていてわかりやすい。
データはいつも以下のように流れていく
```
action -> store -> view -> action -> store -> view -> action -> ...
```

## Redux
### ActionCreator
```
// The action creator is just a function...
var actionCreator = function() {
    // ...that creates an action (yeah, the name action creator is pretty obvious now) and returns it
    return {
        type: 'AN_ACTION'
    }
}
```
ただの関数。
ActionとはActionCreatorが発行するオブジェクトのこと。
だからActionを発行するということは定義したActionCreatorを呼ぶということ。

### Store
Storeは完全にReduxが管理している。
Storeをインスタンス化するには以下のようにする。
```
import { createStore } from 'redux'

var store = createStore(() => {})
```
`createStore`には例として無名関数を渡しているが、実際にはReducerを渡すことになる。

### Reducer
Reducerは現在のアプリの状態とActionを引数に取り、新しいアプリの状態を返すただの関数だ。
Fluxで言うところのStoreに近い働きをするが、少し違って
FluxのStoreはそれ自体が状態を持つが、ReduxのReducerはstateを引数に取り新たな状態を返すためRedux自体が状態を持たない。
ReducerはステートレスなStoreというわけだ。

以下を実行してみる
```
import { createStore } from 'redux'

var reducer_0 = function (state, action) {
    console.log('reducer_0 was called with state', state, 'and action', action)
}

var store_0 = createStore(reducer_0)
// Output: reducer_0 was called with state undefined and action { type: '@@redux/INIT' }

console.log('store_0 state after initialization:', store_0.getState())
// Output: store_0 state after initialization: undefined
```
明示的にActionを呼ばなくてもStoreの初期化の際に`{ type: '@@redux/INIT' }`が呼ばれる。
だから`reducer`が実行されて`console.log`の内容が出力される。
しかし、 Reducerはconsole.logするだけで何も返していないからアプリのstateはまだundefinedになる。

じゃあ、まだstateがundefinedだったとき、stateを`{}`で初期化する処理を追加しよう。
とは言ってもES6では関数の引数に初期値を与えることができるから、簡単にこう書ける。
```
var reducer_2 = function (state = {}, action) {
    console.log('reducer_2 was called with state', state, 'and action', action)

    return state;
}

var store_2 = createStore(reducer_2)
// Output: reducer_2 was called with state {} and action { type: '@@redux/INIT' }
```
これでstateの初期化はできるようになった。
次はActionに応じた状態を返すようにしてみる。

```
var reducer_3 = function (state = {}, action) {
    console.log('reducer_1 was called with state', state, 'and action', action)

    switch (action.type) {
        case 'SAY_SOMETHING':
            return {
                ...state,
                message: action.value
            }
        case 'DO_SOMETHING':
            // ...
        case 'LEARN_SOMETHING':
            // ...
        case 'HEAR_SOMETHING':
            // ...
        case 'GO_SOMEWHERE':
            // ...
        // etc.
        default:
            return state;
    }
}
```

1. action contains a type and a value property
2. actionに対して返す値を決めるswitch文がある
3. default文を使って必ずstateを返すようにしている。さもないと`undifined`が返されてアプリの状態が失われてしまう。
4. `{ ...state, message: action.value }`というふうにstateに新しい要素をマージして返している。
`...state`と書けるのはES7のObject Spreadのおかげ。
5. Note also that this ES7 Object Spread notation suits our example because it's doing a shallow
//        copy of { message: action.value } over our state (meaning that first level properties of state
//        are completely overwritten - as opposed to gracefully merged - by first level property of
//        { message: action.value }). But if we had a more complex / nested data structure, you might choose
//        to handle your state's updates very differently:
//        - using Immutable.js (https://facebook.github.io/immutable-js/)
//        - using Object.assign (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
//        - using manual merge
//        - or whatever other strategy that suits your needs and the structure of your state since
//          Redux is absolutely NOT opinionated on this (remember, Redux is a state container).

一つのReducerだけで全部のActionを処理するのは見た目ごちゃごちゃしてつらいので、複数に分ける。
```
var userReducer = function (state = {}, action) {
    console.log('userReducer was called with state', state, 'and action', action)

    switch (action.type) {
        case 'SET_NAME':
            return {
                ...state,
                name: action.name
            }
        default:
            return state;
    }
}
var itemsReducer = function (state = [], action) {
    console.log('itemsReducer was called with state', state, 'and action', action)

    switch (action.type) {
        case 'ADD_ITEM':
            return [
                ...state,
                action.item
            ]
        default:
            return state;
    }
}
```
そしてcombineReducersヘルパー関数を使ってReducerを一つにまとめる。
```
import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
    user: userReducer,
    items: itemsReducer
})
// Output:
// userReducer was called with state {} and action { type: '@@redux/INIT' }
// userReducer was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_9.r.k.r.i.c.n.m.i' }
// itemsReducer was called with state [] and action { type: '@@redux/INIT' }
// itemsReducer was called with state [] and action { type: '@@redux/PROBE_UNKNOWN_ACTION_4.f.i.z.l.3.7.s.y.v.i' }
var store_0 = createStore(reducer)
// Output:
// userReducer was called with state {} and action { type: '@@redux/INIT' }
// itemsReducer was called with state [] and action { type: '@@redux/INIT' }

console.log('store_0 state after initialization:', store_0.getState())
// Output:
// store_0 state after initialization: { user: {}, items: [] }
```
新しい`reducer`を定義した時に実行された`userReducer was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_9.r.k.r.i.c.n.m.i' }`はReducerが`undefined`を返さないかcombineReducersがチェックするために自動的に発行されるActionによるものだ。

さぁActionを渡してみよう。
要はこういうふうにActionを渡すわけだけど、
```
store_0.dispatch({
    type: 'AN_ACTION'
})
```
せっかくだからActionCreatorを使う。
```
var setNameActionCreator = function (name) {
    return {
        type: 'SET_NAME',
        name: name
    }
}
```
そして以下のように`dispatch`を使ってActionを発行する。
```
store_0.dispatch(setNameActionCreator('bob'))
```
Actionが発行されて`store_0`が変更されたことは`subscriber`を使って検知することができる。
```
store_0.subscribe(function() {
    console.log('store_0 has been updated. Latest store state:', store_0.getState());
    // Update your views here
})
```
一連の流れをまとめると、こうなる。
```
import { createStore, combineReducers } from 'redux'

var itemsReducer = function (state = [], action) {
    console.log('itemsReducer was called with state', state, 'and action', action)

    switch (action.type) {
        case 'ADD_ITEM':
            return [
                ...state,
                action.item
            ]
        default:
            return state;
    }
}

var reducer = combineReducers({ items: itemsReducer })
var store_0 = createStore(reducer)

store_0.subscribe(function() {
    console.log('store_0 has been updated. Latest store state:', store_0.getState());
    // Update your views here
})

var addItemActionCreator = function (item) {
    return {
        type: 'ADD_ITEM',
        item: item
    }
}

store_0.dispatch(addItemActionCreator({ id: 1234, description: 'anything' }))

// Output:
//     ...
//     store_0 has been updated. Latest store state: { items: [ { id: 1234, description: 'anything' } ] }
```
`user.name`の値を`'bob'`にセットするActionを呼んで、アプリの状態を変えることに成功した。
Reduxの状態の変化を検出できるようになったので、これをReactに伝えればいいのだが、そういったボイラープレートを書かなくてもReduxとReactを接続してくれる、`react-redux`というライブラリが存在する。

## React と Redux の接続

## Middleware
でもこれだけでは例として簡単すぎる。
例えばWebアプリにはhttpリクエストの結果を使うような処理が含まれることが多いので、非同期にActionを発行する方法も知らなければならない。
非同期処理が必要な例として他には「ユーザーがボタンをクリックしてから2秒後に"Hi"と表示したい」など考えられる。
そこでmiddlewareが登場するわけだけど、ちょっと保留。
多少複雑だから飛ばしてもいいかな。

## React と Redux の接続
application.js

Providerの目的はアプリケーションのすべてのComponentにReduxの状態を行き渡らせることだ。

```
import React from 'react'
import Home from './home'
import { Provider } from 'react-redux'

export default class Application extends React.Component {
  render () {
    return (
      // As explained above, the Provider must wrap your application's Root component. This way,
      // this component and all of its children (even deeply nested ones) will have access to your
      // Redux store. Of course, to allow Provider to do that, you must give it the store
      // you built previously (via a "store" props).
      <Provider store={ this.props.store }>
        <Home />
      </Provider>
    )
  }
}

// Go to ./home.jsx to discover how you could read from state and dispatch an action from a React component.
```
Store生成の詳細は./create-store.jsに記述されています。
createStore関数を使ってReduxのインスタンスを生成します。
ApplicationはアプリケーションのルートコンポーネントでありReduxのProviderを保持しているものです。
ReactDom.render()を使ってDOMに対して描画しましょう。
propsとしてReduxのstoreを与えるとProviderが適切に動きます。

index.jsx
```
import React from 'react'
import { render } from 'react-dom'
import createStore from './create-store'
import Application from './application'

const store = createStore()

render(
  <Application store={store} />,
  document.getElementById('app-wrapper')
)
```
index.htmlはこんな感じ。
bundle.jsはコンパイルを経て結合されたやつ。
この一つのjsファイルだけをhtmlに渡す。
複数ファイルに分けるよりかは接続を確立するまでにかかる時間ぶん読み込みが早くなる。
```
<!DOCTYPE html>
<html>
  <head>
    <meta charSet="utf-8" />
  </head>
  <body>
    <div id="app-wrapper"></div>
    <script type="text/javascript" src="bundle.js"></script>
  </body>
</html>
```

## Middleware
// action ---> dispatcher ---> middleware 1 ---> middleware 2 ---> reducers

// Our middleware will be called each time an action (or whatever else, like a function in our
// async action creator case) is dispatched and it should be able to help our action creator
// dispatch the real action when it wants to (or do nothing - this is a totally valid and
// sometimes desired behavior).
