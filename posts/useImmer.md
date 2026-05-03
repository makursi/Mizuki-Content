---
title: useImmer
published: 2026-05-02
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](./react/reactcute.webp)

---
# useImmer

`useImmer` 是 Immer 库为 React 提供的自定义 Hook，它基于 Immer 核心的 "可变式操作不可变数据" 理念，让开发者可以以直观的、可变的方式修改状态，同时底层自动生成不可变的新状态，无需手动解构或拼接对象/数组，大幅简化 React 中复杂状态的更新逻辑。

Immer 本身是由 React 核心团队成员开发的库，`useImmer` 是其针对 React 状态管理场景的专属封装，兼容 React 函数组件的 Hook 规范，常作为 `useState` 的替代方案用于处理嵌套/复杂结构的状态。

# useImmer的核心作用

1. **简化不可变状态更新**：解决 React 中使用 `useState` 更新嵌套对象/数组时，需要手动展开（如 `...` 扩展运算符）、逐层复制的繁琐问题，通过 "草稿（draft）" 对象的可变操作，自动生成不可变的新状态。
2. **保持状态不可变性**：开发者操作的是草稿对象（临时可变副本），原状态不会被直接修改，最终输出的新状态仍符合 React 不可变数据的设计原则，避免因直接修改状态导致的渲染异常。
3. **降低代码复杂度**：对于多层嵌套的状态（如 `{ user: { info: { address: [...] } } }`），无需嵌套的扩展运算符，只需像操作普通可变对象一样修改草稿，大幅提升代码可读性和可维护性。
4. **兼容 `useState` 用法**：`useImmer` 的基础用法与 `useState` 高度一致，学习成本低，可平滑替换，同时支持函数式更新逻辑。

# useImmer的使用方法

### 第一步：安装依赖
首先需安装 Immer 库（`useImmer` 包含在 immer 包中）：
```bash
npm install immer --save
# 或
yarn add immer
```

### 第二步：基础用法
```jsx
import { useImmer } from 'immer';

function App() {
  // 初始化状态，用法与 useState 一致
  const [state, setState] = useImmer({
    name: '张三',
    age: 20,
    hobbies: ['读书', '运动']
  });

  // 修改状态：通过可变方式操作 draft 草稿对象
  const updateUser = () => {
    setState((draft) => {
      // 直接修改草稿对象，无需手动复制/解构
      draft.age += 1;
      draft.hobbies.push('编程'); // 数组直接 push，无需 [...hobbies, '编程']
      draft.name = '李四';
    });
  };

  return (
    <div>
      <p>姓名：{state.name}</p>
      <p>年龄：{state.age}</p>
      <p>爱好：{state.hobbies.join(', ')}</p>
      <button onClick={updateUser}>更新信息</button>
    </div>
  );
}
```

### 第三步：嵌套状态更新（核心优势场景）
```jsx
import { useImmer } from 'immer';

function App() {
  const [user, setUser] = useImmer({
    info: {
      basic: {
        name: '张三',
        address: {
          province: '北京',
          city: '朝阳区'
        }
      },
      scores: {
        math: 90,
        english: 85
      }
    }
  });

  const updateAddress = () => {
    setUser((draft) => {
      // 直接修改嵌套层级的属性，无需多层扩展运算符
      draft.info.basic.address.city = '海淀区';
      draft.info.scores.math = 95;
    });
  };

  return (
    <div>
      <p>城市：{user.info.basic.address.city}</p>
      <p>数学成绩：{user.info.scores.math}</p>
      <button onClick={updateAddress}>更新地址和成绩</button>
    </div>
  );
}
```

### 第四步：替换式更新（非函数式）
与 `useState` 类似，也可直接传入新状态（但失去 Immer 核心优势，仅兼容场景）：
```jsx
setUser({
  ...user,
  info: {
    ...user.info,
    basic: {
      ...user.info.basic,
      name: '王五'
    }
  }
});
```

# useImmer的使用案例

### 案例1：表单状态管理
处理多字段、嵌套结构的表单时，简化状态更新：
```jsx
import { useImmer } from 'immer';

function UserForm() {
  const [form, setForm] = useImmer({
    basic: {
      username: '',
      email: '',
      phone: ''
    },
    preferences: {
      receiveEmail: true,
      theme: 'light'
    },
    skills: []
  });

  // 输入框 onChange 处理
  const handleInputChange = (field, value) => {
    setForm((draft) => {
      // 动态修改不同层级的表单字段
      const [level1, level2] = field.split('.');
      if (level2) {
        draft[level1][level2] = value;
      } else {
        draft[level1] = value;
      }
    });
  };

  // 添加技能
  const addSkill = (skill) => {
    setForm((draft) => {
      draft.skills.push(skill);
    });
  };

  return (
    <div>
      <input
        placeholder="用户名"
        value={form.basic.username}
        onChange={(e) => handleInputChange('basic.username', e.target.value)}
      />
      <input
        placeholder="邮箱"
        value={form.basic.email}
        onChange={(e) => handleInputChange('basic.email', e.target.value)}
      />
      <button onClick={() => addSkill('React')}>添加React技能</button>
      <button onClick={() => setForm((d) => (d.preferences.theme = 'dark'))}>切换深色主题</button>
    </div>
  );
}
```

### 案例2：列表数据操作（增删改）
管理表格/列表类数据时，简化行数据的修改、删除：
```jsx
import { useImmer } from 'immer';

function TodoList() {
  const [todos, setTodos] = useImmer([
    { id: 1, text: '学习React', completed: false },
    { id: 2, text: '掌握useImmer', completed: false }
  ]);

  // 切换待办完成状态
  const toggleTodo = (id) => {
    setTodos((draft) => {
      const todo = draft.find((t) => t.id === id);
      if (todo) todo.completed = !todo.completed;
    });
  };

  // 删除待办
  const deleteTodo = (id) => {
    setTodos((draft) => {
      const index = draft.findIndex((t) => t.id === id);
      if (index !== -1) draft.splice(index, 1); // 直接 splice，无需 filter
    });
  };

  // 添加待办
  const addTodo = (text) => {
    setTodos((draft) => {
      draft.push({ id: Date.now(), text, completed: false });
    });
  };

  return (
    <div>
      <button onClick={() => addTodo('新的待办事项')}>添加待办</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => toggleTodo(todo.id)}>切换状态</button>
            <button onClick={() => deleteTodo(todo.id)}>删除</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

# useImmer的使用注意事项

1. **依赖安装**：需确保安装 `immer` 包（版本 >= 9.0.0 推荐），低版本可能存在兼容性问题，且 `useImmer` 需从 `immer` 包中导入（而非单独的 `react-immer`，该包已废弃）。
2. **草稿对象仅在 setState 回调内有效**：`draft` 是临时对象，仅能在 `setState(draft => { ... })` 的回调函数内操作，不可将其赋值给外部变量或异步使用，否则会导致状态更新异常。
   ```jsx
   // 错误示例
   let tempDraft;
   setState((draft) => {
     tempDraft = draft;
     draft.name = '张三';
   });
   // 异步操作中使用 tempDraft 无效，且会破坏不可变性
   setTimeout(() => {
     tempDraft.age = 30; // 无任何效果，且可能引发内存问题
   }, 1000);
   ```
3. **避免直接返回新对象**：在 `setState` 回调中，若直接返回新对象（而非修改 draft），Immer 会退化为普通 `useState` 行为，失去其核心价值：
   ```jsx
   // 不推荐（失去 Immer 优势）
   setState((draft) => {
     return { ...draft, age: 30 };
   });
   // 推荐
   setState((draft) => {
     draft.age = 30;
   });
   ```
4. **数组操作注意事项**：对数组 draft 使用 `pop`/`push`/`splice`/`sort`/`reverse` 等可变方法是安全的（Immer 会追踪修改），但需注意：`sort` 和 `reverse` 会直接修改草稿数组的顺序，若需保留原数组顺序，可先复制再操作（但 Immer 场景下无需额外处理）。
5. **与 React 并发模式兼容**：Immer 及 `useImmer` 完全兼容 React 并发渲染（Concurrent Mode），状态更新逻辑不会因并发渲染导致数据不一致。
6. **性能考量**：`useImmer` 的性能与手动使用扩展运算符的 `useState` 接近，甚至在复杂嵌套状态下更优（因 Immer 内部采用结构化共享）；但避免在每次渲染时创建大量临时状态，建议将状态更新逻辑抽离为函数。
7. **不可修改原状态**：即使使用 `useImmer`，也不可直接修改 `state` 本身（如 `state.age = 30`），必须通过 `setState` 操作 draft 对象，否则会违反 React 不可变状态原则，导致组件不重新渲染。
8. **TypeScript 支持**：使用 TypeScript 时，`useImmer` 可自动推导状态类型，无需额外定义；若需手动指定类型，可通过泛型：
   ```tsx
   interface User {
     name: string;
     age: number;
   }
   const [user, setUser] = useImmer<User>({ name: '张三', age: 20 });
   ```
---
