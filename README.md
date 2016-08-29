
包含使用 dva 的最小知识集。

## JavaScript 语言

### 变量声明

#### const 和 let

#### 模板字符串

#### 默认参数

### 箭头函数

### 模块的 Import 和 Export

### ES6 对象和数组

#### 析构赋值

#### 对象字面量改进

#### Spread Operator
    
### Promises

### Generators

## React Component

###  Stateless Functional Components

### JSX

#### Component 嵌套

#### className

#### 注释

#### JavaScript 表达式

#### Mapping Arrays to JSX

#### Spread Attributes

### Props

#### propTypes

#### 往下传数据

#### 往上传数据

### CSS Modules

#### 定义 CSS

#### 在 js 里引用 CSS

#### `classnames` Package

## Reducer

reducer 是一个函数，接受 state 和 action，返回老的或新的 state 。即：`(state, action) => state`

### 增删改

以 todos 为例。

```javascript
app.model({
  namespace: 'todos',
  state: [],
  reducers: {
    add(state, { payload: todo }) {
      return state.concat(todo);
    },
    remove(state, { payload: id }) {
      return state.filter(todo => todo.id !== id);
    },
    update(state, { payload: updatedTodo }) {
      return state.map(todo => {
        if (todo.id === updatedTodo.id) {
          return { ...todo, ...updatedTodo };
        } else {
          return todo;
        }
      });
    },
  },
};
```

### 嵌套数据的增删改

建议最多一层嵌套，以保持 state 的扁平化，深层嵌套会让 reducer 很难写和难以维护。

```javascript
app.model({
  namespace: 'app',
  state: {
    todos: [],
    loading: false,
  },
  reducers: {
    add(state, { payload: todo }) {
      const todos = state.todos.concat(todo);
      return { ...state, todos };
    },
  },
});
```

下面是深层嵌套的例子，应尽量避免。

```javascript
app.model({
  namespace: 'app',
  state: {
    a: {
      b: {
        todos: [],
        loading: false,
      },
    },
  },
  reducers: {
    add(state, { payload: todo }) {
      const todos = state.a.b.todos.concat(todo);
      const b = { ...state.a.b, todos };
      const a = { ...state.a, b };
      return { ...state, a };
    },
  },
});
```

## Effect

示例：

```javascript
app.model({
  namespace: 'todos',
  effects: {
    *addRemote({ payload: todo }, { put, call }) {
      yield call(addTodo, todo);
      yield put({ type: 'add', payload: todo });
    },
  },
});
```

### Effects

#### put

用于触发 action 。

```javascript
yield put({ type: 'todos/add', payload: 'Learn Dva' });
```

#### call

用于调用异步逻辑，支持 promise 。

```javascript
const result = yield call(fetch, '/todos');
```

#### select

用于从 state 里获取数据。

```javascript
const todos = yield select(state => state.todos);
```

### 错误处理

#### 全局错误处理

dva 里，effects 和 subscriptions 的抛错全部会走 `onError` hook，所以可以在 `onError` 里统一处理错误。

```javascript
const app = dva({
  onError(e) {
    console.log(e.message);
  },
});
```

然后 effects 里的抛错和 reject 的 promise 就都会被捕获到了。

#### 本地错误处理

如果需要对某些 effects 的错误进行特殊处理，需要在 effect 内部加 `try catch` 。

```javascript
app.model({
  effects: {
    *addRemote() {
      try {
        // Your Code Here
      } catch(e) {
        console.log(e.message);
      }
    },
  },
});
```

### 异步请求

异步请求基于 whatwg-fetch，API 详见：https://github.com/github/fetch

#### GET 和 POST

```javascript
import request from '../util/request';

// GET
request('/api/todos');

// POST
request('/api/todos', {
  method: 'POST',
  body: JSON.stringify({ a: 1 }),
});
```

#### 统一错误处理

加入约定后台返回以下格式时，做统一的错误处理。

```javascript
{
  status: 'error',
  message: '',
}
```

编辑 `utils/request.js`，加入以下中间件：

```javascript
function parseErrorMessage({ data }) {
  const { status, message } = data;
  if (status === 'error') {
    throw new Error(message);
  }
  return { data };
}
```

然后，这类错误就会走到 `onError` hook 里。

## Subscription

### 异步数据初始化

#### `path-to-regexp` Package

## Router

### Config with JSX Element (router.js)

#### Route, Redirect, IndexRoute, IndexRedirect

### Route Components

#### 通过 connect 绑定数据

#### Injected Props (e.g. location)

### 基于 action 进行页面跳转

## dva 配置

### Redux Middleware

### history

#### 切换 hsitory 为 browserHistory

#### 去除 hashHistory 下的 _k 查询参数

## 工具

### 通过 dva-cli 创建项目



