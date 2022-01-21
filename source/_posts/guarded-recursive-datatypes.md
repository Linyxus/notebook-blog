---
title: 论文笔记：Guarded Recursive Datatype Constructors
date: 2022-01-16 19:45:14
tags:
---

Xi 于 2003 年发表的 Guarded Recursive Datatype Constructors 论文是编程语言中 Generalized Algebraic Data Types (GADTs) 的理论基础。在这篇论文中，作者首先设计了一种支持 GADT 的演算，对 GADT 的理论特性进行建模，并且证明 soundness；此外，作者也对 GADT 的实际应用进行了讨论。

<!-- more -->

## Motivation

编程语言中，多态函数时分常见。下面的 `map` 函数就是 Haskell 中非常典型的多态函数。
``` haskell
map :: (a -> b) -> [a] -> [b]
map [] = []
map (x:xs) = f x : map f xs
```
`map` 函数具有一个非常重要的特性：它对于不同的参数类型`a`和`b`，在运行时进行的操作完全一致。然而，现实生活中我们很可能需要对不同的参数类型执行不同的操作，这应当如何实现呢？

GADT 是一个可能的答案。下面用 Scala 给出了一个 GADT 类型的定义。
``` scala
enum Type[A]:
  case IntType extends Type[Int]
  case TupType[A, B](a: Type[A], b: Type[B]) extends Type[(A, B)]
  case FunType[A, B](a: Type[A], b: Type[B]) extends Type[A => B]
  case TypeType[A](a: Type[A]) extends Type[Type[A]]
```
基于上面的 GADT 类型定义，我们能够实现一个在运行时对不同参数类型有不同行为的函数。
``` scala
def printVal[X](typ: Type[X], x: X): String = typ match {
    case IntType => s"<int: $x>"
    case TupType(a, b) => s"(${printVal(a, x._1)}, ${printVal(b, x._2)})"
    case FunType(_, _) => "a function"
    case TypeType(_) => "a type"
}
```

## $\lambda_{2,G\mu}$ 演算

为了建模 GADT，也即论文中的 g.r. datatypes，作者提出了一种演算：$\lambda_{2,G\mu}$。
下面给出了这一演算的语法定义。
{% asset_img lambda_2Gmu_syntax.png Syntax of Lambda2Gmu %}

这一演算的语法定义非常常规。值得注意的是其关于**类型变量**的相关语法结构。在 $\lambda_{2,G\mu}$ 中，$\Delta$ 表示类型变量的环境 (type variable context)。$\Delta$ 可以被认为是一系列类型变量的**声明**，列出了环境中所有有效的类型变量，以及类型变量之间的等式。而相应的，$\Theta$ 是类型变量的定义，本质上是从类型变量到实际类型的代换。

## GADT 构造器

在 $\lambda_{2,G\mu}$ 的语法中，GADT 的构造器可以被定义如下。
``` ocaml
typecon (type, type, ..., type) T = {'a1, 'a2, ..., 'an} . (t1, t2, ..., tn) c1 of T1
                                  | {'a1, 'a2, ..., 'an} . (t1, t2, ..., tn) c2 of T2
                                  | ...
                                  | {'a1, 'a2, ..., 'an} . (t1, t2, ..., tn) ck of Tk
```
由此定义的 GADT 类型，可以被表示为 $\mu t. \lambda \vec{\alpha} (\exists \{ \vec{\alpha}_1, \vec{\tau}_1 \equiv \vec{\alpha} \}. \tau_1 + \cdots + \exists \{ \vec{\alpha}_k, \vec{\tau}_k \equiv \vec{\alpha} \}. \tau_k)$。
