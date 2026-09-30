# React 入门介绍

> 课程资料 · 阅读时间约 20 分钟
> 适合有 HTML / CSS / JavaScript 基础、第一次接触 React 的同学。

## 1. React 是什么

React 是一个用于构建用户界面的 JavaScript 库，由 Meta（原 Facebook）在 2013 年开源。它的核心思想是：

**界面是数据的函数**。给定同样的数据（state），就渲染出同样的界面；数据变了，React 负责把界面上变化的部分更新掉。

它只解决「视图层」这一件事，所以常和路由（React Router）、状态管理、构建工具一起组合使用，构成一个完整的前端应用。

### 为什么用 React

- **组件化**：把页面拆成一个个可复用、可独立维护的组件。
- **声明式**：你描述「界面应该长什么样」，而不是用 `document.createElement` 一步步手动操作 DOM。
- **生态成熟**：社区庞大，遇到问题基本都能找到现成方案。

## 2. 环境搭建

推荐使用官方脚手架 Vite 创建项目（Node.js 需 18 及以上）：

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

启动后浏览器打开终端提示的地址（通常是 `http://localhost:5173`），即可看到初始页面。

## 3. JSX：在 JavaScript 里写标签

JSX 是 React 推荐使用的语法扩展，看起来像 HTML，实际会被编译成 JavaScript 函数调用。

```jsx
const element = <h1>Hello, React</h1>;
```

和 HTML 相比需要注意：

- `class` 要写成 `className`，`for` 要写成 `htmlFor`。
- 标签必须闭合，如 `<img />`、`<br />`。
- 一个组件只能返回**一个**根节点，多个元素需要用一个父标签或 Fragment（`<>...</>`）包起来。
- 花括号 `{}` 里可以写任意 JavaScript 表达式：`<p>{1 + 2}</p>`、`<p>{user.name}</p>`。

## 4. 组件：界面的积木

组件就是一个返回 JSX 的函数，**函数名必须大写开头**，这样 React 才能区分它是组件而不是普通 HTML 标签。

```jsx
function Welcome() {
  return <h1>欢迎来到 React</h1>;
}

function App() {
  return (
    <>
      <Welcome />
      <Welcome />
    </>
  );
}
```

### Props：从外部传入的数据

Props 是父组件传给子组件的参数，**只读，不能在子组件里修改**。

```jsx
function Welcome({ name, level = '同学' }) {
  return <p>你好，{name}（{level}）</p>;
}

// 使用
<Welcome name="小明" level="初级" />
```

### State：组件内部会变化的数据

State 是组件自己的记忆，通过 `useState` 声明，修改时必须调用 set 函数，不能直接赋值。

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      点了 {count} 次
    </button>
  );
}
```

**Props 与 State 的区别**：Props 由外部决定，State 由组件自己维护。

## 5. 事件处理

React 事件用驼峰命名，传入函数而不是字符串。

```jsx
function Form() {
  const [text, setText] = useState('');

  function handleSubmit(e) {
    e.preventDefault(); // 阻止表单默认提交行为
    console.log('提交内容：', text);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">提交</button>
    </form>
  );
}
```

`value` + `onChange` 的写法称为**受控组件**，表单的值由 React 状态统一管理。

## 6. 条件渲染与列表渲染

```jsx
function TodoList({ todos, isLoading }) {
  if (isLoading) return <p>加载中…</p>;

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

- 条件渲染常用三元表达式 `条件 ? A : B` 或 `条件 && A`。
- 列表渲染要用 `map`，并为每一项提供唯一的 `key`，帮助 React 高效地识别增删改。

## 7. 常用 Hooks

| Hook | 作用 |
| --- | --- |
| `useState` | 声明组件状态 |
| `useEffect` | 处理副作用，如请求数据、订阅事件 |
| `useRef` | 保存不触发渲染的值，或引用 DOM 节点 |
| `useContext` | 读取跨层级共享的数据 |
| `useMemo` / `useCallback` | 缓存计算结果或函数，优化性能 |

`useEffect` 示例（组件挂载后请求一次数据）：

```jsx
import { useEffect, useState } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => {
        if (!cancelled) setUser(data);
      });

    return () => {
      cancelled = true; // 清理：组件卸载或依赖变化时避免写入过期数据
    };
  }, [userId]);

  if (!user) return <p>加载中…</p>;
  return <h2>{user.name}</h2>;
}
```

> 有两条规则要牢记：Hooks 只能在组件或自定义 Hook 的**顶层**调用；不要在循环、条件或嵌套函数里调用。

## 8. 数据的流动方向

React 的数据默认**自上而下**流动：父组件通过 Props 把数据传给子组件，子组件通过调用父组件传下来的回调函数把变化「上报」回去。

```jsx
function Parent() {
  const [value, setValue] = useState('');
  return <Child value={value} onChange={setValue} />;
}

function Child({ value, onChange }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
```

当组件层级很深、状态需要多处共享时，再考虑使用 Context 或专门的状态管理库。

## 9. 继续学习的方向

- **路由**：React Router，实现多页面导航。
- **状态管理**：Context、Zustand、Redux。
- **数据请求**：TanStack Query、SWR。
- **样式方案**：CSS Modules、Tailwind CSS、CSS-in-JS。
- **服务端渲染**：Next.js、Remix。
- **官方文档**：<https://react.dev>（有简体中文版，建议作为第一手资料）。

## 10. 动手练习

1. 用 Vite 创建一个 React 项目，把默认页面改成一个「个人名片」组件。
2. 实现一个计数器，包含「加一」「减一」「重置」三个按钮。
3. 实现一个待办清单：可以添加任务、勾选完成、删除任务，并统计未完成数量。
4. 用 `useEffect` 从公开接口（如 `https://jsonplaceholder.typicode.com/users`）拉取用户列表并渲染，处理加载中和出错状态。

## 11. 常见坑

- 直接修改 state：`count++` 不会触发更新，必须用 `setCount(count + 1)`；更新对象或数组时也要创建新对象。
- 忘记 `key` 或使用数组下标作为 `key`，导致列表更新出错。
- 在 `useEffect` 中漏写依赖项，造成读到旧值或重复请求。
- 组件名小写开头，React 会把它当成普通 HTML 标签。
