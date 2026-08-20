# 0x08 HAM 神谕及系统调用

神谕的定义：一个系统自身之外的影响因素

HAM 需要神谕来完成功能，常见的神谕形式如下：

- 副作用的产生与接收
- 不可预测的递归函数
- 无限循环
- 无限递归
- 随机性

## 神谕的通用接口

集合形式：如一些底层的数据实现方式

单子形式：如一些操作序列的组装方式

## 系统调用

HAM 提供一种 monad 形式的系统调用，并提供 monad 转换器神谕将 monad 转换为目标程序。

在编译的时候，需要指定程序的“入口”，这个“入口”是一个表达式，解析为 monad 被 monad 转换器转换后的目标程序。

HAM 可以有多个入口，也可以有多条 monad 链同时存在，甚至允许 monad 链分支，形成树形，但 HAM 的一个入口只对应一个 monad 链，在编译的时候其他部分会被惰性忽略。

你通常需要把 os 在函数之间传递或者覆写来进行系统调用。

### import

外部接口用 `import(name)` 获取，`import` 返回一个组合，可以用提取语法取出需要的键：

```HAM
{ os } = import("os"),   // 操作系统接口
{ cl } = import("cl")    // Common Lisp 后端
```

`os` 提供系统相关的神谕，如 `random`、`input`、`output` 等。调用它们会推进 monad 状态，通常把结果覆写回 `os` 并继续传递：

```HAM
os = os.random,   // 请求一个随机值
ans = os.value,   // 读取随机值
os = os.output("hi\n") // 输出副作用
```

`cl` 是 monad 转换器神谕的一个实例：`cl(入口表达式)` 把入口的 monad 转换为 Common Lisp 目标程序。编译时以 `--entry` 指定入口，`--output` 指定输出：

```HAM
mainCl = cl(game(os)), // 将 game(os) 编译为 Common Lisp
```

```sh
ham ./guess.ham --entry='mainCl' --output guess
```
