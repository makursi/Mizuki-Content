---
title: Object-Oriented Programming (OOP)-面向对象编程
published: 2026-07-23
description: ''
image: ''
tags: []
category: 'OOP'
draft: false
pinned: false
lang: ''
---


# 对象、类、接口、包和继承

## 1. 对象 

### 定义

**对象是包含相关状态和行为的软件集合**

对象都具有两个共同特征：状态和行为。 例如狗有状态（名字、颜色、品种、饥饿状态）和行为（吠叫、叼东西、摇尾巴）。自行车也有状态（当前档位、当前踏频、当前速度）和行为（换档、改变踏频、刹车）。


对于真实对象，我们要考虑他们处于**哪些状态**和**执行哪些行为**的问题. 由于真实对象的复杂程度都不同,例如一个台灯可能只有两种状态（开和关）和两种行为（开和关），但一个桌面收音机可能还有其他状态（开、关、当前音量、当前电台）和行为（开、关、增大音量、减小音量、搜索、扫描和调谐）。

对于真实对象的观察是进入面向对象编程的起点

### 软件对象由状态和相关行为构成。

**对象将其状态存储在字段（某些编程语言中的变量）中，并通过方法（某些编程语言中的函数）公开其行为。**

方法操作对象的内部状态，是对象间通信的主要机制。隐藏内部状态并要求所有交互都通过对象的方法执行，这被称为*数据封装*——面向对象编程的基本原则。

以自行车为例:

通过赋予对象状态（当前速度、当前踏频和当前档位）并提供改变该状态的方法，对象可以控制外部世界如何使用它。

### 代码打包成独立的软件对象的好处(为什么要用OOP编程)

1. 模块化：一个对象的源代码可以独立于其他对象的源代码编写和维护。对象一旦创建，即可在系统内部轻松传递。

2. 信息隐藏: 通过仅与对象的方法进行交互，其内部实现的细节对外界保持隐藏。

3. 代码重用: 某些由开发者实现的对象经过严格的测试，调试的复杂对象，可以直接在程序中使用

4. 可插拔性和易于调试 :如果某个对象出现问题，可以直接将其从应用程序中移除，并用另一个对象替换它。


# 2. 类

创建的对象会有很多同类对象,例如:可能存在成千上万辆自行车，它们的品牌和型号都完全相同。


我们可以简单定义一个Bicycle类并赋予它状态和行为:


        class Bicycle {

            int cadence = 0;
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


其中 `cadenc` ,`speed` ,`gear` 为由Bicycle类的对象的状态，`changeCadence`,`changeGear`,`speedUp`,`applyBrakes`,`printStates` 为方法,定义了对象与外部世界的交互。


在BicycleDemo创建两个独立Bicycle对象并调用它们方法的类:

        class BicycleDemo {
            public static void main(String[] args) {

                // Create two different 
                // Bicycle objects
                Bicycle bike1 = new Bicycle();
                Bicycle bike2 = new Bicycle();

                // Invoke methods on 
                // those objects
                bike1.changeCadence(50);
                bike1.speedUp(10);
                bike1.changeGear(2);
                bike1.printStates();

                bike2.changeCadence(50);
                bike2.speedUp(10);
                bike2.changeGear(2);
                bike2.changeCadence(40);
                bike2.speedUp(10);
                bike2.changeGear(3);
                bike2.printStates();
            }
        }

结果: 

        cadence:50 speed:10 gear:2
        cadence:40 speed:20 gear:3



## 3. 继承

不同类型的对象之间通常有一些共同之处。例如，山地车、公路车和双人自行车都具备自行车的基本特征（当前速度、当前踏频、当前档位）。 

但他们也有一些自己独特的特征：双人自行车有两个车座和两组车把；公路车使用弯把；一些山地车则配备额外的链轮，从而拥有更低的齿比。


### 面向对象编程允许类从其他类继承常用的状态和行为。

Java中每个类只能有一个直接超类，而每个超类可以拥有无​​限数量的子类


使用extends 关键字,后跟要继承的类的名称创建子类：

        class MountainBike extends Bicycle {

            // new fields and methods defining 
            // a mountain bike would go here

        }


子类就拥有了MountainBike与父类相同的所有字段(和变量一个性质的东东)和方法Bicycle，**同时又能将代码的重点放在使其独一无二的特性上**。



## 4.接口

**对象通过它们公开的方法来定义与外部世界的交互。方法构成了对象与外部世界的接口**

例如，电视机前面板上的按钮就是你与塑料外壳另一侧的电线之间的接口。你按下“电源”按钮来打开和关闭电视。

接口最常见的形式是一组具有空方法体的相关方法。 使用接口来描述自行车行为:

        interface Bicycle {

            //  wheel revolutions per minute
            void changeCadence(int newValue);

            void changeGear(int newValue);

            void speedUp(int increment);

            void applyBrakes(int decrement);
        }

### 实现接口

实现此接口，类名要更改（例如，更改为特定品牌的自行车，例如ACMEBicycle），并且要在类声明中使用implements关键字:

        class ACMEBicycle implements Bicycle {

            int cadence = 0;
            int speed = 0;
            int gear = 1;

        // The compiler will now require that methods
        // changeCadence, changeGear, speedUp, and applyBrakes
        // all be implemented. Compilation will fail if those
        // methods are missing from this class.

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

实现接口可以让类更正式地定义其承诺提供的行为。接口在类和外部世界之间形成了一种契约，而这种契约会在编译时由编译器强制执行。

**如果你的类声明实现了某个接口，那么该接口定义的所有方法都必须出现在其源代码中，类才能成功编译。**

## 包

在 Java 中，**包（Package）是一种命名空间（namespace）**，用于组织一组相关的类（class）和接口（interface）。  

可以把包理解为计算机中的**文件夹**：
- HTML 文件放在一个文件夹
- 图片放在另一个文件夹
- 脚本或程序放在第三个文件夹

通过包，可以将功能相似、逻辑相关的类归类在一起，避免命名冲突，让项目结构清晰有序。

### 包的作用

Java 程序往往由成百上千个类组成。如果不进行组织，代码会变得极其混乱。使用包可以有效管理这些类，提高代码的可维护性和可读性。

### Java 平台 API（核心类库）

Java 平台提供了一个庞大的**标准类库**，被称为 **Application Programming Interface（API）**。这个 API 由大量的包组成，覆盖了绝大多数通用编程任务。

**常见示例**：
- `String` 类：处理字符字符串
- `File` 类：创建、删除、检查、修改文件
- `Socket` 类：进行网络通信
- 各种 GUI 类：控制按钮、复选框等图形界面元素
