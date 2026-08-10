---
title: React从0到1入门篇
categories:
  - react
tags:
  - react
---

## 前言

本文对比 Vue 把 React 的基础技能过一遍，帮助你快速上手 React。

- 你熟悉组件化开发，熟悉 JS，熟悉 Vue，学会 React 是分分钟的事情。
- React 组件就是 JS 函数，组件的属性就是函数参数，事件也是函数参数。
- JSX 的插值、属性、事件和样式，规则很少，更简洁。
- JSX 判断和循环完全使用 JS 语法。
- React `useState` 语义明确，没有心智负担。
- React state 就是原始数据，没有任何包裹处理。
- React `useEffect` 涵盖所有副作用操作，API 更简洁。

本文一边学习 React 基础技能，一边做一个 todo-list 项目，学以致用。

---

## 创建项目

使用 `create-react-app` 脚手架创建一个 React 项目并启动：

```bash
npx create-react-app react-todo-list
cd react-todo-list
npm start
```

> PS：React 官网已经不推荐使用 `create-react-app` 了，但这里并不影响本文的学习和使用。

---

## 组件

无论 Vue 还是 React，页面都是由一个一个组件嵌套组成的，让我们先从组件开始。

### 组件定义

React 组件一般是一个 JS 文件，用一个 JS 函数定义组件。组件函数会返回一段 JSX 代码，JSX 语法和 HTML 语法很相似，也很好理解。

```jsx
// src/App.js
function App() {
  return (
    // JSX 语法，类似于 HTML
    <div>
      <p>hello React</p>
    </div>
  )
}

export default App
```

**React 本质是 JS 函数**，这不难吧。至于 JSX，后面再讲。

### 组件结构

无论 Vue React 都是支持组件嵌套的，嵌套规则和格式都是参考 HTML 语法。

对于自定义组件的 tag 名称，Vue 和 React 现在都推荐使用 **PascalCase** 写法，如 `<PostBlogPage />`，即首字母大写。

```jsx
// src/components/TodoInput.js
function TodoInput() {
  return (
    <div>
      <p>todo input</p>
    </div>
  )
}

export default TodoInput
```

```jsx
// src/components/TodoList.js
function TodoList() {
  return (
    <div>
      <p>todo list</p>
    </div>
  )
}

export default TodoList
```

```jsx
// src/App.js
import TodoInput from './components/TodoInput'
import TodoList from './components/TodoList'

function App() {
  return (
    <div>
      <TodoInput />
      <TodoList />
    </div>
  )
}

export default App
```

### 组件属性

React 组件是一个函数，**它的属性就是函数的参数**，就是如此简单。

```jsx
// src/components/TodoList.js
function TodoList(props) {
  console.log(props.foo)
  return (
    <div>
      <p>todo list</p>
    </div>
  )
}

export default TodoList
```

```jsx
// src/App.js
<TodoList foo="hello foo" />
```

**React 组件属性就是函数参数**，简单明了。

### 组件事件

React 中：**组件事件和属性一样，还是函数的参数**，执行事件就是执行一个函数，就是简单的 JS 代码。

```jsx
// src/components/TodoList.js
function TodoList(props) {
  const { onDelete } = props

  function deleteHandler() {
    onDelete('event-id')
  }

  return (
    <div>
      <p>todo list</p>
      <button onClick={deleteHandler}>删除</button>
    </div>
  )
}

export default TodoList
```

```jsx
// src/App.js
function deleteItem(id) {
  console.log('delete item', id)
}

<TodoList onDelete={deleteItem} />
```

### 子组件

React 中，子组件通过 `props.children` 获取。

```jsx
function Container(props) {
  return (
    <div>
      <p>Container</p>
      {props.children}
    </div>
  )
}

// 使用
<Container>
  <p>这是子组件内容</p>
</Container>

---

## 模板（JSX）

### 插值

React JSX 中用 `{}` 来插值，和 Vue 的 `{{}}` 类似，但只用一个花括号。

```jsx
function App() {
  const name = 'React'
  const count = 10

  return (
    <div>
      <p>hello {name}</p>
      <p>count: {count}</p>
    </div>
  )
}
```

### 属性

JSX 中的属性写法：

```jsx
function App() {
  const imgUrl = 'https://example.com/img.png'
  const isActive = true

  return (
    <div>
      <img src={imgUrl} />
      <input disabled={isActive} />
      {/* className 对应 class */}
      <p className="title">hello</p>
      {/* htmlFor 对应 for */}
      <label htmlFor="input-id">label</label>
    </div>
  )
}
```

> PS：JSX 中用 `className` 代替 `class`，用 `htmlFor` 代替 `for`，因为它们是 JS 关键字。

### 自定义事件

React 事件和 HTML 事件的写法一样，使用驼峰命名。

```jsx
function App() {
  function handleClick() {
    alert('clicked')
  }

  return (
    <button onClick={handleClick}>click me</button>
  )
}
```

### 样式

React 中内联样式是一个 JS 对象，属性名使用驼峰命名。

```jsx
function App() {
  const style1 = { fontSize: '30px', color: 'blue' }
  const style2 = { ...style1, fontWeight: 'bold' }

  return (
    <div>
      <p style={style1}>hello</p>
      <p style={style2}>world</p>
      <p style={{ fontSize: '20px', color: 'red' }}>inline style</p>
    </div>
  )
}
```

### DOM 事件

React 中 DOM 事件对象就是标准的 Event 对象。

```jsx
function App() {
  function handleClick(event) {
    console.log(event) // 标准的 Event 对象
  }

  return (
    <button onClick={handleClick}>click me</button>
  )
}
```

---

## 条件渲染

React 没有 `v-if` / `v-show` 这样的指令，而是完全使用 JS 语法来做条件渲染。

### 三元表达式

```jsx
function App() {
  const [show, setShow] = useState(true)

  return (
    <div>
      {show ? <p>显示内容</p> : <p>隐藏内容</p>}
    </div>
  )
}
```

### 逻辑与 `&&`

```jsx
function App() {
  const [show, setShow] = useState(true)

  return (
    <div>
      {show && <p>条件为真时显示</p>}
    </div>
  )
}
```

### if / else

```jsx
function App() {
  const [type, setType] = useState('a')

  if (type === 'a') {
    return <p>type is a</p>
  } else {
    return <p>type is not a</p>
  }
}
```

---

## 列表渲染

React 中没有 `v-for` 指令，而是使用 JS 数组的 `map` 方法来循环。


---

## 状态和响应式

### useState - 值类型

React 使用 `useState` 来定义状态，修改状态必须通过 `setXxx` 函数。

```jsx
import { useState } from 'react'

function App() {
  const [count, setCount] = useState(0)

  function increase() {
    setCount(count + 1) // 显式调用 setXxx
  }

  return (
    <div>
      <p>count: {count}</p>
      <button onClick={increase}>increase</button>
    </div>
  )
}

export default App
```

**关键点：**

- React state 就是原始数据，没有 Vue 那样的 Proxy 包裹。
- 修改 state 必须显式调用 `setXxx`，不能直接修改。
- `useState` 返回一个数组：`[当前值, 修改函数]`。

### React 也有 ref

React 的 `useRef` 和 Vue 的 `ref` 类似，可以用来存储不需要触发重新渲染的值，也可以引用 DOM 元素。

```jsx
import { useRef } from 'react'

function App() {
  const inputRef = useRef(null)

  function focusInput() {
    inputRef.current.focus()
  }

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={focusInput}>聚焦</button>
    </div>
  )
}
```

### 对象和数组

**对象：** 修改对象类型的 state，必须创建新对象。

```jsx
import { useState } from 'react'

function App() {
  const [user, setUser] = useState({ name: 'Tom', age: 20 })

  function changeName() {
    // ❌ 错误：直接修改不触发重新渲染
    // user.name = 'Jerry'

    // ✅ 正确：创建新对象
    setUser({ ...user, name: 'Jerry' })
  }

  return (
    <div>
      <p>{user.name} - {user.age}</p>
      <button onClick={changeName}>改名</button>
    </div>
  )
}
```

**数组：** 修改数组类型的 state，必须创建新数组。

```jsx
import { useState } from 'react'

function App() {
  const [list, setList] = useState([1, 2, 3])

  function addItem() {
    // ❌ 错误
    // list.push(4)

    // ✅ 正确：创建新数组
    setList([...list, 4])
  }

  function deleteItem(index) {
    setList(list.filter((_, i) => i !== index))
  }

  return (
    <div>
      {list.map((item, index) => (
        <p key={index}>
          {item}
          <button onClick={() => deleteItem(index)}>删除</button>
        </p>
      ))}
      <button onClick={addItem}>添加</button>
    </div>
  )
}
```

### 数据的复杂性

React state 就是原始数据，没有任何 Proxy 或 defineProperty 包裹。这意味着：

- 你直接修改 state 不会触发重新渲染（必须用 `setXxx`）。
- 没有 Vue 那样的响应式追踪，不会出现 `ref` 和 `reactive` 的混淆问题。
- 心智负担更低，你只需要记住：**想更新 UI，就用 `setXxx` 创建新数据**。
```jsx
function App() {
  const [list, setList] = useState([

---

## 副作用和组件生命周期

### useEffect - 初次渲染

React 使用 `useEffect` 来处理所有副作用（包括组件生命周期）。

```jsx
import { useState, useEffect } from 'react'

function App() {
  useEffect(() => {
    console.log('组件渲染完成，类似于 mounted')
  }, []) // 空数组 = 只在初次渲染后执行

  return <div>App</div>
}
```

### useEffect - 组件更新

`useEffect` 可以监听某些 state 变化后执行副作用。

```jsx
import { useState, useEffect } from 'react'

function App() {
  const [page, setPage] = useState(0)
  const [keyword, setKeyword] = useState('')

  function fetchData(page, keyword) {
    console.log('发起 ajax 请求', page, keyword)
  }

  // 1. 组件渲染完成触发
  // 2. page 或 keyword 变化时触发
  useEffect(() => {
    fetchData(page, keyword)
  }, [page, keyword])

  return (
    <div>
      <button onClick={() => setPage(page + 1)}>setPage</button>
      <button onClick={() => setKeyword(keyword + 'x')}>setKeyword</button>
      <p>App page</p>
    </div>
  )
}
```

> PS：不要把 `useEffect` 对比组件生命周期，用"副作用"概念去理解它。它可以在组件渲染完成后执行副作用，也可以监听某些 state 变化后执行副作用——就这两条，即可满足所有开发需求。

### useEffect - 组件销毁

`useEffect` 函数里面可以再 `return` 一个函数，用于监听组件销毁。

---

## watch 和 computed

### watch（监听数据）

Vue `watch` 监听某个数据，React 使用 `useEffect` 即可实现（上文已讲）。

### computed（计算属性）

Vue `computed` 计算数据，在 React 中分两种情况：

**第一，不用缓存计算结果（绝大部分情况）：**

直接写就行，没有任何障碍。

```jsx
const [count, setCount] = useState(0)
const doubleCount = count * 2 // 直接计算

return (
  <div>
    <button onClick={() => setCount(count + 1)}>increase</button>
    <span>{count} {doubleCount}</span>
  </div>
)
```

**第二，需要缓存计算结果（少数情况）：**

使用 `useMemo` 做缓存。只要 `count` 不变，`doubleCount` 就不会重新计算。

```jsx
import { useMemo } from 'react'

const [count, setCount] = useState(0)
const doubleCount = useMemo(() => {
  return count * 2
}, [count]) // 只有 count 变化时才重新计算

return (
  <div>
    <button onClick={() => setCount(count + 1)}>increase</button>
    <span>{count} {doubleCount}</span>
  </div>
)
```

---

## 表单（受控组件）

React 推荐使用**受控组件**来处理表单项的值。

```jsx
import { useState } from 'react'

function App() {
  const [text, setText] = useState('')

  function handleChangeText(event) {
    setText(event.target.value)
  }

  return (
    <div>
      <input value={text} onChange={handleChangeText} />
    </div>
  )
}
```

**受控组件的原理：**

- 定义 `text` 状态
- 在 `input` 的 `onChange` 中 `setText`
- 本质上就是自己实现了一套 Vue `v-model`，功能很相似

React 设计上是"单项数据流"，所以不会内置 `v-model`，要让你自己写。代码多写几行没关系，设计上的统一性更重要。

### TodoInput 实战

```jsx
import { useState } from 'react'

function TodoInput(props) {
  const { addTodo } = props
  const [text, setText] = useState('')

  function addTodoHandler() {
    addTodo(text)
    setText('')
  }

  function handleChangeText(event) {
    setText(event.target.value)
  }

  return (
    <div>
      <input value={text} onChange={handleChangeText} /> {/* 受控组件 */}
      <button onClick={addTodoHandler}>todo input</button>
    </div>
  )
}

export default TodoInput
```

---

## 开发 todo-list 项目

有了以上的基础技能，就可以轻松开发出一个 React todo-list 项目。

项目源码在：[github.com/wangfupeng1989/react-todo-list](https://github.com/wangfupeng1989/react-todo-list)

---

## 总结

| 特性 | React |
|------|-------|
| 组件 | JS 函数 |
| 属性 | 函数参数 |
| 事件 | 函数参数，执行事件就是执行 JS 函数 |
| JSX | 规则很少，更简洁 |
| 条件/列表渲染 | 完全使用 JS 语法 |
| useState | 语义明确，没有心智负担 |
| state | 原始数据，没有任何包裹处理 |
| useEffect | 涵盖所有副作用操作，API 更简洁 |
| 表单 | 受控组件比 Vue v-model 多写几行代码，但没有其他心智负担 |

**React 设计的很简洁，React 很简单！**

> 原文出处：[掘金 - 只会 Vue 不会 React ？22 点证明 React 比 Vue3 更简单](https://juejin.cn/post/7344536653463207973)

```jsx
useEffect(() => {
  // 组件渲染后执行

  return () => {
    console.log('组件销毁之前，如解绑自定义事件')
  }
})
```
    { id: 1, title: '任务1' },
    { id: 2, title: '任务2' },
    { id: 3, title: '任务3' },
  ])

  return (
    <ul>
      {list.map(item => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  )
}
```

> PS：`key` 是必须的，和 Vue 的 `:key` 一样，用于 Diff 算法优化。
```