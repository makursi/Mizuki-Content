---
title: 优质React hook开源库
published: 2026-05-02
description: ''
image: ''
tags: []
category: 'React'
draft: false 
lang: ''
---
![Alt text](../react/reactcute.webp)

---

## 一、ahooks（阿里开源，企业级通用 Hook 库）
- **GitHub**：https://github.com/alibaba/hooks
- **Star**：18k+
- **核心亮点**
  - 国内首选，**中文文档完善**，阿里线上大规模验证。
  - 60+ 个常用 Hook，分类清晰（状态、副作用、DOM、传感器等）。
  - 内置强大的 `useRequest`（请求缓存、节流、轮询、取消）。
  - **TS 全量类型**、SSR 友好、支持 Tree-shaking。
- **常用 Hook**：`useRequest`、`useLocalStorageState`、`useWindowSize`、`useDebounce`、`useVirtualList`
- **示例（useRequest）**
```jsx
import { useRequest } from 'ahooks';

function User() {
  const { data, loading, error } = useRequest(() =>
    fetch('/api/user').then(res => res.json())
  );
  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;
  return <div>用户名：{data?.name}</div>;
}
```
- **适用场景**：中后台系统、企业级应用、国内团队、需要请求/状态管理一体化。

---

## 二、react-use（经典全能型，生态最丰富）
- **GitHub**：https://github.com/streamich/react-use
- **Star**：35k+
- **核心亮点**
  - 老牌库，**覆盖最广**（90+ Hook），浏览器 API 集成度高。
  - 分类：Browser、State、Element、Effect、Sensor、Lifecycle 等。
  - SSR 兼容、TS 支持、Tree-shaking、文档有在线 Demo。
- **常用 Hook**：`useLocalStorage`、`useMedia`、`useWindowScroll`、`useHover`、`useKey`
- **示例（useLocalStorage）**
```jsx
import { useLocalStorage } from 'react-use';

function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      切换主题：{theme}
    </button>
  );
}
```
- **适用场景**：复杂交互、需要大量浏览器 API 封装、开源项目、技术栈偏国际化。

---

## 三、@react-hookz/web（现代企业级，强类型+高性能）
- **GitHub**：https://github.com/react-hookz/web
- **Star**：4k+
- **核心亮点**
  - 由 react-use 前维护者开发，**设计更现代、性能更优**。
  - 60+ Hook，**TS 优先**，类型定义精准，杜绝 `any`。
  - 内置 `useAsync`（异步状态管理）、`useLocalStorageValue`（强类型存储）。
  - SSR 友好、零依赖、轻量。
- **常用 Hook**：`useAsync`、`useLocalStorageValue`、`useMediaQuery`、`useEventListener`
- **示例（useAsync）**
```jsx
import { useAsync } from '@react-hookz/web';

function Data() {
  const { result, error, status } = useAsync(async () => {
    const res = await fetch('/api/data');
    return res.json();
  }, []);

  if (status === 'loading') return <div>加载中...</div>;
  if (status === 'error') return <div>错误：{error.message}</div>;
  return <div>数据：{JSON.stringify(result)}</div>;
}
```
- **适用场景**：追求类型安全、高性能、现代架构的中大型项目。

---

## 四、react-hook-form（表单专用，性能之王）
- **GitHub**：https://github.com/react-hook-form/react-hook-form
- **Star**：40k+
- **核心亮点**
  - **表单管理首选**，极致性能（减少重渲染）、体积小（5KB）。
  - 内置校验、支持复杂表单、嵌套字段、表单数组。
  - 零依赖、支持 React Native、TS 友好、SSR 兼容。
- **常用 Hook**：`useForm`、`useController`、`useWatch`、`useFormContext`
- **示例（基础表单）**
```jsx
import { useForm } from 'react-hook-form';

function Login() {
  const { register, handleSubmit, formState: { errors } } = useForm();
  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: '邮箱必填' })} placeholder="邮箱" />
      {errors.email && <p>{errors.email.message}</p>}
      <input {...register('password', { required: '密码必填' })} type="password" placeholder="密码" />
      {errors.password && <p>{errors.password.message}</p>}
      <button type="submit">登录</button>
    </form>
  );
}
```
- **适用场景**：所有表单场景（登录、注册、中后台表单、复杂表单）。

---

## 五、@tanstack/react-query（服务端状态专用，数据请求之王）
- **GitHub**：https://github.com/TanStack/query
- **Star**：35k+
- **核心亮点**
  - **服务端状态管理首选**，处理异步数据、缓存、同步、错误处理。
  - 自动缓存、自动重试、后台更新、乐观更新、分页/无限滚动支持。
  - 零依赖、TS 友好、SSR/SSG 兼容、DevTools 调试。
- **常用 Hook**：`useQuery`、`useMutation`、`useInfiniteQuery`、`useQueryClient`
- **示例（数据请求）**
```jsx
import { useQuery } from '@tanstack/react-query';

function Posts() {
  const { data, loading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: () => fetch('https://jsonplaceholder.typicode.com/posts').then(res => res.json())
  });

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;
  return <ul>{data?.map(post => <li key={post.id}>{post.title}</li>)}</ul>;
}
```
- **适用场景**：所有服务端数据请求场景（列表、详情、分页、无限滚动、数据同步）。

---
