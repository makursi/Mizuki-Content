---
title: 组件CSS预处理架构
published: 2026-04-08
description: ''
image: ''
tags: []
category: 'Vue'
draft: false 
lang: ''
---

# BEM架构

## 一、BEM 是什么
**BEM** 是 Yandex 推出的一套 **CSS 命名规范 & 前端组件化架构思想**
全称：**Block / Element / Modifier**
用来解决：CSS 样式混乱、命名随意、层级嵌套过深、样式污染、多人协作冲突。

---

## 二、三大核心概念（官方定义）
### 1. Block 块
> 独立、可复用、不依赖其他模块的**独立组件**
- 作用：页面独立功能模块（头部、按钮、卡片、弹窗）
- 规则：
  - 自身语义独立，不依赖父元素
  - 命名：纯单词，短横线连接
  - 示例：`.header`、`.card`、`.btn`

### 2. Element 元素
> 属于某个 Block 内部、**无法单独复用**的子节点
- 作用：块的组成部分，脱离块就无意义
- 命名规则：`块名__元素名`（**双下划线**）
- 示例：
  `.card__title`、`.card__desc`、`.btn__icon`

### 3. Modifier 修饰符
> 用来描述 **块/元素 的状态、样式变体**
- 作用：区分不同形态、状态（禁用、高亮、尺寸、主题）
- 命名规则：`块名/元素名--修饰符`（**双短横线**）
- 示例：
  `.btn--primary`、`.btn--disabled`、`.card--dark`

---

## 三、标准命名格式
```css
/* Block 块 */
.block {}

/* Block 内 Element 元素 */
.block__element {}

/* 修饰符：块的变体 */
.block--modifier {}

/* 修饰符：元素的变体 */
.block__element--modifier {}
```

示例：
```html
<div class="card card--dark">
  <h3 class="card__title">标题</h3>
  <p class="card__desc card__desc--gray">描述</p>
</div>
```

---

## 四、关键核心作用
1. **彻底解决 CSS 全局污染**
所有样式都捆绑在独立块命名下，不会出现类名重复冲突。

2. **扁平化 CSS，杜绝深层嵌套**
告别 `div ul li a` 多层嵌套，选择器权重统一、更好维护。

3. **语义化命名，可读性极强**
看类名就知道：属于哪个模块、是什么角色、什么状态，协作一目了然。

4. **组件高可复用、易扩展**
新增样式只需要加**修饰符**，不改动原有基础样式，符合开闭原则。

5. **统一团队编码规范**
多人开发、大型项目统一书写规则，降低沟通、维护成本。

---

# CSS预处理器的使用

## 1. 基础概念
**CSS 预处理器**：
是一种**拥有编程语法、拓展能力**的专属样式语言，**不属于原生 CSS**，需要经过「编译」转换成浏览器可识别的原生 CSS 才能运行。
> 原生 CSS 是**标记语言、无逻辑、无变量、无复用**；
> 预处理器给 CSS 增加了**编程能力**，让样式开发工程化。

## 2. 核心本质
- 写：**Less / Sass / Stylus** 语法
- 编译：构建工具（Vite/Webpack）自动转 原生CSS
- 运行：浏览器只识别最终编译后的 CSS

---

# 二、CSS 预处理器 核心概念
1. **变量**
支持定义颜色、尺寸、间距、主题变量，统一管理、全局修改。
2. **嵌套语法**
支持层级嵌套，和 HTML 结构对应，减少重复书写选择器。
3. **代码复用**
混合、继承、引入、抽离公共样式，消除冗余代码。
4. **逻辑运算**
支持四则运算、条件判断、循环、函数，动态生成样式。
5. **模块化拆分**
支持文件拆分、按需导入，大型项目分模块管理样式。
6. **作用域控制**
配合架构规范（BEM），降低全局样式污染。

---

# 三、关键作用（解决原生 CSS 痛点）
1. **解决原生 CSS 无变量问题**
原生无法统一维护主题色、圆角、字体；预处理器全局统一管控。
2. **解决选择器冗余、层级混乱**
嵌套写法，结构清晰，告别重复书写长选择器。
3. **解决样式无法复用**
原生重复写 flex 居中、清除浮动、圆角；预处理器通过混合一键复用。
4. **解决大型项目难以维护**
样式拆分文件、模块化开发，多人协作更规范。
5. **增强动态能力**
尺寸计算、循环生成布局、条件判断主题样式，原生 CSS 做不到。
6. **兼容下沉**
编译阶段自动处理部分兼容写法，减少手写兼容代码。

---

# 四、主流 CSS 预处理器 分类 & 语法区别
## 1. Less
- 诞生：2009年，阿里主推、国内老旧项目存量极大
- 语法：**完全兼容原生 CSS**，上手最简单
- 核心标识：`@变量`、`@mixin`
- 特点：弱编程、轻量、学习成本低
```less
@primary: #1677ff;
.box { color: @primary; }
```

## 2. Sass / SCSS
- 诞生：2006年，国外主流、现代项目首选
- 两种格式：
  - `sass`：缩进式语法（无大括号、无分号）
  - `scss`：完全兼容 CSS，企业通用标准
- 核心标识：`$变量`、`@include`、功能最强
- 特点：完整编程能力（循环/条件/函数）、生态最强
```scss
$primary: #1677ff;
.box { color: $primary; }
```

## 3. Stylus
- 诞生：前端早期流行，移动端旧项目多见
- 语法极简：可省略大括号、分号、冒号
- 现状：**基本淘汰**，新项目不再使用

> 总结：
> - 老旧项目/阿里系：**Less**
> - 新项目/跨框架/大厂通用：**SCSS(Sass)**
> - Stylus 已过时，不用学

---

# 五、配套三方库 / 框架 / 工程化支持
## 1. 构建工具原生支持
- **Vite**：内置支持 Less / SCSS，零配置直接用
- **Webpack**：需安装 loader 编译
  - Less：`less + less-loader`
  - Sass：`sass + sass-loader`
- **Vue / React / Next / Nuxt**：全部原生适配预处理器

## 2. 成熟 UI 框架底层依赖
1. **Ant Design**：底层基于 **Less** 开发
2. **Element Plus / Element UI**：底层基于 **SCSS**
3. **Vant / TDesign**：SCSS 开发，支持主题定制
4. **Tailwind CSS**：可结合 SCSS 变量自定义主题

## 3. 全局主题定制库
- 基于预处理器变量，实现**一键换肤、主题切换**
- 例如：组件库通过修改 `scss/less 变量` 定制主题色、圆角、间距

## 4. 工具类预设库
- `sass-resources-loader`：全局注入通用变量、混合，无需每个文件单独导入

---

# 六、补充：现代方案对比
现在除了传统预处理器，还有两类替代方案：
1. **CSS Variables（原生变量）**
浏览器原生支持，运行时动态切换，无编译成本，但无嵌套/混合能力
2. **CSS-in-JS（Styled-Components）**
JS 写样式，React 生态常用，脱离 CSS 文件体系
3. **原子化 CSS（Tailwind）**
弱化预处理器，靠工具类写样式

> 企业现状：
> 中大型项目依旧**SCSS / Less 为主流**，是前端必备技能。

---

# Sass的基本使用
---

## 一、Sass 是什么？
**Sass = CSS 的超集** → 让 CSS 拥有**变量、嵌套、混合、函数、模块化**等编程能力。
最终会**编译成普通 CSS**运行在浏览器。

---

## 二、核心概念（4 个必懂）
### 1. 变量（Variables）
把常用值存起来，统一修改。

### 2. 嵌套（Nesting）
像 HTML 结构一样写 CSS，**不用重复写父选择器**。

### 3. 混合（Mixins）
可复用的 CSS 代码片段，像函数一样调用。

### 4. 模块化（Partials / Import）
把 CSS 拆成多个文件，统一引入，方便维护。

---

## 三、关键作用（为什么要用 Sass）
1. **告别重复代码** → 变量 + 混合极大减少冗余
2. **结构更清晰** → 嵌套写法和 HTML 结构对应
3. **便于团队维护** → 统一主题、统一规范
4. **支持逻辑运算** → 计算、判断、循环
5. **文件拆分管理** → 大型项目必备

---

## 四、最常用基本语法（直接复制就能用）
### 1. 变量 `$`
```scss
// 定义
$primary-color: #1677ff;
$font-size: 14px;

// 使用
.btn {
  color: $primary-color;
  font-size: $font-size;
}
```

---

### 2. 嵌套（最常用！）
```scss
// Sass 嵌套
.card {
  background: #fff;
  
  // 子元素
  .title {
    font-size: 18px;
  }
  
  // 伪类 & 代表父级
  &:hover {
    transform: scale(1.02);
  }
}
```

**编译后 CSS**
```css
.card { background: #fff; }
.card .title { font-size: 18px; }
.card:hover { transform: scale(1.02); }
```

---

### 4. 继承 `@extend`（抽公共样式）
```scss
.btn {
  padding: 8px 16px;
  border-radius: 4px;
}

.btn-primary {
  @extend .btn;
  background: blue;
}
```

---

### 5. 导入 `@import`（模块化）
```scss
// 把多个 Sass 文件合成一个
@import './variables';
@import './mixins';
@import './components/button';
```

---

### 6. 运算（直接算尺寸/颜色）
```scss
.box {
  width: 50% - 20px;
  margin: 10px * 2;
}
```

---

# 构建BEM架构需要掌握的核心概念
# 一、插值语句 `#{}` (Interpolation)
## 官方核心概念
**插值语句 `#{}` 是 Sass 最强大的语法之一**，作用：
**把 Sass 变量、表达式、计算结果，插入到 CSS 的“字符串/选择器/属性名”里。**

> 原生 Sass 变量只能用在**属性值**里；
> 想在**选择器、属性名、注释、字符串**里用变量，必须用 `#{}`。

## 关键作用
1. 在**选择器**中使用变量
2. 在**属性名**中使用变量
3. 在**字符串/路径**中嵌入变量
4. 拼接动态类名（配合 BEM 神器）

## 基本语法
```scss
// 1. 变量定义
$name: card;
$attr: color;
$val: #ff0000;

// 2. 插值用在 选择器
.#{$name} {
  // 3. 插值用在 属性名
  #{$attr}: $val;

  // 4. 插值用在 字符串
  background: url("./images/#{$name}.png");
}
```

**编译后**
```css
.card {
  color: #ff0000;
  background: url("./images/card.png");
}
```

## 最常用场景（BEM 必用）
```scss
$block: btn;

.#{$block} {
  &__text { }     // btn__text
  &--primary { }  // btn--primary
}
```

---

# 二、@at-root
## 官方核心概念
`@at-root` 是 Sass 的**跳出嵌套指令**。
作用：**让嵌套的样式，强制跳出父级选择器，直接输出到根层级**。

## 关键作用
1. 跳出深层嵌套，避免选择器层级过深
2. 在嵌套内写**全局样式**
3. 配合媒体查询、动画、关键帧使用
4. 保持代码结构清晰，又不生成冗余选择器

## 基本语法
```scss
.card {
  background: #fff;

  // 正常嵌套
  .title {
    font-size: 20px;
  }

  // 强制跳出到根层级
  @at-root .desc {
    color: #333;
  }
}
```

**编译后**
```css
.card { background: #fff; }
.card .title { font-size: 20px; }
.desc { color: #333; } /* 直接在根层级 */
```

## 高级常用（写动画/媒体查询）
```scss
.box {
  @at-root {
    @keyframes move { }
    @media (max-width:768px) { }
  }
}
```

---

# 三、Sass 参数（Mixins / Functions）
## 官方核心概念
Sass 的 **`@mixin` 和 `@function` 都支持参数**，让代码片段变成**可配置、可复用的动态样式工具**。

参数分为 3 种：
1. **普通参数**
2. **默认参数**
3. **不定长参数（可变参数）...**

## 关键作用
1. 让混合宏、函数**动态可配置**
2. 大幅减少重复代码
3. 实现通用工具样式（居中、尺寸、主题、圆角）

## 基本语法
### 1. 普通参数
```scss
@mixin size($w, $h) {
  width: $w;
  height: $h;
}

.box {
  @include size(100px, 100px);
}
```

### 2. 默认参数（最常用）
```scss
@mixin flex($dir: row, $justify: center) {
  display: flex;
  flex-direction: $dir;
  justify-content: $justify;
}

.box {
  @include flex; // 使用默认值
  @include flex(column); // 传第一个
}
```

### 3. 可变参数 ...
```scss
@mixin shadow($shadows...) {
  box-shadow: $shadows;
}

.box {
  @include shadow(0 0 5px red, 0 0 10px blue);
}
```

---

# 使用Sass构建BEM架构系统(Element Plus )

# 一、Element Plus 官方 BEM 规则（必须背会）
## 1. 命名格式（Element 官方标准）
```
[命名空间]-[块名]__[元素名]--[修饰符]
```

- **命名空间**：`el`（你项目可以自定义，如 `yz`、`app`）
- **Block**：`el-button`、`el-card`
- **Element**：`el-button__icon`
- **Modifier**：`el-button--primary`

## 2. 核心符号
- 双下划线 `__`：**元素**
- 双短横线 `--`：**修饰符/状态**

---

# 二、Element Plus 官方封装的 **BEM Sass 工具函数**
这是 Element 真实源码里用的核心 Sass 工具

## 1. 全局 BEM 配置 `bem.scss`
```scss
// 命名空间（Element 用 el，你可以改成自己项目的）
$namespace: 'app';

// Block 前缀：app-
$block-sel: '-' !default;

// Element 前缀：__
$element-sel: '__' !default;

// Modifier 前缀：--
$modifier-sel: '--' !default;

// 生成 Block：.app-block
@mixin b($block) {
  .#{$namespace}#{$block-sel}#{$block} {
    @content;
  }
}

// 生成 Element：.app-block__element
@mixin e($element) {
  &[#{$element-sel}]#{$element} {
    @content;
  }
}

// 生成 Modifier：.app-block--modifier
@mixin m($modifier) {
  &[#{$modifier-sel}]#{$modifier} {
    @content;
  }
}
```

---

# 三、实际组件中如何写？（真实业务代码）
```scss
// 引入 BEM 工具
@import '@/styles/bem.scss';

// 编写 Button 组件
@include b(button) {
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;

  // 元素：图标
  @include e(icon) {
    margin-right: 6px;
  }

  // 修饰符：主要按钮
  @include m(primary) {
    background: #409eff;
    color: #fff;
  }

  // 修饰符：成功按钮
  @include m(success) {
    background: #67c23a;
    color: #fff;
  }
}
```

## 编译后 CSS（标准 BEM 结构）
```css
.app-button {
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
.app-button__icon {
  margin-right: 6px;
}
.app-button--primary {
  background: #409eff;
  color: #fff;
}
.app-button--success {
  background: #67c23a;
  color: #fff;
}
```

---

# 四、全局 Sass 文件目录结构（Element Plus 标准结构）
项目结构：
```
src/
└── styles/
    ├── index.scss       # 主入口
    ├── bem.scss         # BEM 工具（Element 同款）
    ├── variables.scss   # 全局变量：颜色、尺寸、字体
    ├── mixins.scss      # 全局混合：居中、ellipsis 等
    └── reset.scss       # 样式重置
```

## 1. variables.scss（全局主题变量）
```scss
// 颜色（完全参考 Element Plus）
$color-primary: #409eff;
$color-success: #67c23a;
$color-warning: #e6a23c;
$color-danger: #f56c6c;

// 尺寸
$font-size-base: 14px;
$border-radius-base: 4px;
```

## 2. mixins.scss
```scss
// 文字溢出省略
@mixin ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// flex 居中
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## 3. index.scss（统一导出）
```scss
@import './variables.scss';
@import './bem.scss';
@import './mixins.scss';
@import './reset.scss';
```

---

# 五、在 Vue3 + Vite 中 **全局注入 Sass**（不用每个组件都 import）
## vite.config.js 配置
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  css: {
    preprocessorOptions: {
      scss: {
        // 自动注入全局样式文件
        additionalData: `@import "@/styles/index.scss";`
      }
    }
  }
})
```

**配置后，所有 Vue 组件、Sass 文件里，可直接使用：**
- 全局变量 `$color-primary`
- BEM 混合 `@include b(button)`
- 工具混合 `@include flex-center`

---

# 六、在 Vue 组件中使用（示例）
```vue
<template>
  <button class="app-button app-button--primary">
    <i class="app-button__icon"></i>
    确认按钮
  </button>
</template>

<style lang="scss" scoped>
// 无需再 import 任何文件！
@include b(button) {
  background: #fff;
  
  @include e(icon) {
    color: $color-primary;
  }

  @include m(primary) {
    background: $color-primary;
  }
}
</style>
```

---
## 可以这么说: Element Plus = Sass 全局变量 + Sass 混合函数 + BEM 命名规范


# 基于 Element UI 风格 + Sass + BEM 的 Layout 布局实战

*首先,我们要重置浏览器的默认样式--->*

## 企业级CSS样式重置文件(reset.css), 开箱即用:

    /* =============================================
      专业级 CSS 重置样式表 - 适用于所有商业项目
      融合：Normalize.css + 业务通用重置
    ============================================= */
    
    /* 1. 全局盒模型统一（最重要） */
    *,
    *::before,
    *::after {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    
    /* 2. 根元素基础设置 */
    html {
      /* 统一字体渲染 */
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      /* 根字体大小（方便 rem 计算） */
      font-size: 16px;
    }
    
    /* 3. body 基础样式 */
    body {
      /* 行业标准无衬线字体栈 */
      font-family:
        -apple-system, BlinkMacSystemFont,
        "Segoe UI", Roboto, "Helvetica Neue", Arial,
        "Noto Sans", sans-serif;
      /* 行高统一 */
      line-height: 1.5;
      /* 默认文字颜色（柔和黑） */
      color: #333;
      /* 背景色 */
      background-color: #fff;
    }
    
    /* 4. 列表重置 */
    ul, ol {
      list-style: none;
    }
    
    /* 5. 链接重置 */
    a {
      text-decoration: none;
      color: inherit;
    }
    
    a:hover {
      text-decoration: none;
    }
    
    /* 6. 表单元素重置 */
    button,
    input,
    optgroup,
    select,
    textarea {
      border: none;
      outline: none;
      font-family: inherit;
      font-size: inherit;
      background: none;
      appearance: none;
    }
    
    /* 按钮点击样式重置 */
    button {
      cursor: pointer;
    }
    
    button:disabled {
      cursor: not-allowed;
    }
    
    /* 7. 图片、媒体重置 */
    img,
    video,
    canvas,
    svg {
      max-width: 100%;
      vertical-align: middle;
      display: inline-block;
    }
    
    /* 8. 表格重置 */
    table {
      border-collapse: collapse;
      border-spacing: 0;
    }
    
    /* 9. 标题重置 */
    h1, h2, h3, h4, h5, h6 {
      font-weight: normal;
      font-size: inherit;
    }
    
    /* 10. 引用、斜体重置 */
    blockquote, q {
      quotes: none;
    }
    
    blockquote:before,
    blockquote:after,
    q:before,
    q:after {
      content: '';
      content: none;
    }
    
    /* 11. 清除浮动（通用工具类） */
    .clearfix::after {
      content: "";
      display: table;
      clear: both;
    }
    
    /* 12. 强制文字不换行 */
    .text-nowrap {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

## 文件价值:
1.  **统一所有浏览器默认样式**
消除 Chrome / Firefox / Safari / Edge 的差异。

2.  **box-sizing: border-box**
让宽高计算永远符合预期，专业开发必备。

3.  **表单、按钮、链接、图片全部重置**
不会出现默认边框、默认边距、默认下划线。

4.  **字体栈是行业标准**
适配 Windows / Mac / 移动端，显示最清晰。

5.  **自带工具类**
clearfix、text-nowrap 直接可用。

## 如何使用
HTML引入:

      <link rel="stylesheet" href="./reset.css">
      <link rel="stylesheet" href="./你的业务样式.css">
Vue / React / 工程化项目：

    import './reset.css'

# 实战编写

---

## 一、项目结构（遵循 Element 标准）
```
src/
├── styles/
│   ├── index.scss       # 全局样式入口
│   ├── variables.scss   # 全局变量
│   ├── bem.scss         # BEM 工具函数
│   └── mixins.scss      # 公共混合
└── components/
    └── Layout/
        ├── index.vue    # 布局组件
        └── index.scss   # 组件样式
```

---

## 二、全局 Sass 配置（核心 BEM 工具）
### 1. `styles/variables.scss`（全局主题变量）
```scss
// 颜色系统（Element UI 标准）
$color-primary: #409eff;
$color-bg-dark: #1f1f1f;
$color-bg-light: #282828;
$color-text-primary: #ffffff;
$color-text-secondary: #a8a8a8;
$color-border: #424242;

// 尺寸系统
$sidebar-width: 260px;
$header-height: 64px;
$border-radius-base: 4px;
$font-size-base: 14px;

// 过渡动画
$transition-base: all 0.3s ease;
```

### 2. `styles/bem.scss`（Element 同款 BEM 工具）
```scss
$namespace: 'app' !default;
$block-sel: '-' !default;
$element-sel: '__' !default;
$modifier-sel: '--' !default;

// 生成 Block: .app-block
@mixin b($block) {
  $B: #{$namespace}#{$block-sel}#{$block};
  .#{$B} {
    @content;
  }
}

// 生成 Element: .app-block__element
@mixin e($element) {
  $selector: &;
  @at-root {
    #{$selector}#{$element-sel}#{$element} {
      @content;
    }
  }
}

// 生成 Modifier: .app-block--modifier
@mixin m($modifier) {
  $selector: &;
  @at-root {
    #{$selector}#{$modifier-sel}#{$modifier} {
      @content;
    }
  }
}
```

### 3. `styles/mixins.scss`（公共工具混合）
```scss
// Flex 居中
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

// 文字溢出省略
@mixin ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// 清除默认样式
@mixin reset-base {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

### 4. `styles/index.scss`（全局入口）
```scss
@import './variables.scss';
@import './bem.scss';
@import './mixins.scss';

// 全局重置
*, *::before, *::after {
  @include reset-base;
}

html, body {
  height: 100%;
  font-size: $font-size-base;
  color: $color-text-primary;
  background-color: $color-bg-dark;
}
```

---

## 三、Vite 全局注入配置（无需手动 import）
在 `vite.config.js` 中配置自动注入全局 Sass：
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { fileURLToPath, URL } from 'node:url'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/index.scss";`
      }
    }
  }
})
```

---

## 四、Layout 组件完整代码
### 1. `components/Layout/index.vue`（组件结构）
```vue
<template>
  <div class="app-layout">
    <!-- 侧边栏 -->
    <aside class="app-layout__sidebar">
      <nav class="app-sidebar">
        <!-- Navigator One -->
        <div class="app-sidebar__group">
          <div class="app-sidebar__item app-sidebar__item--parent">
            <span class="app-sidebar__icon">✉️</span>
            <span class="app-sidebar__text">Navigator One</span>
            <span class="app-sidebar__arrow">⌃</span>
          </div>
          <div class="app-sidebar__children">
            <div class="app-sidebar__group-title">Group 1</div>
            <div class="app-sidebar__item">Option 1</div>
            <div class="app-sidebar__item">Option 2</div>
            <div class="app-sidebar__group-title">Group 2</div>
            <div class="app-sidebar__item">Option 3</div>
            <div class="app-sidebar__item app-sidebar__item--parent">
              <span class="app-sidebar__text">Option4</span>
              <span class="app-sidebar__arrow">⌄</span>
            </div>
          </div>
        </div>

        <!-- Navigator Two -->
        <div class="app-sidebar__group">
          <div class="app-sidebar__item app-sidebar__item--parent">
            <span class="app-sidebar__icon">🧩</span>
            <span class="app-sidebar__text">Navigator Two</span>
            <span class="app-sidebar__arrow">⌄</span>
          </div>
        </div>

        <!-- Navigator Three -->
        <div class="app-sidebar__group">
          <div class="app-sidebar__item app-sidebar__item--parent">
            <span class="app-sidebar__icon">⚙️</span>
            <span class="app-sidebar__text">Navigator Three</span>
            <span class="app-sidebar__arrow">⌃</span>
          </div>
          <div class="app-sidebar__children">
            <div class="app-sidebar__group-title">Group 1</div>
            <div class="app-sidebar__item">Option 1</div>
          </div>
        </div>
      </nav>
    </aside>

    <!-- 主内容区 -->
    <div class="app-layout__main">
      <!-- 头部 -->
      <header class="app-layout__header">
        <div class="app-header__user">
          <span class="app-header__icon">⚙️</span>
          <span class="app-header__name">Tom</span>
        </div>
      </header>

      <!-- 内容区 -->
      <main class="app-layout__content">
        <table class="app-table">
          <thead class="app-table__head">
            <tr>
              <th>Date</th>
              <th>Name</th>
              <th>Address</th>
            </tr>
          </thead>
          <tbody class="app-table__body">
            <tr v-for="i in 8" :key="i">
              <td>2016-05-02</td>
              <td>Tom</td>
              <td>No. 189, Grove St, Los Angeles</td>
            </tr>
          </tbody>
        </table>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
// 组件逻辑（可扩展菜单展开/收起）
</script>

<style lang="scss" scoped>
@include b(layout) {
  display: flex;
  height: 100vh;
  overflow: hidden;

  @include e(sidebar) {
    width: $sidebar-width;
    background-color: $color-bg-dark;
    border-right: 1px solid $color-border;
    flex-shrink: 0;
  }

  @include e(main) {
    flex: 1;
    display: flex;
    flex-direction: column;
    background-color: $color-bg-dark;
  }

  @include e(header) {
    height: $header-height;
    background-color: #2c4260;
    display: flex;
    justify-content: flex-end;
    align-items: center;
    padding: 0 20px;
    border-bottom: 1px solid $color-border;
  }

  @include e(content) {
    flex: 1;
    padding: 20px;
    overflow-y: auto;
  }
}

// 侧边栏样式
@include b(sidebar) {
  height: 100%;
  padding: 20px 0;

  @include e(group) {
    margin-bottom: 16px;
  }

  @include e(item) {
    padding: 12px 24px;
    cursor: pointer;
    transition: $transition-base;
    font-size: 18px;
    display: flex;
    align-items: center;
    gap: 12px;

    &:hover {
      background-color: $color-bg-light;
    }

    @include m(parent) {
      font-weight: 500;
    }
  }

  @include e(icon) {
    font-size: 20px;
    width: 24px;
    text-align: center;
  }

  @include e(text) {
    flex: 1;
  }

  @include e(arrow) {
    font-size: 16px;
    color: $color-text-secondary;
  }

  @include e(children) {
    padding-left: 24px;
  }

  @include e(group-title) {
    padding: 8px 24px;
    color: $color-text-secondary;
    font-size: 14px;
  }
}

// 头部样式
@include b(header) {
  @include e(user) {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 18px;
  }

  @include e(icon) {
    font-size: 20px;
  }
}

// 表格样式
@include b(table) {
  width: 100%;
  border-collapse: collapse;

  @include e(head) {
    th {
      text-align: left;
      padding: 12px 16px;
      border-bottom: 1px solid $color-border;
      color: $color-text-secondary;
      font-size: 16px;
    }
  }

  @include e(body) {
    td {
      padding: 16px;
      border-bottom: 1px solid $color-border;
      font-size: 16px;
    }

    tr:hover {
      background-color: $color-bg-light;
    }
  }
}
</style>
```

---
