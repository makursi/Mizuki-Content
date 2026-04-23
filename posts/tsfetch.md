---
title: TS+FetchAPI的前端请求标准
published: 2026-04-01
description: ''
image: ''
tags: []
category: ''
draft: false 
lang: ''
---
# 前端 Fetch API 严格专业规范（TypeScript + 企业级标准）

目标：
- 严格遵循 TS 类型安全
- 彻底解决 Fetch 所有原生缺陷
- 避免网络、超时、状态码、解析、参数、跨域、重复请求等所有错误
- 写出**专业、可维护、无隐患**的前端请求代码

---

# 一、Fetch 官方核心缺陷（必须先解决）
根据 **MDN 官方定义**，Fetch 有 4 个 **天生缺陷**，不处理 = 必出 bug：

1. **HTTP 错误（4xx/5xx）不会进入 catch**
   → 只在**网络断开、请求被取消**时 reject
2. **无内置超时机制**
   → 请求会一直挂起，导致页面卡死
3. **默认不携带 Cookie**
   → 登录态、权限直接失效
4. **错误类型模糊**
   → 无法区分“超时/网络异常/业务错误”

---

# 二、专业级标准封装（TypeScript 严格版）
这是**前端工程化标准请求层**，所有项目必须使用，禁止直接写 `fetch()`

## 1. 类型定义（TS 官方规范）
```ts
// src/api/request.ts
// --------------- 类型定义（严格遵循 TS 规范）---------------
interface ApiResponse<T = any> {
  code: number;
  message: string;
  data: T;
}

type RequestOptions = RequestInit & {
  timeout?: number;
  headers?: HeadersInit;
};
```

## 2. 全局配置（专业固定配置）
```ts
// 基础配置（统一管理）
const BASE_URL = import.meta.env.VITE_API_URL || '/api';
const DEFAULT_TIMEOUT = 10 * 1000; // 10秒（行业标准）
const DEFAULT_HEADERS = {
  'Content-Type': 'application/json',
  Accept: 'application/json',
};
```

## 3. 核心请求方法（专业、严谨、无错）
```ts
/**
 * 严格专业的 Fetch 请求封装（TS + 企业标准）
 * @param url 请求地址
 * @param options 请求配置
 * @returns Promise<ApiResponse<T>>
 */
export async function request<T = any>(
  url: string,
  options: RequestOptions = {}
): Promise<ApiResponse<T>> {
  const {
    timeout = DEFAULT_TIMEOUT,
    headers = {},
    credentials = 'include', // 强制带Cookie（专业规范）
    ...restOptions
  } = options;

  // 1. 超时控制器（官方标准方案）
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeout);

  try {
    // 2. 发起请求
    const response = await fetch(`${BASE_URL}${url}`, {
      headers: { ...DEFAULT_HEADERS, ...headers },
      signal: controller.signal,
      credentials, // 必传，解决登录态
      ...restOptions,
    });

    clearTimeout(timer);

    // 3. HTTP 状态码判断（MDN 推荐写法）
    if (!response.ok) {
      let errorMsg = response.statusText;
      try {
        const errData = await response.json();
        errorMsg = errData.message || errorMsg;
      } catch {}

      // 抛出标准化错误
      throw new Error(`[${response.status}] ${errorMsg}`);
    }

    // 4. 安全解析 JSON（防止后端返回非 JSON 崩溃）
    const data: ApiResponse<T> = await response.json().catch(() => {
      throw new Error('数据解析失败');
    });

    // 5. 业务状态码判断（前后端统一规范）
    if (data.code !== 200) {
      throw new Error(data.message || '业务请求失败');
    }

    return data;
  } catch (err) {
    clearTimeout(timer);
    // 6. 标准化错误处理（专业区分错误类型）
    handleFetchError(err as Error);
    throw err; // 向上抛出，让调用层可继续处理
  }
}
```

## 4. 错误统一处理（避免所有异常崩溃）
```ts
// 统一错误处理（专业前端必备）
function handleFetchError(err: Error) {
  if (err.name === 'AbortError') {
    console.error('请求超时');
    alert('请求超时，请稍后重试');
  } else if (err.message.includes('NetworkError')) {
    console.error('网络异常');
    alert('网络连接失败，请检查网络');
  } else {
    console.error('请求错误：', err.message);
    alert(err.message);
  }
}
```

## 5. 快捷方法（RESTful 规范）
```ts
// GET 请求（参数自动拼接，避免手写字符串错误）
export function get<T>(url: string, params?: Record<string, any>) {
  const queryStr = params ? new URLSearchParams(params).toString() : '';
  return request<T>(queryStr ? `${url}?${queryStr}` : url, { method: 'GET' });
}

// POST 请求
export function post<T>(url: string, body?: any) {
  return request<T>(url, { method: 'POST', body: JSON.stringify(body) });
}

// PUT 请求
export function put<T>(url: string, body?: any) {
  return request<T>(url, { method: 'PUT', body: JSON.stringify(body) });
}

// DELETE 请求
export function del<T>(url: string) {
  return request<T>(url, { method: 'DELETE' });
}
```

---

# 三、严格专业的使用规范（必须遵守）
## 1. 调用必须使用 `try/catch`（TS 强制规范）
```ts
import { get, post } from '@/api/request';

async function getUserList() {
  try {
    const res = await get('/user/list', { page: 1, pageSize: 10 });
    console.log(res.data);
  } catch (err) {
    // 错误已在封装层处理，这里可做额外逻辑
  }
}
```

## 2. 必须防重复请求（专业必备）
```ts
let loading = false;

async function submitData() {
  if (loading) return; // 拦截重复
  loading = true;

  try {
    await post('/user/create', { name: 'test' });
  } finally {
    loading = false;
  }
}
```

## 3. 必须加 Loading（用户体验标准）
```ts
async function fetchData() {
  showLoading();
  try {
    const res = await get('/data');
  } finally {
    hideLoading(); // 无论成败都关闭
  }
}
```

---

# 四、所有基础错误 + 专业规避方案（完整版）
## 1. 400/401/403/404/500 不进 catch
**原因**：Fetch 官方设计如此
**解决**：判断 `!response.ok` 并手动抛错

## 2. 请求永不超时
**解决**：`AbortController` + 定时器

## 3. 跨域登录态失效
**解决**：固定 `credentials: 'include'`

## 4. JSON 解析崩溃
**解决**：`response.json().catch()` 捕获解析错误

## 5. GET 参数拼接错误
**解决**：使用 `URLSearchParams`

## 6. 前端直接写死请求地址
**解决**：统一 `BASE_URL`

## 7. 未处理后端业务错误
**解决**：判断 `code !== 200` 抛错

## 8. 按钮重复点击发送多次请求
**解决**：`loading` 状态锁

## 9. 没有 Loading，用户体验差
**解决**：`try/finally` 控制 Loading

## 10. 错误不提示，用户不知道发生什么
**解决**：统一错误提示系统

---

# 五、TypeScript 官方推荐的最佳实践
1. **所有接口必须定义返回类型**
   ```ts
   interface User {
     id: number;
     name: string;
   }
   const res = await get<User[]>('/user/list');
   ```
2. **禁止 any 泛滥**
3. **请求层统一封装，禁止业务代码直接使用 fetch**
4. **错误必须标准化**
5. **所有异步必须使用 try/catch**

---

# 六、最终 10 条严格专业规则
1. **禁止直接使用原生 fetch**
2. **必须统一封装请求**
3. **必须带超时**
4. **必须带 Cookie**
5. **必须处理 HTTP 错误**
6. **必须处理 JSON 解析错误**
7. **必须处理业务状态码**
8. **必须防重复请求**
9. **必须加 Loading**
10. **必须使用 TypeScript 类型约束**
