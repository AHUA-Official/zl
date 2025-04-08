---
title: 深入理解CSS变量
date: 2023-06-20 10:15:00
tags:
  - CSS
  - 前端
  - 设计
---

# 深入理解CSS变量

CSS变量（又称自定义属性）是现代前端开发中的强大工具，它为我们提供了创建可复用、可维护CSS的新方法。本文将深入探讨CSS变量的使用技巧和实战应用。

## 基础知识

CSS变量使用`--`前缀定义，通过`var()`函数调用：

```css
:root {
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
  --font-size-base: 16px;
}

.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-base);
}
```

## 变量作用域

CSS变量遵循级联规则，可以在不同的选择器中定义相同名称的变量，形成作用域：

```css
:root {
  --text-color: black;
}

.dark-theme {
  --text-color: white;
}

p {
  color: var(--text-color); /* 继承最近的定义 */
}
```

## 响应式设计与CSS变量

结合媒体查询，CSS变量可以轻松实现响应式设计：

```css
:root {
  --container-width: 1200px;
}

@media (max-width: 768px) {
  :root {
    --container-width: 100%;
  }
}

.container {
  width: var(--container-width);
}
```

## JavaScript交互

CSS变量可以通过JavaScript动态修改，实现交互效果：

```javascript
// 获取CSS变量值
const rootStyles = getComputedStyle(document.documentElement);
const primaryColor = rootStyles.getPropertyValue('--primary-color').trim();

// 设置CSS变量值
document.documentElement.style.setProperty('--primary-color', '#ff0000');
```

## 主题切换实现

CSS变量最常见的应用之一是实现主题切换功能：

```css
:root {
  /* 浅色主题（默认） */
  --bg-color: #ffffff;
  --text-color: #333333;
  --border-color: #dddddd;
}

.dark-theme {
  /* 深色主题 */
  --bg-color: #222222;
  --text-color: #eeeeee;
  --border-color: #444444;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
}

.card {
  border: 1px solid var(--border-color);
}
```

配合JavaScript切换类名：

```javascript
function toggleTheme() {
  document.body.classList.toggle('dark-theme');
  // 保存用户偏好
  localStorage.setItem('theme', document.body.classList.contains('dark-theme') ? 'dark' : 'light');
}
```

## 高级应用：CSS变量与计算

结合`calc()`函数，CSS变量可以参与计算：

```css
:root {
  --spacing-unit: 8px;
}

.card {
  padding: var(--spacing-unit);
  margin-bottom: calc(var(--spacing-unit) * 2);
}

.card-title {
  margin-bottom: calc(var(--spacing-unit) * 1.5);
  font-size: calc(var(--font-size-base) * 1.2);
}
```

## 浏览器兼容性

CSS变量在现代浏览器中得到广泛支持，但IE不支持。可以使用回退值和预处理器提供兼容方案：

```css
.button {
  background-color: #3498db; /* 回退值 */
  background-color: var(--primary-color, #3498db);
}
```

## 总结

CSS变量为样式系统带来了前所未有的灵活性，特别适合构建主题系统、组件库和响应式设计。通过恰当使用CSS变量，我们可以编写更加模块化、可维护的样式代码。

在实际项目中，可以将CSS变量与预处理器（如Sass）结合使用，获得两者的优势。CSS变量用于运行时需要变化的值，而预处理器变量用于编译时确定的值。

你是如何在项目中应用CSS变量的？欢迎分享你的经验！ 