---
title: Java中的变量
published: 2026-07-23
description: 
tags: [Java]
category: 'java'
draft: false
---

# 字段和变量

### 变量是一个存储位置，它拥有一个类型，该类型可以是基本类型或引用类型。

变量 = 程序运行过程中用于保存数据的一个命名存储空间。

### 字段是作为类或接口成员的变量。

字段必须满足：
- 是变量
- 属于某个 class 或 interface
- 定义在类的成员位置


所以字段是变量的一种表达形式 



## Java中的变量

以Bicycle类为例: 

        class Bicycle {

            int static cadence = 0;
            int speed = 0;
            int gear = 1;

            void changeCadence(int newValue) {
                cadence = newValue;
            }

            void changeGear(int newValue) {
                gear = newValue;
            }

            void speedUp(int increment) {
                speed = speed + increment;   
            }

            void applyBrakes(int decrement) {
                speed = speed - decrement;
            }

            void printStates() {
                IO.println("cadence:" +
                    cadence + " speed:" + 
                    speed + " gear:" + gear);
            }
        }

1. **实例变量(非静态字段)**,对象会将自身的状态存储在“非静态字段”中，也就是没有使用 `static` 关键字声明的字段。非静态字段也称为实例变量，因为它们的值对于类的每个实例（每个对象）都是唯一的；

2. **类变量（静态字段）类变量**,是指任何static使用 `@class` 修饰符声明的字段；这告诉编译器，无论类被实例化多少次，该变量都只有一个副本。

3.  **局部变量**,类似于对象将状态存储在字段中的方式，方法通常会将临时状态存储在局部变量中。没有特殊的关键字来指定变量是否为局部变量；这种判断完全取决于变量的声明位置——即方法的左大括号和右大括号之间。因此，局部变量仅对声明它们的方法可见；类的其他部分无法访问它们。

4.  **参数**,参数始终被归类为“变量”，而不是“字段”。

在上面这个例子中， cadence 为类变量 , speed,gear为实例变量，局部变量的一个很好的例子就是在函数式编程中for循环中的(),{} 中定义的变量为局部变量


# Java 基本数据类型（Primitive Types）

## 概述

Java 是静态类型语言。所有变量使用前必须声明类型和名称。基本数据类型由语言预定义，使用保留关键字命名，共八种，不共享状态。

## 八种基本数据类型

- **byte**：8 位有符号整数。范围：-128 ~ 127。适用于节省内存的大型数组。
- **short**：16 位有符号整数。范围：-32768 ~ 32767。适用于节省内存场景。
- **int**：32 位有符号整数。默认整数类型。范围：-2³¹ ~ 2³¹-1。
- **long**：64 位有符号整数。范围：-2⁶³ ~ 2⁶³-1。数值后加 `L` 或 `l`。
- **float**：32 位单精度浮点数。不适合精确计算（如货币），推荐使用 `BigDecimal`。
- **double**：64 位双精度浮点数。默认浮点类型。
- **boolean**：布尔类型。仅 `true` 或 `false`。
- **char**：16 位 Unicode 字符。范围：'\u0000' ~ '\uffff'。

## 默认值

未初始化的字段自动获得默认值：
- 数值类型：0（`long` 为 0L）
- `boolean`：false
- `char`：'\u0000'

局部变量必须显式初始化。

## 字面量（Literals）

直接在代码中表示固定值。

- **整数字面量**：默认 `int`，加 `L`/`l` 为 `long`。支持下划线分隔（如 `1_000_000`）。
- **浮点字面量**：默认 `double`，加 `F`/`f` 为 `float`。
- **字符字面量**：单引号，如 `'C'`。
- **布尔字面量**：`true`、`false`。

## 使用建议

- 优先使用 `int` 处理整数。
- 大整数使用 `long`。
- 浮点计算优先使用 `double`。
- 节省内存时考虑 `byte`、`short`、`float`。
- 精确值避免使用 `float` 或 `double`。
