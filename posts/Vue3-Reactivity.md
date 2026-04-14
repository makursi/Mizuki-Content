---
title: Vue3-响应式原理
published: 2026-04-08
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---

# Vue2 vs Vue3 响应式
- Vue2：**Object.defineProperty**，属性级劫持
  - 缺陷：对象新增属性需`Vue.set`；数组索引/`length`修改无法监听；初始化递归全量遍历性能差。
- Vue3：**ES6 Proxy + Reflect**，对象级代理
  - 优势：天然支持新增/删除属性、数组索引与`length`；**懒递归**（访问时才代理嵌套对象），性能更优。

# Vue3响应式核心四件套实现

## 1. reactive：创建响应式代理
- 用`Proxy`拦截`get/set`，配合`Reflect`保证this指向正确
- `get`：执行`track`收集依赖；值为对象则**递归reactive**实现深层响应式
- `set`：执行`trigger`触发更新，返回修改结果

代码:

    export const reactive = <T extends object>(target)=>{
    return new Proxy(target,{
    // target → 原始对象
    // key → 你读的属性名（如 'name'）
    // receiver → 代理对象自己（保证 this 不乱跑
    
      get(target,key,receiver){
        const res = Reflect.get(target, key , receiver) as object
        
        return res
      }
    }) 
    
    // target → 原始对象
    // key → 你改的属性名
    // value → 新值
    // receiver → 代理对象自己
    
    set(target,key,value,receiver){
      const res = Reflect.set(target,key,value,receiver)
      return res
    }
    }


## 2. effect：副作用函数注册
- 全局`activeEffect`标记当前执行的副作用
- 立即执行一次`fn`，触发内部响应式数据的`get`，完成依赖绑定
代码:

      let activeEffect;// 作用:存放当前正在运行的函数
      export const effect = (fn:Function)=>{// fn为你想让它自动更新的函数
      
      const _effect = function(){
        activeEffect = _effect; // 标记当前运行函数
        fn() // 执行业务逻辑,会读取响应式数据触发拦截器
      }
      _effect()
      }

## 3. track：依赖收集
- 数据结构：`WeakMap(target → Map(key → Set<effect>))`
- 读取属性时，将`activeEffect`存入对应属性的依赖集合
代码:

    const targetMap = new WeakMap() // 存储所有对象的响应式依赖
    export const track = (target,key) =>{ // target:当前访问的对象 key:当前访问对象的属性名
      let depsMap = targetMap.get(target)
      if(!depsMap){
          depsMap = new Map() // 存属性,用到这个属性的函数
          targetMap.set(target,depsMap)
      }
      let deps = depsMap.get(key)
      if(!deps){
          deps = new Set()// 存函数(effect)
          depsMap.set(key,deps)
      }
    
      deps.add(activeEffect)
    }


## 4. trigger：触发更新
- 修改属性时，取出该属性所有`effect`并执行，驱动视图/逻辑更新
代码:

    export const trigger = (target,key)=>{
      const depsMap = targetMap.get(target)// 收集对象的依赖
      const deps = depsMap.get(key)// 收集对象属性的依赖
      deps.forEach((effect)=>{
      effect()// 重新执行
      })
    }

# 5.递归实现reactive

    import { track, trigger } from './effect'
    
    const isObject = (target) => target != null && typeof target == 'object'
    
    export const reactive = <T extends object>(target: T) => {
        return new Proxy(target, {
            get(target, key, receiver) {
                const res = Reflect.get(target, key, receiver) as object
    
                track(target, key)// 收集依赖
    
                if (isObject(res)) {
                    return reactive(res)
                }
    
                return res
            },
            set(target, key, value, receiver) {
                const res = Reflect.set(target, key, value, receiver)
    
                trigger(target, key)// 触发更新
    
                return res
            }
        })
    }
